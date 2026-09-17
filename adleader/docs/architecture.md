# Architecture — AdLeader

Next.js 14 (App Router) + Firebase, TypeScript, Hebrew RTL, multi‑tenant. Deploy target: Vercel
(serverless) + Firebase (Firestore rules/indexes, Storage rules). Every claim below is verified
against source.

---

## 1. System context

```mermaid
graph TD
  subgraph Public["Public / unauthenticated"]
    Visitor["Ad visitor"]
    Owner["Business owner (browser)"]
    Providers["External providers<br/>(WhatsApp / Green API, Gmail,<br/>Sumit, email-marketing, form builders)"]
  end

  subgraph App["AdLeader — Next.js 14 on Vercel"]
    MW["middleware.ts<br/>(cookie session gate, i18n, demo mode)"]
    Dash["(dashboard) app shell<br/>RTL, next-intl"]
    Admin["admin console"]
    Capture["Public capture routes<br/>/r/[code] · /f/[formId] · /embed · /g/[businessId]"]
    BizAPI["api/business/** (~60 routes)"]
    Hooks["api/webhooks/** (leads, green-api, sumit, payment)"]
    Cron["api/cron/automations"]
    Domain["lib/server/domain/*<br/>(server-only domain layer)"]
  end

  subgraph FB["Firebase"]
    FS["Firestore<br/>businesses/{id}/… subcollections"]
    Rules["Security Rules<br/>(tenant isolation)"]
    Storage["Storage / Google Drive<br/>(attachments)"]
  end

  Visitor -->|click / submit| Capture
  Providers -->|inbound webhooks| Hooks
  Owner -->|authenticated UI| MW --> Dash --> BizAPI
  Admin --> BizAPI
  Capture --> Domain
  BizAPI --> Domain
  Hooks --> Domain
  Cron --> Domain
  Domain -->|Admin SDK| FS
  Rules -.->|enforces isolation| FS
  Domain --> Storage
  Domain -->|encrypted creds| Providers
```

**Key idea:** the browser and public callers never touch Firestore directly for writes. Everything
funnels through the **server‑only domain layer** (`import "server-only"`), which derives the tenant
from the session and builds every path from `businessRef(businessId)`. Firestore security rules make
`businesses` **read‑only to clients** and lock sensitive subcollections entirely, so the rules are a
second wall behind the domain layer, not the primary one.

### Route groups (`app/`)

| Group | Purpose |
| --- | --- |
| `(auth)/` | `login`, `register`, `invite/[token]` |
| `(dashboard)/` | tenant app shell: `dashboard`, `leads`, `crm/*`, `campaigns`, `sources`, `tracking-links`, `reports`, `business-dashboard`, `communications`, `integrations`, `settings/*`, `billing`, `gift`, `automations` |
| `admin/` | platform‑operator console (`users`, `invitations`, `coupons`, `costs`, `logs`, `support`) |
| `api/business/**` · `api/admin/**` · `api/webhooks/**` | tenant API · platform API · inbound webhooks |
| root public | `r/[trackingCode]` (redirect), `f/[formId]` + `embed/[formId]` (forms), `g/[businessId]` (gift) |

`middleware.ts` is cookie‑only (no Firestore at the edge): it checks the `firebase-token` cookie for
presence to gate protected paths, handles demo mode, and confines pending/suspended users. It is
wrapped in try/catch that falls through to `NextResponse.next()` so a middleware crash can never take
down every page. **Real authorization happens server‑side in route handlers**, verified with
`verifySessionCookie(cookie, true)`.

---

## 2. Multi‑tenant data model

A tenant is a **business**. Every tenant entity is a subcollection under `businesses/{businessId}`; a
`users/{uid}` doc carries exactly one `businessId`, mirrored in `businesses/{id}/members/{uid}`.

