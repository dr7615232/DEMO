# Technical Deep‑Dives — AdLeader

Four pieces of engineering worth a closer look. All code shown is real (pure logic, no secrets), and
where something is an implemented‑but‑unwired library it is labelled as such.

---

## 1. Race‑safe lead identity resolution and first/last‑touch attribution

**The problem.** A returning customer can inquire twice in the same minute — once from a WhatsApp
click, once from a form — on two stateless serverless invocations that run concurrently. Naïvely,
that creates two lead records and loses the connection between visits. Attribution must also survive
the merge: the *first* source that ever brought the lead should be preserved, while the *most recent*
source is advanced.

**The approach.** Identity is resolved inside a single Firestore transaction against a `leadKeys`
index (`phone:<normalized>` / `email:<normalized>` → `leadId`). The transaction reads both keys,
merges into an existing lead if either matches (falling back to a legacy query), otherwise creates a
new document — and **claims the keys in the same transaction**, so two concurrent inquiries can't both
win. On merge it preserves first‑touch and advances last‑touch, increments `inquiryCount` /
`touchpointCount`, appends an immutable touchpoint subdoc, and — for a lead already in the CRM — sets
`hasNewInquiry` instead of silently folding the visit in.

```ts
// lib/server/domain/leads.ts — first-touch preserved, last-touch advanced (shape)
const firstSourceId  = existing?.firstSourceId  ?? incoming.sourceId;
const firstCampaignId = existing?.firstCampaignId ?? incoming.campaignId;
const lastSourceId   = incoming.sourceId  ?? existing?.lastSourceId;
const lastCampaignId  = incoming.campaignId ?? existing?.lastCampaignId;
// keys are written inside the SAME transaction (claimKeys) → race-safe dedup
```

**Why it matters.** This is the backbone of honest attribution: every inquiry is deduplicated and
bound to a source exactly once, idempotently under provider retries, so per‑source counts and P&L are
trustworthy. Side effects (mailing‑list sync, automations) are pushed to a `jobs` collection so the
webhook answers immediately and never loses work if a downstream provider is slow.

---

## 2. Attribution at capture time — parsing the real world

**The problem.** Inbound leads arrive as JSON, URL‑encoded, or multipart, from form builders
(Elementor, Zapier, and friends) whose field shapes are unpredictable and whose labels are in Hebrew
*or* English. The tracking code may be in a body field or a URL query parameter. And the endpoint is
public, so it must be hostile‑input‑safe and must never leak whether a lead exists.

**The approach.** `app/api/webhooks/leads/route.ts` runs a strict order of operations:
per‑IP throttle → **resolve the token to a business before any write** → per‑business throttle →
`flattenFormLikePayload` (recursively flattens arbitrary shapes with a depth cap) → match Hebrew and
English field aliases (`שם`/`טלפון`/`מייל`) → extract the `MAKOR` tracking code from body or query →
`upsertLeadWithTouchpoint`. Attribution is decided at this moment:

```ts
// If MAKOR resolves to a tracking link of the SAME business:
//   attributionStatus = "confirmed",  attributionMethod = "tracking_link"
// else:
//   attributionStatus = "manual_required"   → surfaced as "דורש שיוך ידני"
```

The hosted/embedded form (`app/api/forms/[formId]/submit/route.ts`) adds a Zod schema with capped
fields and a **honeypot** (`website` field → fake success, no write), required‑field and
consent enforcement, and audit logging. The public response is uniform so it can't be used to probe
for existing leads.

**Why it matters.** Attribution is only as good as capture. By binding the source at the moment the
lead is created — from whatever mess the outside world sends — the system closes the gap between "we
paid for an ad" and "this specific inquiry came from it."

---

## 3. AES‑256‑GCM at rest for every third‑party credential

**The problem.** The app stores tenants' WhatsApp tokens, SMTP passwords, email‑marketing API keys,
and OAuth refresh tokens. These must be encrypted at rest, authenticated (tamper‑evident), and
decryptable only server‑side.

**The approach.** One small primitive (`lib/server/encryption.ts`) encrypts arbitrary JSON with
AES‑256‑GCM: a fresh 12‑byte IV per encryption, the GCM auth tag stored alongside the ciphertext, and
a 32‑byte key loaded from `INTEGRATION_ENCRYPTION_KEY` (accepting either 64‑hex or base64).

