# שובר אוטומטי (Voucher Automation) — Documentation Package

A portfolio documentation package for **שובר אוטומטי**, an automation that turns an
incoming donation into a personalized, uniquely numbered voucher and emails it to the
donor — with zero manual work — while watching voucher stock and alerting the operator
before anything runs out.

> **Note on scope & sourcing.** This project was reconstructed from the operator's own
> description of the workflow (the manual process that existed before, and the automated
> flow that replaced it), not from a shared source repository. The architecture, data
> model, and code in this package are a **representative reference implementation** of that
> described workflow, written to portfolio quality. They illustrate a faithful, buildable
> version of the system rather than transcribing a specific existing codebase. Where a
> capability is illustrative rather than confirmed-in-production, it is labeled as such.

---

## What the system does (one paragraph)

Before the automation, every donation was handled by hand: a donation-confirmation email
arrived, the operator copied the donor's details one by one into a document, typed a unique
voucher number, and emailed the finished voucher back. After the automation, the donation
details flow straight into the system, the donor receives a short form to choose **where**
they want their voucher, and the system generates a designed voucher with a unique number
and emails it automatically. Stock per redemption location is tracked continuously, and the
operator is alerted well before any location runs low. One screen shows the whole picture.

---

## Package index

| File | What's inside |
|------|---------------|
| [`case-study.md`](./case-study.md) | The story: problem, solution, role, and result. |
| [`architecture.md`](./architecture.md) | System context, data model, and key flows (Mermaid diagrams). |
| [`technical-deep-dives.md`](./technical-deep-dives.md) | 4 engineering deep-dives with reference code. |
| [`code-snippets.md`](./code-snippets.md) | Selected, publish-safe code excerpts with commentary. |
| [`project-data.json`](./project-data.json) | Structured project metadata (stack, features, highlights). |
| [`seo-metadata.md`](./seo-metadata.md) | Titles, descriptions, OG/Twitter tags, JSON-LD, keywords. |
| [`suggested-components.md`](./suggested-components.md) | Ready-to-use React/Next components for a portfolio site. |
| [`demo-instructions.md`](./demo-instructions.md) | How to open and present the interactive demo. |
| [`integration-instructions.md`](./integration-instructions.md) | How to publish these materials on a portfolio site. |
| [`assets/`](./assets/) | Logo and graphic assets used by the page and demo. |

**Live portfolio materials**

- 📄 Project page (Hebrew): [`../site/index.html`](../site/index.html)
- 🎟️ Interactive demo (Hebrew, RTL): [`../demo/index.html`](../demo/index.html)

---

## Anonymization checklist (read before publishing)

These materials are already written with **example data only**. Before making anything
public, confirm:

- [ ] **No real donor names.** All donors in the demo and docs are invented (e.g. "יעל אזולאי", "משפחת ברזני").
- [ ] **No real email addresses.** Addresses are masked (`yael•••@mail.com`) and fictional.
- [ ] **No real redemption partners.** Location names ("סופר השכונה", "ביתן הספרים") are generic placeholders, not real businesses.
- [ ] **No real voucher numbers.** The sequence shown (100476–100483) is illustrative.
- [ ] **No secrets.** No API keys, SMTP credentials, tokens, connection strings, or internal hostnames appear anywhere.
- [ ] **No client identity.** The business is referred to generically ("the organization / operator"); no real brand, logo with a personal name, phone number, or domain is included.
- [ ] **Amounts are illustrative.** Donation sums and stock counts are examples, not real figures.

If you add real screenshots or data later, re-run this checklist first.