```mermaid
erDiagram
  BUSINESS ||--o{ MEMBER : has
  BUSINESS ||--o{ CAMPAIGN : owns
  BUSINESS ||--o{ SOURCE : owns
  BUSINESS ||--o{ TRACKINGLINK : owns
  BUSINESS ||--o{ CLICK : logs
  BUSINESS ||--o{ LEAD : owns
  BUSINESS ||--o{ DEAL : owns
  BUSINESS ||--o{ EXPENSE : owns
  BUSINESS ||--o{ INTEGRATION : configures
  BUSINESS ||--o{ AUTOMATION : configures
  USER ||--|| BUSINESS : "belongs to (businessId)"
  CAMPAIGN ||--o{ TRACKINGLINK : groups
  SOURCE ||--o{ TRACKINGLINK : "advertised via"
  TRACKINGLINK ||--o{ CLICK : produces
  TRACKINGLINK ||--o{ LEAD : attributes
  LEAD ||--o{ TOUCHPOINT : accumulates
  LEAD ||--o{ DEAL : converts_to
  CAMPAIGN ||--o{ EXPENSE : "ad cost"

  BUSINESS {
    string id
    string ownerId
    string status
    object billing "plan, cycle, status, customerId"
    object enabledModules "crm, campaigns, finance, gift, insights"
  }
  USER {
    string uid
    string email
    string role "admin | user"
    string status "pending|active|inactive|suspended|invited"
    string businessId
    object permissions
  }
  LEAD {
    string id
    string normalizedPhone
    string normalizedEmail
    string firstSourceId "first-touch"
    string lastSourceId "last-touch"
    string attributionStatus "confirmed|manual_required|manual_confirmed|estimated"
    string attributionMethod
    string status
    int inquiryCount
  }
  TRACKINGLINK {
    string code
    string campaignId
    string sourceId
    string destinationType
    string webhookTokenHash
    int clickCount
    int leadCount
  }
  DEAL {
    string leadId
    string status "won|…"
    number amount
    string campaignId
    string sourceId
  }
```

**Identity index:** a top‑level `leadKeys/{kind:value}` collection maps `phone:…` / `email:…` → a
`leadId`, so the atomic upsert can de‑duplicate returning inquiries inside a transaction. Other
top‑level lookup/lock collections: `trackingLinkCodes`, `trackingLinkWebhookTokens`,
`paymentWebhookTokens`, `rateLimits`, `webhookEvents` (idempotency), `jobs` (deferred work),
`invites`, `adminActivityLogs`, `systemErrorLogs`. `firestore.indexes.json` defines 13 composite
indexes for the hot query paths (leads by status/updatedAt, deals by status/closedAt, messages by
leadId/createdAt, clicks by campaignId/createdAt, jobs by status/runAfter …).

> **Two type systems, one live.** `lib/types.ts` (a `userId`‑based scaffold model) is dead — only
> demo data and the KPI library import it. The live model is the untyped‑at‑rest, business‑scoped
> data constructed in `lib/server/domain/*.ts`.

---

## 3. Key flow — lead capture and attribution

```mermaid
sequenceDiagram
  actor V as Ad visitor
  participant R as /r/[code] (redirect)
  participant LP as Landing page / form
  participant WH as api/webhooks/leads
  participant TX as upsertLeadWithTouchpoint (Firestore txn)
  participant J as jobs (deferred)

  V->>R: click smart link (MAKOR=code)
  R->>R: rate-limit (5/min/IP+code), log click, increment count
  R-->>V: 302 → destinationUrl
  V->>LP: submit inquiry (form / WhatsApp / phone)
  LP->>WH: POST payload (JSON / urlencoded / multipart)
  WH->>WH: per-IP throttle → resolve token → business
  WH->>WH: flatten payload, match Heb/En field aliases, extract MAKOR
  WH->>TX: normalize phone+email, open transaction
  TX->>TX: read leadKeys(phone),(email) → existing? merge : new
  TX->>TX: preserve firstSource, advance lastSource, append touchpoint, claim keys
  TX-->>WH: leadId (attributionStatus: confirmed | manual_required)
  WH->>J: enqueue mailing-sync + automations
  WH-->>LP: 200 (never leaks lead existence)
  J->>J: cron drains jobs (exp. backoff)
```

