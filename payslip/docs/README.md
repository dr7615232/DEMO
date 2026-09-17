# Payslip Check (בדיקת תלוש שכר) — Portfolio Package

> **What this is:** a self-contained portfolio kit extracted from the *Payslip Check*
> codebase. It is a **separate deliverable** — nothing here modifies, integrates into, or
> deploys the original application. Use it to build a case-study page, a résumé entry, or a
> talking-points sheet for interviews.

---

## The project in one paragraph

**Payslip Check** is a Hebrew, right-to-left (RTL) web application that tells a person whether
their Israeli payslip is correct. The user uploads a payslip (PDF or image), and optionally an
attendance report and an employment contract. The system extracts the data, shows it for
verification and correction, then runs a **rules engine grounded in Israeli labour law and
extension orders**. It returns a clear verdict — **valid** or **needs review** — with a list of
findings, each carrying what was found, what was expected, the gap, a legal source, a confidence
level, a plain-language explanation, and an **estimated monetary impact** (separated into
high-certainty amounts and estimates). It serves two audiences from one codebase: a private
employee checking their own slip, and a payroll professional running checks for clients.

- **Stack:** Next.js 14 (App Router, React 18) · TypeScript · Tailwind CSS · PostgreSQL (with a
  file-based fallback) · Supabase Auth · pdf-parse / tesseract.js / multi-provider AI extraction
  (Gemini · OpenAI · Anthropic) · Zod · Vitest · GitHub Actions.
- **Shape:** ~12,960 lines of TypeScript/TSX across 144 files · a **17-rule** labour-law engine ·
  18 pages · 21 API route handlers · dual storage backends behind one interface.

---

## Files in this package

| File | What it's for |
|------|---------------|
| `README.md` | This index + the anonymization checklist. |
| `case-study.md` | The main narrative case study — problem, solution, role, outcome. Drop-in copy for a portfolio page. |
| `architecture.md` | System architecture with Mermaid diagrams (system context, data model, key request flows). |
| `technical-deep-dives.md` | Four engineering deep-dives written for a technical reader (the rules engine, date-versioned legal parameters, structured AI extraction with cost logging, defense-in-depth privacy). |
| `code-snippets.md` | Curated, real code excerpts with commentary — the "show me the code" appendix. |
| `project-data.json` | Structured, machine-readable project metadata for a portfolio site/CMS. |
| `seo-metadata.md` | Titles, meta descriptions, Open Graph / Twitter tags, JSON-LD, and keywords. |
| `suggested-components.md` | Suggested portfolio-site UI components (React/Next) with example code to present this project. |
| `demo-instructions.md` | How to run/demo the original app locally, and what to show. |
| `integration-instructions.md` | Exact steps to publish this material to a portfolio site (kept **outside** the original repo). |
| `assets/mark.svg` | A neutral brand mark (the app's own ₪ tile) for cards and favicons. |

**Live links**

- 📄 Project page (Hebrew): https://claude.ai/artifact/EhcaNPusrQKdZjJ5EkGEjD
- 🧾 Interactive demo: https://claude.ai/artifact/JiYZ78FcF8Kb3GWsyr6YN1

*(The links are private — they open when you are signed in to your Claude account.)*

---

## Suggested one-liners (pick per surface)

- **Résumé bullet:** "Built a Hebrew-RTL payslip-auditing web app (Next.js 14 + TypeScript) with
  a 17-rule labour-law engine that returns per-finding legal sources, confidence levels, and
  estimated monetary gaps, plus date-versioned legal parameters and structured AI extraction with
  per-use cost logging."
- **Portfolio card subtitle:** "Full-stack payroll compliance: document extraction, a rules
  engine grounded in Israeli labour law, and clear plain-Hebrew findings."
- **Interview hook:** "The hard part wasn't the CRUD — it was making a compliance engine that
  says 'I can't determine this yet' and asks one good question, instead of confidently reporting a
  gap that isn't there."

---

## ⚠️ Before you publish — anonymization checklist

This tool processes real, sensitive payroll data. Before putting any of this online, review:

1. **No real payslips or people.** The demo and all examples use invented names
   (e.g. "מיכל לוי"), an invented employer ("מרכז ורדים בע\"מ"), and made-up amounts. Never
   publish a real person's payslip, name, national ID, or salary figures.
2. **Legal parameter values are illustrative.** Minimum wage, convalescence day value, pension
   rates, tax brackets, and travel caps change periodically and several are tagged
   medium/low confidence in the code. Do not present any number here as current legal fact.
3. **Secrets.** None are included. The original `.env.example` holds only empty placeholders
   (`DATABASE_URL`, `PAYSLIP_ENC_KEY`, Supabase keys, AI provider keys). Never publish a filled
   `.env`, and note that the admin email placeholder should be scrubbed before sharing.
4. **Screenshots.** No screenshots of real data are bundled. If you add any, use the demo (example
   data only), never a real check.
5. **Not legal advice.** Any public writeup should keep the app's own disclaimer: this is a
   preliminary check, not binding legal advice; values must be verified against official sources.
6. **Repository visibility.** The original repo is private by design. Decide deliberately whether
   to show code publicly or keep it to the snippets in `code-snippets.md`.

---

## Provenance & honesty note

All technical claims here were extracted directly from the codebase (`src/lib/engine`,
`src/lib/law`, `src/lib/extraction`, `src/lib/store`, `src/lib/security`, `src/app`) and the
project's own `README.md`. A few honest nuances worth keeping in any writeup:

- The **rules engine and law parameters are fully implemented in code** (17 rules, date-versioned
  constants, managed overrides).
- **AI OCR / structured extraction and the AI anomaly layer are multi-provider and real**, but
  depend on an admin-configured provider key; with no provider the app falls back gracefully to
  the heuristic parser and manual entry.
- Some **legal values are deliberately marked medium/low confidence** and are meant to be updated
  against official publications — the engine surfaces that uncertainty rather than hiding it.
- The **PostgreSQL backend targets cloud deployment**; local development defaults to the
  file-based store.
