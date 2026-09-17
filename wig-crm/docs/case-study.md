# Wig CRM (אומנות בפאות) — Case Study

> A production, Hebrew-RTL business-management system for a wig studio, delivered as a
> **single self-contained HTML file** that runs offline by double-click. No install, no
> server, no internet, no dependencies.

---

## The client & the problem

The client runs a wig studio: she sells custom and second-hand wigs, and does styling and
"upgrade" work (recolouring, lengthening). It is a small, high-touch business where each
wig can be worth thousands of shekels, is bought from a supplier at a real cost, and is
often paid for in installments.

Before this system, the whole operation lived in her head, in notebooks, and over the
phone: who is coming and when, who already paid, how much is still owed, what a wig cost
from the supplier, and how much was actually left at the end of the month. Every simple
question — *"how much did I make this month?"*, *"how much do I still owe the supplier?"* —
meant sitting down and recomputing from scratch. It was easy to miss a payment or forget to
call a customer back.

She is not a technical user. She needed something that just **opens and works**, in Hebrew,
with nothing to maintain.

---

## The solution

One tidy place for the whole business, delivered as a single HTML file she double-clicks to
open. It works entirely offline and keeps all of her data on her own computer. Eight
screens cover the business end to end:

- **Dashboard** — today's appointments, open tasks, due reminders, and this month's profit.
- **Calendar** — a month grid of appointments and events.
- **Clients** — contacts with statuses, groups, and birthdays; import from a spreadsheet,
  export to Excel.
- **Tasks** — manual and auto-generated to-dos.
- **Deals** — a simple five-stage inquiry/sales pipeline.
- **Finances** — four tabs: an at-a-glance overview (income, supplier cost, other expenses,
  net profit, open debts, a six-month trend, and a by-service breakdown), a full
  transactions list, a six-month forecast of upcoming credit-card and supplier payments, and
  a rolling account per supplier.
- **Settings** — the price list, payment methods, groups, statuses, suppliers, automations,
  and branding, all editable in-app.
- **Client card** — per-customer visit history, payments, and a running balance.

### The core idea: the money adds up by itself

The heart of the system is one rule that keeps the numbers honest:

> **Appointment** = what was sold · **Payment** = money in · **Expense** = money out ·
> **Supplier** = what is owed to them.

The important design decision is that a product's **cost** and its **supplier** are attached
to the *sub-service* in the price list, not to the payment. So the moment a sale is recorded
on an appointment, three things happen on their own:

1. The **true profit** of that sale is known (charge minus cost), and stays correct even if
   the customer only pays part of it now.
2. A **charge is auto-posted to that supplier's rolling account**, so "what the supplier is
   owed minus what I already paid" is always right, with nothing typed in by hand.
3. The customer only ever sees the **price**. The cost fields live in a hidden internal area,
   so the screen she can turn toward the customer never reveals what a wig cost the studio.

### Working while she's busy

A small rule engine creates reminders on its own: the day before a styling appointment it
adds a "wash the wig" reminder, and a few days before a customer's birthday it adds a
"send a greeting" reminder. Each reminder is created exactly once, so re-opening the app
never spams duplicates.

### Booking in two natural steps

Appointments match how the studio actually works. On the phone she books a day, a customer,
and just a **service** ("styling"). When the customer arrives, she opens the appointment and
marks the exact **sub-service** that was done, and the price and cost flow in automatically.

---

## My role

Sole developer and system designer. I did the discovery with the client, modelled the
business rules, designed the rose-gold Hebrew RTL interface, and built the entire
application — front end, data model, business logic, storage, import/export, and the
automation engine — plus a branded operating guide for the client.

---

## Engineering approach & constraints

The defining constraint was **zero infrastructure**. The client should never have to install
anything, log in, pay for hosting, or worry about an internet connection or a service going
down. That led to a deliberate architecture:

- **One file, no build, no dependencies.** The whole app — HTML, CSS, ~1,950 lines of
  vanilla JavaScript, and even the logo (base64-inlined) — is one ~300 KB file. There is no
  framework and no bundler; screens are rendered with plain template strings and a tiny
  view router.
- **Offline-first storage.** State is a single JSON document written to `localStorage` on
  every change. For users who want a real backup file, an optional mode uses the browser's
  File System Access API to mirror every change into a chosen file, with the file handle
  preserved across sessions in IndexedDB.
- **Deterministic money.** The supplier ledger is *regenerated* from appointments rather than
  edited in place: saving an appointment deletes and rebuilds that appointment's supplier
  charges, so the books can never drift out of sync with what was actually sold.

A mandatory pre-delivery check runs the single script through a syntax check and a headless
simulation of every screen and the changed flow, and the file is only delivered at zero
errors.

---

## Outcome

The studio now runs from one place. The owner can answer *"how much did I make this month,
who still owes me, and how much do I owe the supplier?"* at a glance instead of recomputing
it by hand, and reminders make sure a customer is never forgotten. The deliverable is
durable by design: because it is a single dependency-free file, it will keep working exactly
the same in years to come, offline, with nothing to maintain.

---

## Honesty & provenance note

Every technical claim here was taken directly from the delivered source file (its data
model, business-logic functions, storage code, and automation engine). Where the project's
own notes list open ideas — for example, aligning revenue recognized on the *payment* date
with cost recorded on the *sale* date, or an option for a full reset that also clears the
price list — this write-up does not present them as shipped.