```ts
export function encryptJson(value: unknown) {
  const iv = crypto.randomBytes(12);
  const cipher = crypto.createCipheriv("aes-256-gcm", getKey(), iv);
  const plaintext = Buffer.from(JSON.stringify(value), "utf8");
  const encrypted = Buffer.concat([cipher.update(plaintext), cipher.final()]);
  const tag = cipher.getAuthTag();
  return { algorithm: "aes-256-gcm", iv: iv.toString("base64"),
           tag: tag.toString("base64"), data: encrypted.toString("base64") };
}
```

Decryption calls `setAuthTag` before `final()`, so any tampering throws. The same primitive protects
tracking‑link webhook tokens and the platform Google‑storage refresh token. `getIntegrationConfig`
returns a decrypted config **only** when the integration's `status === "active"`, and every webhook
token is stored as a SHA‑256 hash plus an AES‑encrypted copy — plaintext is never persisted in the
clear.

**Why it matters.** It's the difference between "we store your credentials" and "a database leak
still doesn't hand an attacker your customers' WhatsApp and payment provider access."

---

## 4. Stateless‑serverless‑aware infrastructure primitives

**The problem.** On Vercel there is no shared memory between invocations, so the usual tricks
(in‑process rate limiters, in‑memory idempotency, background timers) don't exist. The app still needs
rate limiting, webhook idempotency, and reliable side effects.

**The approach.** All three are built on Firestore:

- **Rate limiter** (`lib/server/rate-limit.ts`): a transactional fixed‑window counter keyed by
  `(key, window)`, layered per‑IP and per‑business on every public endpoint, TTL‑swept, and
  **fail‑open** on a Firestore error so a monitoring blip can't lock out real traffic.
- **Idempotency** (`withWebhookClaim`): uses a Firestore `create()` as an atomic lock —
  `ALREADY_EXISTS` means "duplicate delivery, skip"; the claim is released if processing throws, so a
  genuine provider retry isn't swallowed. 30‑day TTL.
- **Deferred work** (`jobs` collection): webhooks enqueue side effects and return immediately; a cron
  (`api/cron/automations`) drains the queue with exponential backoff (1m → 4m → 16m … capped at 6h),
  with a `waitUntil` fast path.

The public tracking‑link redirect ties several of these together and stays deliberately small:

```ts
// app/r/[trackingCode]/route.ts
const ip = clientIp(request) || "unknown";
const limit = await checkRateLimit({ key: `click:${ip}:${code}`, limit: 5, windowSeconds: 60 });
const result = await resolveTrackingClick(code, request, { recordClick: limit.allowed });
if (result.ok)       return NextResponse.redirect(result.destinationUrl, { status: 302 });
if (result.fallbackUrl) return NextResponse.redirect(result.fallbackUrl, { status: 302 });
// else: friendly Hebrew 404
```

A bot hammering one link is still redirected (good UX), but only the first few visits per minute count
as clicks, so stats and quotas aren't inflated.

**Why it matters.** It's a coherent answer to "how do you do rate limiting, idempotency, and reliable
background work with no Redis and no server?" — and it's the kind of infrastructure reasoning that
separates a toy from something you'd trust with a client's money.

---

## Appendix — the ads‑KPI library (implemented, not yet wired)

`lib/utils/kpiCalculations.ts` is a complete, unit‑tested library of paid‑ads metrics (ROAS, ROI, CPA,
CPL, CTR, CPC, CPM, CVR, frequency, engagement, CTOR — all divide‑by‑zero‑safe) and **five attribution
models**. It is currently imported only by its tests and demo data; the live reporting path uses the
lead/deal/expense P&L in `lib/calculations/metrics.ts` plus lead‑level first/last‑touch. Presented
here as a ready building block, not as the running attribution model.

```ts
// lib/utils/kpiCalculations.ts — five attribution models (excerpt)
if (model === "first_click")   add(sorted[0].id, 1);
if (model === "last_click")    add(sorted[sorted.length - 1].id, 1);
if (model === "linear")        sorted.forEach((p) => add(p.id, 1 / sorted.length));
if (model === "position_based"){ add(sorted[0].id, 0.4); add(sorted.at(-1).id, 0.4);
                                  sorted.slice(1, -1).forEach((p) => add(p.id, 0.2 / (sorted.length - 2))); }
if (model === "time_decay") {  // 7-day half-life
  const weights = sorted.map((p) => Math.pow(0.5, days(conversionDate, p) / 7));
  const total = weights.reduce((s, w) => s + w, 0);
  sorted.forEach((p, i) => add(p.id, weights[i] / total));
}
```