If the `MAKOR` code resolves to a tracking link of the **same** business, the lead is
`attributionStatus: "confirmed"`, `attributionMethod: "tracking_link"`; otherwise `manual_required`,
surfaced in the UI as "דורש שיוך ידני". Side effects (email‑marketing sync, automations) are deferred
to the `jobs` collection so the webhook can answer immediately.

---

## 4. Key flow — billing webhook (defense in depth)

```mermaid
sequenceDiagram
  participant S as Sumit
  participant WH as api/webhooks/sumit
  participant D as withWebhookClaim (idempotency)
  participant B as business (Firestore)

  S->>WH: payment webhook (raw body)
  WH->>WH: verifySumitWebhook (shared secret OR HMAC, timingSafeEqual)
  WH->>WH: classifySumitStatus — negative patterns FIRST
  WH->>D: claim(externalId) via Firestore create() (ALREADY_EXISTS = dup)
  WH->>WH: amount reconciliation (>= expected − coupon − ₪1)
  alt paid, verified, sufficient
    WH->>B: activate / extend plan
  else underpaid / failed / duplicate
    WH-->>S: 200 (no plan change)
  end
```

`classifySumitStatus` checks failure patterns (`unpaid|invalid|declin|fail…`) *before* success ones,
so an "InvalidCard"/"Unpaid" status can never be misread as a payment. A charge below
`expected − allowed coupon discount − ₪1 tolerance` will not activate a plan.

---

## 5. Cross‑cutting concerns

```mermaid
flowchart LR
  A["Every route<br/>withApiHandler"] --> B["Sentry + systemErrorLogs<br/>generic Hebrew 500 (no leak)"]
  A --> C["Firestore rate limiter<br/>fixed-window, fail-open"]
  A --> D["requireActiveBusinessContext<br/>session → businessId → RBAC"]
  E["Credentials"] --> F["AES-256-GCM at rest<br/>(INTEGRATION_ENCRYPTION_KEY)"]
  G["Public IPs"] --> H["hashIp = sha256(salt : ip)"]
  I["Provider retries"] --> J["withWebhookClaim<br/>Firestore create() lock, 30-day TTL"]
```

- **Server/client split:** `firebase-admin`, gRPC/protobuf, `nodemailer`, `xlsx`, `jspdf`, `resend`,
  `isomorphic-dompurify`, `@sentry/node` are marked `serverComponentsExternalPackages` in
  `next.config.js`. `instrumentation.ts` validates env once per server start.
- **RBAC** (`lib/auth/permissions.ts`): 40+ `PermissionAction`s; every active member does daily work,
  an `OWNER_ONLY_BY_DEFAULT` set (deletes, exports, integrations, billing, members, automations) is
  owner‑only, per‑user `permissions` overrides, platform admin via custom claim.
- **Security headers** (`next.config.js`): HSTS, `X-Content-Type-Options`, `Referrer-Policy`,
  `Permissions-Policy`, `X-Frame-Options: DENY` + CSP `frame-ancestors 'none'` everywhere except
  `/embed/*` (intentionally embeddable).

---

## 6. Testing topology

```mermaid
graph LR
  U["Vitest unit<br/>server-only aliased to stub"] --> L["pure logic:<br/>KPI, metrics, encryption,<br/>Sumit parsing, permissions"]
  E["Emulator (firebase emulators:exec)"] --> R["firestore-rules.test<br/>(rules enforced)"]
  E --> T["tenant-isolation.test<br/>(real handlers vs emulator)"]
  P["Playwright e2e"] --> F["public flows:<br/>form, tracking-link,<br/>lead-webhook, rate-limit,<br/>security-headers"]
```

Roughly 30 `*.test.ts` files. The emulator suite runs **real App Router handlers** against a Firestore
emulator with only the session layer mocked, so cross‑tenant access is proven to fail at both the
rules layer and the handler layer.
