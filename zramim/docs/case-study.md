# Zramim — a studio-management platform with a self-service phone line

**Role:** Full-stack engineer & system designer (sole developer)
**Timeline:** ~3 weeks, ~85 commits
**Stack:** Next.js 15 · React 19 · TypeScript · Supabase (PostgreSQL, Auth, RLS) · Tailwind CSS · Yemot HaMashiach IVR · Nedarim Plus · Resend/SMTP · Vercel + GitHub Actions
**Domain:** SMB operations / vertical SaaS / telephony · Hebrew, RTL, mobile-first

---

## The problem

A swimming and water-aerobics studio ran its entire operation — hundreds of members across
two departments, weekly recurring classes, subscriptions, freezes, waitlists, debts, and
collections — out of a sprawl of Airtable bases (23 automations, 7 interfaces) plus manual
spreadsheets and phone calls. The setup had three chronic failures:

1. **The money never reconciled.** Payments were mis-allocated, applied twice, or showed
   phantom credit balances. Nobody could answer "how much has actually been earned this
   month vs. paid in advance?"
2. **The front desk was a bottleneck.** Every registration, freeze, class swap, and "how
   many lessons do I have left?" required a secretary on the phone during office hours.
3. **Data integrity was unenforced.** A subscription could exist without its classes booked;
   a class could be over-capacity; the same customer could be entered twice.

The client wanted one unified system, in Hebrew, that a non-technical manager and secretary
could run — and eventually, a phone line customers could serve themselves on.

---

## What I built

A single web application that runs the whole back office, plus an automated telephone system
that lets customers self-serve without staff.

### 1. Correct-by-construction domain model

The heart of the system is a relational model in PostgreSQL where the invariants are
enforced by the *database*, not by hopeful application code:

- **Atomic registration.** Buying a subscription creates the subscription, books it into the
  next *N* concrete class occurrences, and posts a charge to the financial ledger — all in a
  single transactional Postgres function (`register_subscription`). There is no code path
  that can leave a half-written state.
- **Capacity as an invariant.** A database trigger rejects any booking that would exceed a
  group's capacity, with a Hebrew error, unless a manager explicitly overrides it. The rule
  can't be forgotten by a screen.
- **Identity that matches reality.** Phone number is the lookup key but is deliberately *not*
  unique (sisters share a phone); a `(phone, full_name)` unique index prevents true
  duplicates while allowing families.

### 2. A ledger that always adds up

Money is tracked **at the customer level**, not per subscription — which is what eliminated
the original double-allocation bug. Overpayment automatically becomes a reusable **credit
balance**. On top of the raw cash ledger I built a **deferred-revenue** layer: a Postgres
snapshot function feeds a small, unit-tested TypeScript "waterfall" that allocates every
shekel a customer has paid into buckets — *earned* (lessons already delivered) → *deferred*
(future booked lessons) → *frozen* (held in a frozen subscription) → *credit* (true surplus)
— alongside a separately-computed *debt*. The buckets always sum to cash received, so the
manager can reconcile the system against the actual bank balance and see, per month, what's
been earned versus paid in advance.

### 3. A self-service telephone line

The standout piece: a full inbound IVR built on Yemot HaMashiach, with the entire menu logic
living in the Next.js app as a **stateless HTTP endpoint**. Callers can, without any staff:

- **Identify themselves** by caller-ID (with a name-picker when a phone maps to several
  people).
- **Check subscription status** and hear their remaining lessons and upcoming dates.
- **Register and pay** end-to-end: choose a subscription type and group, hear the terms and a
  health declaration (non-skippable), record their name, and clear a credit card through
  Yemot's PCI-handled Nedarim Plus extension — with the payment written straight into the
  ledger.
- **Swap a single class** under a 24-hour rule, **unfreeze** a frozen subscription, join a
  **waitlist**, or register via an **organized group code**.
- **Hear and acknowledge a debt** ("press 1 to confirm you heard this") — which then
  suppresses the automated escalation call.

All of it speaks natural Hebrew: static menus are hand-vocalized with *nikud* (vowel points),
and dynamic content (names, dates, amounts) is vocalized at runtime via lookup tables plus a
best-effort call to Dicta's Nakdan API — validated so it can never corrupt pronunciation.

### 4. Automation & multichannel notifications

An hourly, idempotent scheduler (a GitHub Actions cron hitting a secured dispatch endpoint)
fires timed jobs: debt alerts timed from a member's *first actual lesson* (not their signup
date), renewal reminders, a Thursday trials-and-expiries report, weekly debt summaries, a
monthly income statement, and freeze expiry. Each customer chooses their channel — email, a
free phone "tzintuk" ping, or a spoken IVR message — and every send is audited.

### 5. Role-based security, in depth

Two staff roles (manager / secretary) plus a hidden developer identity. Authorization is
enforced **twice**: at the database via Row-Level Security (a secretary literally cannot read
the expenses table), and again at the app layer via guarded server components. Financial
reports and global P&L are manager-only; technical logs are developer-only and
URL-guess-proof.

---

## Engineering decisions worth calling out

- **Adapter pattern for every uncertain integration.** The email provider was undecided, the
  payment merchant account wasn't issued yet, and phone clearing was a later phase. So email,
  payments, and TTS are each a runtime-selected interface with a safe default (`noop` email
  that never falsely reports "sent"; `manual` payments). The whole app runs end-to-end before
  any real integration lands, and swapping one in is a config change, not a rewrite.
- **Push logic into Postgres where atomicity matters.** Registration, freeze/unfreeze, group
  changes, and rescheduling are RPC functions so multi-row writes are transactional and
  capacity is enforced by trigger — not re-implemented per screen.
- **Timezone correctness as a first-class concern.** Every scheduling and day-bucketing
  decision uses `Intl.DateTimeFormat` pinned to `Asia/Jerusalem` (DST-aware), never
  server-local time — because "the next 4 Tuesdays, skipping holidays" has to be right across
  a daylight-saving boundary.
- **Fail-safe by default.** Technical logging never throws; config loaders degrade to
  defaults when a migration hasn't run; background work (emails, waitlist notifications) runs
  after the response so a phone caller is never left waiting.
- **Hebrew-first, but safe.** Every user- and manager-facing error is translated to Hebrew
  (raw English server text never leaks); template variables are HTML-escaped; emails and IVR
  text are RTL- and nikud-aware.

---

## Outcome

Zramim consolidated ~23 Airtable automations and 7 interfaces into one coherent system and
closed the specific correctness bugs (double-allocation, phantom credits, over-capacity,
duplicate customers) that motivated the rebuild. The manager configures providers, schedules,
email templates, and the entire phone menu from in-app settings screens — no code or vendor
console required. Coverage was tracked against a written spec in the repo's own audit doc; the
core (registration → scheduling → billing → dashboard), operations (freeze, waitlists, debts,
reports), and the full telephony layer are implemented.

---

## What I'd highlight in a conversation

> "Two things made this more than a CRUD app. First, the accounting: I modeled revenue two
> ways — a customer-level cash ledger and an accrual/deferred waterfall — so the numbers
> reconcile against the bank and the owner can see earned-vs-prepaid per month. Second, the
> phone line: Yemot's API is stateless and re-sends every prior keypress on each request, so I
> built a per-call navigation stack keyed by call-ID to give a stateless protocol real
> back-navigation, and handled Hebrew TTS with vowelization so dates and names are actually
> pronounceable. The rest of the discipline — transactional RPCs, RLS defense-in-depth,
> adapter-based integrations, Jerusalem-timezone correctness — is what kept it a system rather
> than a pile of screens."
