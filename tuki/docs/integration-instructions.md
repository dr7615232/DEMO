# Integration Instructions — publishing these materials

How to publish the תוכי פרינט case study. All files are static and self-contained.

## What you have

```
tuki/
├── site/index.html      designed Hebrew project page (standalone)
├── demo/index.html      interactive catalog-builder demo (standalone)
└── docs/                this documentation package + assets/
```

Both HTML files are fully self-contained (only web fonts are external). The logo is
embedded in the pages; a copy is in `docs/assets/logo.webp`.

## Option A — Claude Artifacts (already done)

- Project page: https://claude.ai/artifact/Q1NbhQTXLwFGPNzXxwSY2c
- Interactive demo: https://claude.ai/artifact/N6HuWafMG1kLRiFnvZsX6p

Artifacts are **private by default** — use the page's share menu to share a link. To
update, re-publish the same file.

## Option B — Host the static files anywhere

Any static host works (GitHub Pages, Netlify, Cloudflare Pages, Vercel static, S3):

- **GitHub Pages:** enable Pages on the repo; files are reachable at `/tuki/site/` and `/tuki/demo/`.
- **Netlify / Cloudflare Pages / Vercel:** point at the repo with **no build command** and
  the repo root as the publish directory.
- **Plain upload:** copy `site/` and `demo/` to any web root.

If you host the demo somewhere new, update the demo link inside `site/index.html`
(the demo bar and footer button) to your demo URL.

## Option C — Embed in an existing portfolio

See [`suggested-components.md`](./suggested-components.md) for ready React/Next
components (project card, demo `<iframe>`, feature grid, dual-CTA hero).

## Before publishing publicly

Run the anonymization checklist in [`README.md`](./README.md): confirm example data
only, decide whether the business name/logo may appear, and note there are no secrets
(the demo is client-side and calls nothing external).
