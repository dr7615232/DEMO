# Selected Code Snippets — AdLeader

Publish‑safe excerpts (pure logic, no secrets, no PII). Each is real code with a short note on why it
is interesting.

---

## 1. The live reporting engine — a lead/deal/expense P&L

`lib/calculations/metrics.ts`. This is the engine actually wired into production reports. It turns
raw leads, deals, and expenses into the numbers a business owner cares about, with divide‑by‑zero
handled once via `safeRatio` (returning `null`, which the UI renders as "אין מספיק נתונים").

```ts
function safeRatio(numerator: number, denominator: number) {
  if (!denominator) return null;
  const value = numerator / denominator;
  return Number.isFinite(value) ? value : null;
}

export function calculateBasicMetrics(input: MetricInput): BasicMetrics {
  const uniqueLeads = input.leads.length;
  const totalInquiries = input.leads.reduce(
    (sum, lead) => sum + Math.max(1, Number(lead.inquiryCount ?? lead.touchpointCount ?? 1)), 0);
  const leads = uniqueLeads;
  const newLeads = input.leads.filter((lead) => lead.status === "new").length;
  const wonDeals = input.deals.filter((deal) => deal.status === "won").length;
  const revenue = input.deals
    .filter((deal) => deal.status === "won")
    .reduce((sum, deal) => sum + Number(deal.amount ?? 0), 0);
  const expenses = input.expenses.reduce((sum, expense) => sum + Number(expense.amount ?? 0), 0);
  const profit = revenue - expenses;

  return {
    leads, uniqueLeads, totalInquiries, newLeads, wonDeals, revenue, expenses, profit,
    averageCostPerLead: safeRatio(expenses, leads),
    averageCostPerWonCustomer: safeRatio(expenses, wonDeals),
    leadToCustomerRate: safeRatio(wonDeals, leads),
    revenuePerAdShekel: safeRatio(revenue, expenses),
  };
}
```

`metricsByDimension(input, "sourceId" | "campaignId")` re‑buckets the same computation per source or
per campaign (respecting the first/last‑touch fallback `sourceId || lastSourceId`), which is what
powers the per‑source ROI table in the demo.

---

## 2. Authenticated AES‑256‑GCM encryption for stored credentials

`lib/server/encryption.ts`. Fresh IV per encryption, GCM auth tag stored with the ciphertext, 32‑byte
key from env accepting hex or base64.

```ts
function getKey() {
  const raw = process.env.INTEGRATION_ENCRYPTION_KEY;
  if (!raw) throw new Error("INTEGRATION_ENCRYPTION_KEY is not configured");
  const decoded = raw.length === 64 ? Buffer.from(raw, "hex") : Buffer.from(raw, "base64");
  if (decoded.length !== 32) throw new Error("INTEGRATION_ENCRYPTION_KEY must be 32 bytes");
  return decoded;
}

export function decryptJson<T>(payload: unknown): T {
  const input = payload as { iv?: string; tag?: string; data?: string };
  if (!input?.iv || !input.tag || !input.data) throw new Error("Encrypted payload is invalid");
  const decipher = crypto.createDecipheriv("aes-256-gcm", getKey(), Buffer.from(input.iv, "base64"));
  decipher.setAuthTag(Buffer.from(input.tag, "base64"));
  const decrypted = Buffer.concat([decipher.update(Buffer.from(input.data, "base64")), decipher.final()]);
  return JSON.parse(decrypted.toString("utf8")) as T;
}
```

---

## 3. The public tracking‑link redirect — small, rate‑limited, friendly on failure

`app/r/[trackingCode]/route.ts`. Every visit is redirected; only the first few per minute count as
clicks; an unknown/inactive link returns a friendly Hebrew 404 rather than an error.

```ts
export async function GET(request: NextRequest, { params }: { params: { trackingCode: string } }) {
  const code = String(params.trackingCode ?? "").slice(0, 32);
  const ip = clientIp(request) || "unknown";
  const limit = await checkRateLimit({ key: `click:${ip}:${code}`, limit: 5, windowSeconds: 60 });
  const result = await resolveTrackingClick(code, request, { recordClick: limit.allowed });

  if (result.ok)          return NextResponse.redirect(result.destinationUrl, { status: 302 });
  if (result.fallbackUrl) return NextResponse.redirect(result.fallbackUrl, { status: 302 });
  return new NextResponse(/* friendly Hebrew 404 page */, {
    status: 404, headers: { "content-type": "text/html; charset=utf-8" },
  });
}
```

---

## 4. Default CRM pipeline — sensible Hebrew defaults with safe fallback

`lib/server/domain/crm.ts`. Stages and lead statuses ship with Hebrew defaults and colours; a
business can customise them, but a stored‑but‑empty list falls back to the defaults so the pipeline
and status pickers never end up with zero options.

```ts
export const DEFAULT_PIPELINE_STAGES: CrmOption[] = [
  { id: "new",         label: "פנייה חדשה", color: "#00b7c7" },
  { id: "contacted",   label: "נוצר קשר",   color: "#15508f" },
  { id: "proposal",    label: "נשלחה הצעה", color: "#7c3aed" },
  { id: "negotiation", label: "במשא ומתן",  color: "#f59e0b" },
];

function sanitizeOptions(raw: unknown, fallback: CrmOption[]): CrmOption[] {
  if (!Array.isArray(raw)) return fallback;
  const cleaned = raw
    .map((s) => ({ id: String(s?.id ?? "").trim(), label: String(s?.label ?? "").trim(),
                   color: String(s?.color ?? "#64748b") }))
    .filter((s) => s.id && s.label);
  return cleaned.length ? cleaned : fallback; // cleared list → defaults, never empty
}
```

---

## 5. Multi‑touch attribution models (library — not yet wired into the live flow)

`lib/utils/kpiCalculations.ts`. A complete, unit‑tested library of five attribution models. The live
system attributes at capture time (first/last‑touch); this is a ready building block for a future
multi‑touch reporting view.

```ts
export function applyAttribution(model: AttributionModel, touchpoints: Touchpoint[], conversionDate = new Date()) {
  if (touchpoints.length === 0) return new Map<string, number>();
  const sorted = [...touchpoints].sort((a, b) => a.occurredAt.getTime() - b.occurredAt.getTime());
  const credit = new Map<string, number>();
  const add = (id: string, value: number) => credit.set(id, (credit.get(id) ?? 0) + value);

  if (model === "first_click") add(sorted[0].id, 1);
  if (model === "last_click")  add(sorted[sorted.length - 1].id, 1);
  if (model === "linear")      sorted.forEach((p) => add(p.id, 1 / sorted.length));
  if (model === "position_based") {
    if (sorted.length === 1) add(sorted[0].id, 1);
    else if (sorted.length === 2) sorted.forEach((p) => add(p.id, 0.5));
    else { add(sorted[0].id, 0.4); add(sorted[sorted.length - 1].id, 0.4);
           sorted.slice(1, -1).forEach((p) => add(p.id, 0.2 / (sorted.length - 2))); }
  }
  if (model === "time_decay") {
    const weights = sorted.map((p) => {
      const days = Math.max(0, (conversionDate.getTime() - p.occurredAt.getTime()) / 86_400_000);
      return Math.pow(0.5, days / 7); // 7-day half-life
    });
    const total = weights.reduce((s, w) => s + w, 0);
    sorted.forEach((p, i) => add(p.id, weights[i] / total));
  }
  return credit;
}
```
