# Case Study — AdLeader

*Multi‑channel lead attribution and advertising P&L for small businesses, in Hebrew.*

---

## The problem

A small business owner advertises in several places at once: Google search, a banner on a news
site, WhatsApp communities, an email list, sometimes a printed ad. Inquiries arrive through just as
many doors — a form, a WhatsApp message, an email, a phone call. At the end of the month the owner
knows how much was **spent**, but not which advertising actually **worked**. The single most
important question — *"where did this customer come from?"* — usually has no reliable answer, and so
budget keeps flowing to channels that may never pay for themselves.

Two failures compound each other:

1. **Lost inquiries.** Leads scattered across channels get written on sticky notes, or simply
   forgotten. Some are never followed up.
2. **Blind spend.** Because inquiries aren't tied back to their source, there is no honest return‑on
   ‑ad‑spend per channel. Decisions are made on gut feeling and vanity metrics (clicks), not profit.

## The solution

AdLeader is one calm, Hebrew‑first place that collects **every** inquiry from **every** channel and
**attributes it, automatically, to the source and campaign that produced it.** Each place you
advertise gets its own smart tracking link; every click and every inquiry that arrives through it is
bound to that source with no manual work. Ad cost attaches to the same source, deals attach to the
lead, and the system computes a real **profit‑and‑loss per source and per campaign** — how many
inquiries it produced, how many closed, and how many shekels it returned for each shekel invested.

Around that core sit the everyday tools a non‑technical owner needs: a leads inbox, a drag‑and‑drop
follow‑up pipeline, WhatsApp + email conversations attached to each lead's card, automations that
make sure nothing falls through the cracks, and reports that compare channels by **real return, not
clicks.**

## My role

**Sole full‑stack engineer and system designer.** Greenfield build, end to end:

- Product and information architecture for a non‑technical Hebrew‑speaking audience (RTL throughout).
- **Multi‑tenant backend** on Firebase: a server‑only domain layer where every entity is scoped under
  `businesses/{businessId}`, with authorization derived from the session, never from client input.
- The **lead‑capture and attribution pipeline**: tracking‑link redirect + click logging, a public
  lead webhook that parses arbitrary form‑builder payloads, hosted/embedded forms, and a **race‑safe
  atomic lead upsert** that de‑duplicates a returning inquiry across phone and email while preserving
  first‑touch and advancing last‑touch attribution.
- The **reporting engine** (lead/deal/expense P&L, per‑source and per‑campaign) and a separate,
  fully unit‑tested ads‑KPI library (ROAS/ROI/CPL/CTR + five attribution models).
- **Security**: AES‑256‑GCM encryption of all third‑party credentials, Firestore security rules with
  tenant isolation, hashed webhook‑token lookups with timing‑safe comparison, payment‑webhook amount
  reconciliation, Firestore‑backed rate limiting and webhook idempotency, IP hashing.
- **Integrations**: WhatsApp (Green API), Gmail OAuth inbound/outbound, Resend, a pluggable Hebrew
  email‑marketing provider registry, Google Drive attachment storage, and Sumit billing.
- **Testing**: Vitest unit tests, Firestore‑emulator rules + route‑level tenant‑isolation tests, and
  Playwright e2e for the public capture flows.

## The result

The owner stops advertising in the dark. Every inquiry lands in one place and is tied to the ad that
produced it, so no lead is lost and each channel earns an honest verdict — grow it, keep it, or stop
it. Advertising spend shifts from what *feels* like it works to what *provably* returns. The demo in
this package shows the real screens with example data; the interactive reconstruction lets you click
through the whole story in a browser.

## Selected engineering highlights

- **Race‑safe lead identity resolution.** A single Firestore transaction de‑duplicates a returning
  inquiry across normalized phone/email via a `leadKeys` index, preserves first‑touch while advancing
  last‑touch, appends an immutable touchpoint, and flags re‑inquiries for a lead already in the CRM.
- **Attribution at capture time.** A `MAKOR` tracking code on the inbound payload resolves to a
  tracking link of the same business → the lead is marked `confirmed` / `tracking_link`; otherwise
  `manual_required`. The public response never leaks whether a lead exists.
- **Defense‑in‑depth on money‑touching webhooks.** Token‑resolves‑business‑before‑any‑write,
  idempotency claims, a fail‑open Firestore rate limiter, negative‑status‑first parsing, and an
  amount‑floor reconciliation (coupon‑aware) so an underpaid or failed charge can never activate a
  plan.
- **Stateless‑serverless‑aware primitives.** Rate limiter and idempotency store live in Firestore
  (no Redis); side effects are deferred to a `jobs` collection drained by a cron with exponential
  backoff, so webhooks answer instantly and nothing is lost.
- **Tenant isolation proven, not just asserted.** Two independent test layers guard the same
  property: Firestore security rules *and* real App Router handlers run against the emulator.

*Stack: Next.js 14 · TypeScript · Firebase (Firestore + Admin SDK) · Tailwind v3 (RTL) · next‑intl ·
Zod · TanStack Query/Table · Recharts · Vitest · Playwright.*
