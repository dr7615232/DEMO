# Zramim — Portfolio Package

> **What this is:** a self-contained portfolio kit extracted from the *Zramim* (זרמים)
> codebase. It is a **separate deliverable** — nothing here modifies, integrates into, or
> deploys the original application. Use it to build a case-study page, a résumé entry, or a
> talking-points sheet for interviews.

---

## The project in one paragraph

**Zramim** is a production, single-tenant management system for an Israeli swimming &
water-aerobics studio. It replaces a tangle of Airtable bases and manual spreadsheets with
one Hebrew, right-to-left (RTL), mobile-first web app that runs the whole back office:
customer records, subscriptions and automatic class scheduling, a customer-level financial
ledger with deferred-revenue accounting, freeze/unfreeze, waitlists, capacity control,
role-based staff access, scheduled email/phone automations, and a **full self-service
telephone (IVR) line** built on the Israeli *Yemot HaMashiach* platform — including
phone-based registration, credit-card clearing, and debt collection with Hebrew
text-to-speech.

- **Stack:** Next.js 15 (App Router, React 19, Server Actions) · Supabase (PostgreSQL +
  Auth + Row-Level Security) · TypeScript · Tailwind CSS · Yemot HaMashiach (telephony) ·
  Nedarim Plus (payments) · Resend/SMTP (email) · hosted on Vercel/Render, scheduled via
  GitHub Actions.
- **Scale:** ~26,500 lines of TypeScript/TSX across 204 files · 71 SQL migrations
  (~1,850 lines) defining ~24 tables, ~15 Postgres functions, ~48 RLS policies · built
  over ~3 weeks.

---

## Files in this package

| File | What it's for |
|------|---------------|
| `README.md` | This index. |
| `case-study.md` | The main narrative case study — problem, solution, role, outcomes. Drop-in copy for a portfolio page. |
| `architecture.md` | System architecture with Mermaid diagrams (system context, data model, request flows). |
| `technical-deep-dives.md` | Four engineering deep-dives written for a technical reader (atomic registration, deferred-revenue ledger, the stateless IVR protocol, defense-in-depth security). |
| `code-snippets.md` | Curated, real code excerpts with commentary — the "show me the code" appendix. |
| `project-data.json` | Structured, machine-readable project metadata for a portfolio site/CMS. |
| `seo-metadata.md` | Titles, meta descriptions, Open Graph tags, JSON-LD, and keywords. |
| `suggested-components.md` | Suggested portfolio-site UI components (React/Next) with example code to present this project. |
| `demo-instructions.md` | How to run/demo the original app locally and what to show. |
| `integration-instructions.md` | Exact steps to publish this material to a portfolio site (kept **outside** the original repo). |
| `assets/logo.png` / `assets/logo.jpg` | The Zramim brand logo (water/petal theme). |

---

## Suggested one-liners (pick per surface)

- **Résumé bullet:** "Built a production, Hebrew-RTL studio-management platform (Next.js 15 +
  Supabase) with atomic transactional subscription registration, a deferred-revenue ledger,
  and a full self-service IVR phone line (Yemot HaMashiach) supporting phone registration,
  card clearing, and automated Hebrew-TTS debt collection."
- **Portfolio card subtitle:** "Full-stack SaaS + telephony: subscriptions, ledger
  accounting, and a self-service Hebrew phone system."
- **Interview hook:** "The interesting part wasn't the CRUD — it was making a *stateless*
  phone menu behave like a stateful app, and getting the money to always add up to the cash
  in the bank."

---

## ⚠️ Before you publish — anonymization checklist

This system was built for a real client and handles real customer data. Before putting any
of this online, review:

1. **Client / individual names.** The source docs mention the studio owner and staff by
   first name. This package deliberately refers only to "the studio" / "the client." Keep it
   that way, or get written permission to name them.
2. **The live phone number & domain.** Source docs reference a real phone line
   (`03-3090061`) and account details. Do **not** publish live contact endpoints.
3. **Screenshots.** No screenshots are bundled (the app requires auth + real data). If you
   add them, use seeded demo data (`supabase/seed-demo.sql`) — never real customers.
4. **Secrets.** None are included here. The original `.env.example` contains only empty
   placeholders; never publish a filled `.env`.
5. **Repository visibility.** The original repo is private by design (the developer retains
   code ownership per the project's account-ownership plan). Decide deliberately whether to
   show code publicly or keep it to snippets like those in `code-snippets.md`.

---

## Provenance & honesty note

All technical claims in this package were extracted directly from the codebase (migrations,
`lib/`, `app/`, and the project's own `docs/`). Where the app's own coverage audit
(`docs/coverage-audit.md`) marks a requirement as partial or deferred to a later phase, this
package does not claim it as shipped. A few honest nuances worth keeping in any writeup:

- The telephony (IVR) subsystem is **fully implemented in code**, though the project's
  planning docs were written as "phase 3" and lag the implementation.
- The Nedarim Plus payment provider is a **graceful stub** (interface + config ready, live
  API wiring pending the client's merchant credentials); office payments are recorded
  manually and phone card-clearing runs through Yemot's native credit-card extension.
- Revenue recognition is modeled two ways (cash ledger + accrual/deferred) precisely because
  the client needed the numbers to reconcile against the bank — that reconciliation story is
  a strength, not a gap.
