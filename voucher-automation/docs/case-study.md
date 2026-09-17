# Case Study — שובר אוטומטי

*Turning a manual, copy-paste voucher process into a hands-off automation.*

> Sourced from the operator's description of the workflow. Details are presented at the
> level of the process; specifics (names, amounts, partner businesses) are illustrative.

---

## The client & the context

A donation-driven organization gives every donor a **thank-you voucher** redeemable at a
partner business the donor chooses (a neighborhood supermarket, a bookshop, a pharmacy, a
café, and so on). Each voucher carries a **unique number** and is personalized to the donor.

Vouchers are a lovely gesture — but at any real volume, producing them by hand becomes a
bottleneck that scales linearly with generosity: the more people give, the more manual work
piles up.

---

## The problem — before the automation

Every single donation triggered the same manual chain:

1. A **donation-confirmation email** landed in the operator's inbox.
2. The operator **opened it and copied** the donor's details — name, amount, contact — one field at a time.
3. They **opened a document** (a word processor), pasted the details, and laid out a voucher.
4. They **typed a unique voucher number** by hand, taking care not to reuse one.
5. They **emailed the finished voucher** back to the donor.

Repeated dozens of times, this had three costs:

- **Time.** Minutes per donor, multiplied across every donation, every day.
- **Errors.** Manual copying invites typos in names, amounts, and — most damaging — duplicate or skipped voucher numbers.
- **Delay.** The donor waited until a human got around to it, which softened the moment of goodwill.

And stock was invisible: the operator only discovered a partner location had "run out" of
allocated vouchers when it was already a problem.

---

## The solution — after the automation

A single system now closes the entire loop on its own:

- **Donation details flow straight in.** No copy-paste from the inbox. The confirmation
  becomes a structured record automatically.
- **The donor chooses.** They receive a short, friendly form and pick **where** to redeem
  their voucher.
- **The voucher generates itself.** The system produces a designed voucher with a **unique,
  never-repeating number**, personalized to the donor.
- **It's emailed instantly.** The donor gets their voucher within seconds — no human in the loop.
- **Stock updates itself.** The chosen location's inventory decrements automatically.
- **The operator is warned early.** When any location's stock nears its threshold, an alert
  goes out well before it runs out, so replenishment happens ahead of time and the automatic
  flow never stalls.
- **One screen, full picture.** Donations in, vouchers out, stock levels, and open alerts —
  all visible at a glance.

---

## The result

| Before | After |
|--------|-------|
| Minutes of copy-paste per donor | Seconds, fully automatic |
| Voucher numbers typed by hand (risk of duplicates) | Unique numbers assigned by the system, guaranteed |
| Donor waits for a human | Donor receives the voucher within seconds |
| Stock discovered empty after the fact | Early low-stock alerts, ahead of time |
| Details scattered across inbox and documents | One dashboard with the whole activity |

What used to be a long, repetitive manual task per donor now happens by itself. Fewer
mistakes, an instant response to the donor, and full control over the whole operation —
without touching a thing.

---

## My role

- Mapped the manual process end to end and identified where automation removes work without
  removing control.
- Designed the donation → form → voucher → email flow, including the **unique numbering** and
  **automatic stock decrement** as a single atomic step.
- Built the **low-stock alerting** so the automation never silently stalls.
- Designed the operator dashboard so the whole operation is legible from one screen, and the
  donor-facing form so choosing a voucher is effortless.

## What this reference package demonstrates

A faithful, buildable version of the described system: a clean data model for donations,
vouchers, and per-location stock; an atomic "issue voucher" operation that can't produce a
duplicate number or oversell stock; and a threshold-based alerting design. See
[`architecture.md`](./architecture.md) and [`technical-deep-dives.md`](./technical-deep-dives.md).
