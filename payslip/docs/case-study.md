# Case Study — Payslip Check (בדיקת תלוש שכר)

## Summary

Payslip Check is a Hebrew, RTL web application that answers a question almost everyone has and
almost nobody can answer for themselves: **is my payslip correct?** A user uploads a payslip
(PDF or image), optionally an attendance report and an employment contract, and within a few
minutes gets a clear verdict — *valid* or *needs review* — with a list of findings, each written
in plain Hebrew, backed by a legal source, and quantified with an estimated shekel impact.

It is built as one codebase serving two audiences: a **private employee** checking their own slip,
and a **payroll professional** running checks for clients with reusable per-client setting cards.

---

## The problem

Israeli payslips are dense and legally intricate. Minimum wage, overtime premiums, pension and
severance contributions, convalescence pay (דמי הבראה), annual leave and sick-day accrual, travel
reimbursement, and dozens of deductions all follow rules from separate laws and extension orders,
several of which change value most years. The result:

- **Employees can't self-check.** The document is opaque; small underpayments accumulate silently.
- **Professionals do it by hand.** Payroll consultants and bookkeepers audit slips manually,
  which is slow and inconsistent across reviewers.
- **Naïve automation is dangerous.** A tool that confidently flags a "violation" from incomplete
  data does real harm — it sends people into a needless confrontation with an employer.

The design brief followed from that last point: automate the audit **without** manufacturing false
positives, and make every result explainable.

---

## The solution

A four-step flow that a non-expert can complete on their own:

1. **Upload.** Payslip is required; attendance and contract are optional and improve accuracy.
   There is no upfront questionnaire.
2. **Verify.** The system extracts the payslip into structured fields and shows them for review.
   Every field and every pay/deduction line can be corrected before anything is judged — so
   heuristic extraction errors never silently corrupt a check.
3. **Follow-up questions.** *Only after* the first analysis, and *only when* something is genuinely
   missing, the engine asks one or two targeted questions (e.g. "what is your hourly rate?").
4. **Results.** A verdict, a findings count, the key findings, and the total estimated monetary
   gap — with a detailed, printable report one click away.

The heart of the product is the **rules engine**. Each labour-law topic is an independent rule
module (17 of them). Every rule receives the same context — the payslip, optional attendance,
answers so far, the payslip month, and the resolved legal parameters — and returns *findings* and,
when needed, *follow-up questions*. Every finding is a rich object: **what was found, what was
expected, the gap, a legal source, a confidence level, a plain explanation, and an estimated
monetary impact** split into *high-certainty* versus *estimate*.

Two decisions make the tool trustworthy rather than merely clever:

- **"Cannot determine" is a first-class outcome.** When a rule lacks the data to judge, it returns
  an informational finding and asks a question, instead of asserting an error.
- **The law is versioned and dated.** Minimum wage, convalescence day value, pension rates, and
  travel caps each carry an `effectiveFrom` date and a confidence tag; the engine picks the value
  valid for the payslip's month and marks estimates accordingly. An admin can update these values
  in-app, overriding the code constants without a redeploy.

Around the engine sits the rest of a real application: multi-provider document extraction
(text-layer PDF parsing, local OCR, and structured AI extraction that returns typed JSON with
per-use cost logging), a printable per-category report, check history with search, multi-month
comparison with trends, personal and per-client setting cards, and an admin area with user
management, an audit log, managed legal parameters, and a usage-and-cost dashboard.

---

## My role

Sole designer and full-stack engineer. Responsible for the product concept, the domain modelling of
Israeli labour law, the extensible engine architecture, the extraction pipeline and its graceful
degradation, the storage abstraction (file and PostgreSQL), the security model (encryption at rest,
owner isolation, ID masking), the auth and role model, and the entire Hebrew RTL interface.

---

## Selected engineering highlights

- **An engine that grows by addition.** Adding a new labour-law check is one new file in
  `engine/rules/` plus one line in the registry — no change to the core runner. Seventeen rules
  ship today.
- **Separation of law from logic.** Deterministic, date-versioned legal parameters live apart from
  the rules that consume them, and can be overridden by admin-managed values at runtime.
- **Honest uncertainty.** Confidence levels flow from the legal data all the way to the UI badges,
  and missing data produces questions, not false alarms.
- **Structured AI extraction with cost accountability.** When a provider is configured, the payslip
  is extracted directly as typed JSON (far more accurate than text parsing), and each call records
  which model ran and what it cost, surfaced in the admin usage screen.
- **Runtime-selected storage behind one interface.** Presence of `DATABASE_URL` switches the whole
  app between a file store and a PostgreSQL store (checks as JSONB, documents as encrypted BYTEA)
  with no other code change.
- **Privacy treated as a requirement, not a feature.** Documents are encrypted at rest with
  AES-256-GCM, every check is owner-isolated, national IDs are masked in list views, and a
  retention policy underpins deletion.

---

## Outcome

The result is a tool that turns a gut feeling into a clear picture: what is owed, what is worth
checking, and how much money is involved — in plain Hebrew, without the user needing to understand
the law. For professionals it standardises an audit that used to vary by reviewer; for employees it
makes a check they would otherwise skip take minutes. Just as important, it is built to be
*right about being unsure*: it declines to guess, cites its sources, and shows its confidence, which
is what makes an automated legal check safe to act on.

> **Disclaimer preserved from the product:** the system provides a preliminary check and is not
> binding legal advice. Legal values change over time and must be verified against official sources
> before acting.
