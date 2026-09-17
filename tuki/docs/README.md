# תוכי פרינט — בונה הקטלוג · Portfolio Documentation

A Hebrew-RTL **product-catalog builder** for a print business. From one working
screen the owner picks which of hundreds of products go into this season's
catalog; the system assembles a designed catalog with a festive cover, the right
per-unit and bulk prices, and a client order form — ready to send in a few clicks.

> ℹ️ These are portfolio materials. This package is grounded in the **interactive
> demo** (`../demo/index.html`) and the **project page** (`../site/index.html`) —
> both self-contained HTML files. Data shown (products, prices, contact) is
> **example only**. See the anonymization checklist below before publishing.

## Contents

| File | What's inside |
|------|---------------|
| [`case-study.md`](./case-study.md) | The story: problem, before, solution, automation, result. |
| [`architecture.md`](./architecture.md) | Screen flow & product data model (Mermaid). |
| [`technical-deep-dives.md`](./technical-deep-dives.md) | 4 engineering notes with real demo code. |
| [`code-snippets.md`](./code-snippets.md) | Selected, publish-safe excerpts from the demo. |
| [`project-data.json`](./project-data.json) | Structured metadata (features, screens, highlights). |
| [`seo-metadata.md`](./seo-metadata.md) | Titles, descriptions, OG/Twitter, JSON-LD, keywords. |
| [`suggested-components.md`](./suggested-components.md) | React/Next components to feature the project. |
| [`demo-instructions.md`](./demo-instructions.md) | How to run & present the demo. |
| [`integration-instructions.md`](./integration-instructions.md) | How to publish these materials. |
| [`assets/`](./assets/) | Logo extracted from the demo. |

## Live materials

- **📄 Project page:** https://claude.ai/artifact/Q1NbhQTXLwFGPNzXxwSY2c
- **🦜 Interactive demo:** https://claude.ai/artifact/N6HuWafMG1kLRiFnvZsX6p
- In-repo: [`../site/index.html`](../site/index.html) · [`../demo/index.html`](../demo/index.html)

## Stack (as delivered)

- **Interactive demo:** a single, self-contained **HTML + CSS + vanilla JavaScript**
  file — no build, no framework, no server. Fonts from Google Fonts (Rubik).
- Hebrew RTL throughout; dates shown in both Gregorian and Hebrew (`Intl` Hebrew calendar).
- The demo holds state **in memory** (it is a faithful UI reconstruction). A production
  build would add persistence and catalog/PDF export; those are out of scope for the demo
  and are not claimed here.

## Anonymization checklist (before publishing publicly)

- [ ] Confirm products, prices, phone and email are example data (`example.com`). No real customer/order data.
- [ ] Replace or confirm the business name/logo if it must stay private (the demo embeds the brand logo).
- [ ] No secrets are present — the demo is client-side only and calls no external services.
