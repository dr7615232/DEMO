# Wig CRM (אומנות בפאות) — Portfolio Package

> **What this is:** a self-contained portfolio kit describing the *Wig CRM* project. It is a
> **separate deliverable** — nothing here modifies, integrates into, or deploys the original
> application. Use it to build a case-study page, a résumé entry, or interview talking points.

---

## The project in one paragraph

**Wig CRM (אומנות בפאות)** is a production, single-tenant business-management system for an
Israeli wig studio, delivered as a **single, self-contained HTML file** that runs offline by
double-click — no install, no server, no internet, and zero dependencies. In one Hebrew,
right-to-left interface it runs the whole business: clients, a two-stage appointment
calendar, tasks, a sales-inquiry pipeline, and a finance suite. Its heart is the money
model: a product's cost and supplier are attached to the price list, so every sale
automatically computes true profit, auto-posts a charge to that supplier's self-reconciling
ledger, and never shows the cost on the screen the customer can see. A small idempotent
rule engine creates wash and birthday reminders on its own.

- **Stack:** Vanilla JavaScript (ES2019+), HTML5, CSS3 — **no framework, no build step** ·
  `localStorage` + File System Access API (optional real-file sync) + IndexedDB (file
  handle) · base64-inlined assets · fully offline.
- **Shape:** one ~300 KB HTML file · 2,495 lines total · ~1,950 lines of JavaScript · 228
  functions · 396 CSS rule blocks · 8 screens · 10 data collections.

---

## Files in this package

| File | What it's for |
|------|---------------|
| `README.md` | This index + the anonymization checklist. |
| `case-study.md` | The main narrative — problem, solution, role, outcome. Drop-in copy for a portfolio page. |
| `architecture.md` | System architecture with Mermaid diagrams (context, data model, money flow, rendering, persistence). |
| `technical-deep-dives.md` | Four engineering deep-dives with real code (supplier ledger, cost-on-sub-service, the idempotent reminder engine, File System Access persistence). |
| `code-snippets.md` | Curated real code excerpts with commentary. |
| `project-data.json` | Structured, machine-readable project metadata for a portfolio site/CMS. |
| `seo-metadata.md` | Titles, meta descriptions, Open Graph/Twitter tags, JSON-LD, keywords. |
| `suggested-components.md` | React/Next components (reading `project-data.json`) to present the project. |
| `demo-instructions.md` | How to run/demo the reconstruction and the real app safely. |
| `integration-instructions.md` | Exact steps to publish this material, kept **outside** the original repo. |
| `assets/mark.svg` | A **neutral** portfolio mark (the client's real logo is deliberately excluded — see below). |

**Live pages:** the designed Hebrew project page (`../site/index.html`) and the interactive
demo (`../demo/index.html`) are also published as private Claude Artifacts — URLs are in
`project-data.json → links`.

---

## Suggested one-liners (pick per surface)

- **Résumé bullet:** "Delivered a production Hebrew-RTL business-management system for a wig
  studio as a single, dependency-free, offline HTML file — with a self-reconciling supplier
  ledger, partial-payment-proof profit accounting, and an idempotent in-browser reminder
  engine."
- **Portfolio card subtitle:** "A whole business in one offline HTML file: clients,
  appointments, supplier ledger, automatic profit."
- **Interview hook:** "The constraint was zero infrastructure — no server, no install, no
  internet — so the interesting problems were making the money reconcile by itself and giving
  a static file a real, durable data story."

---

## ⚠️ Before you publish — anonymization checklist

This system was built for a real client and handles real customer data. Before putting any of
this online, review:

1. **Client / individual names.** The real product's default configuration and **logo**
   contain the studio owner's personal name. This package refers only to "the studio" / "the
   owner", the portfolio pages use a **neutral wordmark**, and the client's logo is
   **deliberately not bundled**. Do not re-introduce the name or logo without written
   permission.
2. **Example data only.** The demo ships with invented customers and obviously-fake phone
   numbers (`050-000-00xx`). Never swap in real records for a screenshot or recording.
3. **No live contact endpoints.** Do not publish the client's real phone number, email, or
   domain.
4. **Secrets.** There are none here, and the app has no API keys or backend — keep it that way.
5. **Repository visibility.** The original file stays private. Decide deliberately whether to
   show code publicly or keep it to the snippets in `code-snippets.md`.

---

## Provenance & honesty note

Every technical claim in this package was extracted directly from the delivered source file
(its data model, business-logic functions, storage code, and automation engine) and its
accompanying README. Honest nuances kept in the write-up:

- Period **income is counted on the payment date** while **cost of goods is on the sale
  date**, so a sale in one month paid the next can skew a single month's profit. It is
  documented as an open business decision (cash vs. accrual), not presented as solved.
- The current **reset** keeps the price list and settings (`config`) and clears only records;
  a full reset that also wipes the price list is listed as a future idea, not a shipped
  feature.
- The optional real-file sync uses the **File System Access API**, available in
  Chromium-based browsers; everywhere else the app falls back to `localStorage`, which is
  always on.
