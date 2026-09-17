# AdLeader — Portfolio Documentation Package

**AdLeader** (internal codename *CampaignIQ*) is a production, Hebrew‑RTL, multi‑tenant SaaS that
answers one question every advertiser keeps asking and can rarely prove: *which ad actually brought
this customer?* It captures every inquiry from every channel (form, WhatsApp, email, tracking link,
phone), attributes it to the source and campaign that produced it, and turns raw spend into a
per‑source profit‑and‑loss so the owner can see the real return on each shekel of advertising.

Built solo as a greenfield **Next.js 14 (App Router) + Firebase (Firestore/Admin SDK)** application,
with a server‑only domain layer, race‑safe lead de‑duplication, AES‑256‑GCM credential encryption,
Firestore‑backed rate limiting and idempotency, and a two‑layer tenant‑isolation test suite.

> This folder is the **technical** package for a portfolio. The client‑facing story lives in the
> Hebrew project page (`../site/`) and the interactive demo (`../demo/`).

---

## What's in here

| File | What it is |
| --- | --- |
| [`case-study.md`](./case-study.md) | The narrative: problem, solution, my role, result. |
| [`architecture.md`](./architecture.md) | System design with Mermaid diagrams (context, data model, key flows). |
| [`technical-deep-dives.md`](./technical-deep-dives.md) | Four engineering deep‑dives with real code. |
| [`code-snippets.md`](./code-snippets.md) | Selected, publish‑safe code excerpts with commentary. |
| [`project-data.json`](./project-data.json) | Structured project metadata (stack, features, highlights). |
| [`seo-metadata.md`](./seo-metadata.md) | Titles, descriptions, OG/Twitter tags, JSON‑LD, keywords. |
| [`suggested-components.md`](./suggested-components.md) | Ready React/Next components to feature the project on a portfolio site. |
| [`demo-instructions.md`](./demo-instructions.md) | How to open and present the interactive demo. |
| [`integration-instructions.md`](./integration-instructions.md) | How to publish these assets, outside the original repo. |
| [`assets/`](./assets/) | Logo and brand assets, copied from the project (not generated). |

Every technical claim in this package was verified against the actual source code. Where the
project is a scaffold, planned, or an implemented‑but‑not‑yet‑wired library, it is marked as such and
never presented as shipped behaviour.

---

## Accuracy notes (so the write‑up stays honest)

- The project's own `README` still calls parts of it a *"scaffold with placeholder Firestore
  wiring."* That language is **outdated** — the live domain layer (`lib/server/domain/*`) is fully
  implemented and covered by unit, emulator, and e2e tests.
- A polished, unit‑tested **multi‑touch attribution library** exists (`lib/utils/kpiCalculations.ts`,
  five models: first/last click, linear, position‑based, time‑decay). It is an **implemented library
  that is not yet wired into the live flow.** The running attribution model is **first‑touch /
  last‑touch, stored on the lead at write time**; production reporting uses the lead/deal/expense
  P&L engine in `lib/calculations/metrics.ts`. This distinction is preserved throughout.
- A **demo mode** (`/demo/*`, `campaigniq-demo` cookie) runs real screens on fixed demo data by
  design — that is what the interactive demo reconstructs.

---

## De‑identification checklist (run before publishing anything from this package)

This is a real client system. Before any asset here goes public, confirm:

- [ ] **No real customer PII.** Names, phone numbers, emails, and addresses in the demo and docs are
      **invented examples** (e.g. "סטודיו אור", "דנה לוי", `052‑111‑2233`, `*@example.com`). Verify
      none map to a real person or business.
- [ ] **No secrets.** No API keys, tokens, `INTEGRATION_ENCRYPTION_KEY`, `WEBHOOK_SECRET`,
      `CRON_SECRET`, Sumit/Resend/Google credentials, service‑account private keys, or `.env` values.
      The code snippets here are pure logic and contain none.
- [ ] **No real endpoints or IDs.** No production hostnames, Firebase project IDs, real webhook URLs,
      business IDs, or payment identifiers. Demo links use `example.com` / `adleader.local`.
- [ ] **No real client identity** unless the client has explicitly approved being named. Refer to the
      customer generically ("a small business / studio / course business").
- [ ] **Screenshots** show demo data only, with the "תצוגת דמו · נתונים לדוגמה" badge visible.
- [ ] **Repository stays private.** The link set here points to the demo and project page, never to
      the source repository.

When in doubt, leave it out.
