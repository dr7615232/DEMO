# Integration Instructions — publishing this project

How to put this project on a portfolio site. Everything here stays **outside** the original
application repository; nothing in this package modifies or deploys the real app.

---

## What you have

```
wig-crm/
├── demo/index.html     # public-safe interactive reconstruction (example data)
├── site/index.html     # designed Hebrew project page (cream/olive/lime)
└── docs/               # this package
    ├── README.md
    ├── case-study.md
    ├── architecture.md          # Mermaid diagrams
    ├── technical-deep-dives.md
    ├── code-snippets.md
    ├── project-data.json        # machine-readable metadata
    ├── seo-metadata.md
    ├── suggested-components.md
    ├── demo-instructions.md
    ├── integration-instructions.md
    └── assets/                  # neutral mark for the portfolio (see note below)
```

Both `site/index.html` and `demo/index.html` are also published as private Claude Artifacts;
their URLs live in `project-data.json → links`.

---

## Option 1 — fastest: link the ready-made pages

Host `site/index.html` and `demo/index.html` as static files on any host (Netlify, Vercel,
GitHub Pages, Cloudflare Pages, or a plain folder). They have **no build step**. Then link
to the site page from your portfolio index. Done.

```bash
# example: publish the two static pages
cp -r wig-crm/site  /path/to/portfolio/public/projects/wig-crm-site
cp -r wig-crm/demo  /path/to/portfolio/public/projects/wig-crm-demo
```

## Option 2 — integrated: render from the data + components

1. Copy `project-data.json` into your content directory.
2. Copy the components from `suggested-components.md` into your app.
3. Copy the case study and deep-dives (Markdown) into your content pipeline (MDX, a CMS, or
   a Markdown loader). The Mermaid blocks in `architecture.md` render with `mermaid` or any
   MDX Mermaid plugin.
4. Embed the demo via the `WigCrmDemoFrame` iframe, or link out to it.

## SEO

Paste the tags from `seo-metadata.md` into the project page `<head>` and set the OG image.

---

## Mermaid rendering

`architecture.md` uses fenced ` ```mermaid ` blocks. To render:

- **GitHub** renders them natively in Markdown preview.
- **MDX / Next:** use `rehype-mermaid` or a client `mermaid.run()` call.
- **Static export:** pre-render with the Mermaid CLI: `mmdc -i architecture.md -o out.md`.

---

## ⚠️ Anonymization — do this before anything goes public

This project was built for a real client. Before publishing, confirm:

1. **No personal names.** The real product's logo and default configuration contain the
   studio owner's personal name. **That logo is deliberately not included** in this package;
   the portfolio pages use a neutral wordmark instead. Do not re-introduce the client's name
   or logo without written permission.
2. **Example data only.** The demo ships with invented customers and obviously-fake phone
   numbers (`050-000-00xx`). Never replace them with real records.
3. **No real contact endpoints.** Do not publish the client's real phone number, email, or
   domain.
4. **No secrets.** There are none in this package, and the app itself has no keys — keep it
   that way.
5. **Repository visibility.** Decide deliberately whether to show code publicly or keep it to
   the snippets in `code-snippets.md`. The original file remains private.
