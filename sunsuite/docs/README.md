# SUNSUITE — Portfolio Documentation Package

A production, Hebrew-RTL **booking & operations platform for a group of luxury
vacation-rental and spa complexes** (short-stay suites, a private spa rented by
the hour or overnight, and a Tiberias unit). Guests book online; the owner runs
the whole operation — approvals, cleaning, calendar, guest messaging — from a
single admin panel.

> ℹ️ These are portfolio materials describing a **real client system**. Screens,
> data and names in the demo and docs are **examples only**. Review the
> anonymization checklist below before making anything public.

---

## Contents

| File | What's inside |
|------|---------------|
| [`case-study.md`](./case-study.md) | The story: problem, solution, my role, outcome. |
| [`architecture.md`](./architecture.md) | System context, data model & key flows (Mermaid diagrams). |
| [`technical-deep-dives.md`](./technical-deep-dives.md) | 4 engineering deep-dives with real code. |
| [`code-snippets.md`](./code-snippets.md) | Selected, publish-safe code excerpts with commentary. |
| [`project-data.json`](./project-data.json) | Structured metadata (stack, features, highlights). |
| [`seo-metadata.md`](./seo-metadata.md) | Titles, descriptions, OG/Twitter tags, JSON-LD, keywords. |
| [`suggested-components.md`](./suggested-components.md) | React/Next components to feature this project on a portfolio site. |
| [`demo-instructions.md`](./demo-instructions.md) | How to run & present the interactive demo. |
| [`integration-instructions.md`](./integration-instructions.md) | How to publish these materials (outside the source repo). |
| [`assets/`](./assets/) | Logo and brand marks copied from the source repo. |

## Live materials

- **📄 Project page (Hebrew):** [`../site/index.html`](../site/index.html) — designed RTL case-study page.
- **🌞 Interactive UI demo:** [`../demo/index.html`](../demo/index.html) — faithful, self-contained reconstruction of the real screens (admin + public site), Hebrew RTL, example data.

Artifact links are listed in the top-level repository `README.md`.

---

## Stack at a glance

- **Frontend/SSR:** TanStack Start (React 19), TanStack Router (file-based), TanStack Query.
- **Styling:** Tailwind CSS v4 (oklch design tokens), shadcn/ui (Radix), Assistant + Rubik fonts, full RTL.
- **Backend:** Supabase — PostgreSQL, Row-Level Security, Realtime, Auth, Storage — accessed through TanStack **server functions** with Zod validation.
- **Integrations:** Google Calendar (two-way event sync), Cloudinary (image/video), Resend/SMTP (email), Hebcal (Shabbat times).
- **Hosting:** Cloudflare (Vite + `@cloudflare/vite-plugin`, `wrangler`).
- **Reports/exports:** Recharts, ExcelJS, jsPDF.

---

## Anonymization checklist (do before publishing publicly)

- [ ] Confirm the demo and docs contain **no real guest** names, phone numbers or emails (all example data uses `example.com`).
- [ ] Remove any real business phone, address, WhatsApp or booking domain from copy.
- [ ] Keep the description generic ("a group of vacation-rental and spa complexes"); avoid the client's marketing/legal name if it must stay private.
- [ ] Ensure no secrets are present: Supabase keys, Cloudinary/Google/Resend credentials, `.env` values. (None are included here — code snippets read from env only.)
- [ ] Double-check screenshots/recordings do not capture a real admin session.
- [ ] If publishing the source separately, verify no `service_role` key or Supabase project ref is committed.
