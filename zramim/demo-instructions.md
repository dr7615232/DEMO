# Zramim — Demo & Run Instructions

How to run the original app locally to capture screenshots or record a walkthrough for the
portfolio. **Use seeded demo data only — never real customer data in anything you publish.**

> These steps run the *original* project. They do not belong in this portfolio package and
> nothing here changes the deployed app; they're a reference for producing demo material.

---

## Prerequisites

- Node.js ≥ 20
- A Supabase project (the free tier is enough for a demo) **or** the Supabase CLI for a local
  stack
- The original repo checked out

## 1. Install & configure

```bash
npm install
cp .env.example .env.local
```

Fill only what a local demo needs in `.env.local`:

- `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`
- Leave `EMAIL_PROVIDER=noop` and `PAYMENT_PROVIDER=manual` (safe defaults — nothing is sent
  or charged)
- Telephony/cron secrets can stay empty for a UI demo

## 2. Apply the database schema

Run the migrations against your Supabase project (via the Supabase CLI `db push`, or by
executing `supabase/schema.sql`, which is the consolidated dump of all migrations). Then seed:

- `supabase/seed-demo.sql` — demo pools, groups, subscription types, and a few fake customers
- `supabase/first-manager.sql` — bootstrap a manager account (follow the file's notes to link
  it to a Supabase Auth user)

## 3. Run

```bash
npm run dev        # http://localhost:3000
npm run typecheck  # optional: verify the TypeScript build
```

Log in as the seeded manager.

---

## A good demo path (what to show)

1. **Class-grid dashboard** — enrolled / waiting / free seats per class, the two-department
   switch (swimming ↔ water-aerobics).
2. **Register a customer → subscription** — show "save & pay," then open the customer card and
   point out the auto-created upcoming lessons and the ledger charge.
3. **Capacity block** — try to over-fill a group; show the Hebrew rejection and the manager
   override.
4. **Freeze / unfreeze** — freeze a subscription (future lessons disappear, balance held),
   then unfreeze into a chosen group.
5. **Debts screen** — show per-customer balance, a credit balance from an overpayment, and the
   collection-call logging fields.
6. **Finances (manager only)** — the income view; then log in as a **secretary** and show that
   Finances/Logs/Settings are gone (RLS + guards).
7. **Settings** — email templates, report recipients, automation schedules, and the **phone**
   settings screen (menu text, hours, recordings) — all editable without touching code.

## Demoing the IVR without a phone

The repo ships a simulator that drives the IVR endpoint over HTTP with no telephony account:

```bash
node scripts/ivr-sim.mjs
```

It walks ~11 scenarios (main menu → registration → terms → health → payment, back-navigation,
group code `2048`, voicemail, multi-name identification) and prints the Hebrew responses. This
is the safest way to show the phone logic in a screen recording. A short terminal capture of
the simulator makes a compelling "the phone system is real" clip.

---

## Capturing assets safely

- Use only seeded/fake names. Blur or replace anything that looks real.
- Record at mobile width too — the app is mobile-first RTL; showing both widths is a plus.
- For an OG/social image, prefer the logo on a brand gradient over a live screenshot (see
  `seo-metadata.md`).
