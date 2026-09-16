# Integration Instructions — publishing these portfolio materials

How to publish the SUNSUITE case study **outside** the original product repo. The
source app repository stays untouched; everything here lives in the separate
portfolio repo (`dr7615232/DEMO`) under `sunsuite/`.

> Do **not** commit these materials into the product's own repository, and do not
> run the product's dev/deploy from here. These are static, standalone files.

## What you have

```
sunsuite/
├── site/index.html      # designed Hebrew project page (standalone)
├── demo/index.html      # interactive UI demo (standalone) + logo.png
└── docs/                # this documentation package + assets/
```

Both HTML files are fully self-contained (only web fonts are external).

## Option A — Publish as Claude Artifacts (already done)

The project page and the demo are each published as an Artifact; the links are in
the top-level repo `README.md`. Artifacts are **private by default** — open the
page's share menu to share a link. To update, re-publish the same file.

## Option B — Host the static files anywhere

Any static host works (GitHub Pages, Netlify, Cloudflare Pages, Vercel static, S3):

- **GitHub Pages:** enable Pages on the portfolio repo; the files are reachable at
  `/sunsuite/site/` and `/sunsuite/demo/`.
- **Netlify / Cloudflare Pages / Vercel:** point the project at the repo with **no
  build command** and the repo root as the publish directory; the paths above work
  as-is.
- **Plain upload:** copy `site/` and `demo/` to any web root. Keep `logo.png` next
  to `demo/index.html`.

The demo links to the project page's demo button via the Artifact URL; if you host
the demo elsewhere, update the two `href`s in `site/index.html` to your demo URL.

## Option C — Embed in an existing portfolio site

See [`suggested-components.md`](./suggested-components.md) for ready React/Next
components. Simplest embed: an `<iframe>` to the demo, plus a project card linking
to the case study. For a framework build, drop the HTML files under `public/` and
link to them.

## Before publishing publicly

Run the anonymization checklist in [`README.md`](./README.md). In short: confirm
only example data is present, remove any real business contact details, and verify
no secrets or Supabase/Cloudinary/Google credentials appear anywhere.

## Keeping it fresh

- If the product UI changes materially, re-capture the relevant screens in
  `demo/index.html` and re-publish the Artifact (same URL).
- Update `project-data.json` when the stack or feature set changes; it's the single
  structured source other surfaces (cards, JSON-LD) can read from.
