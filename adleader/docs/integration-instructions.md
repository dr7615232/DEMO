# Integration / Publishing Instructions — AdLeader

How to publish these portfolio assets **outside** the original product repository. The source repo
stays private and untouched; everything here lives in the separate `dr7615232/DEMO` portfolio repo.

---

## What you have

```
adleader/
├── site/index.html      ← Hebrew project page (self-contained)
├── demo/
│   ├── index.html       ← interactive demo (self-contained)
│   ├── adleader-logo.png
│   └── adleader-icon.png
└── docs/                ← this package
    ├── README.md, case-study.md, architecture.md, technical-deep-dives.md,
    ├── code-snippets.md, project-data.json, seo-metadata.md,
    ├── suggested-components.md, demo-instructions.md, integration-instructions.md
    └── assets/          ← logo + brand assets
```

Both HTML files are standalone (no build step). The demo references its two PNGs by relative path, so
keep them beside `demo/index.html`.

---

## Option A — static hosting (recommended)

Any static host works (Vercel, Netlify, GitHub Pages, Cloudflare Pages, S3 + CloudFront).

1. Publish the `adleader/site/` and `adleader/demo/` folders as static assets.
2. Wire routes, e.g. `/projects/adleader` → `site/index.html`, `/projects/adleader/demo` →
   `demo/index.html`.
3. Update the demo/project links in `site/index.html` (the `demobar` and footer `href`) and in
   `project-data.json` if you host your own copies instead of the Claude Artifacts.

Vercel example (`vercel.json`, static):

```json
{
  "cleanUrls": true,
  "rewrites": [
    { "source": "/projects/adleader", "destination": "/adleader/site/index.html" },
    { "source": "/projects/adleader/demo", "destination": "/adleader/demo/index.html" }
  ]
}
```

## Option B — embed on an existing portfolio (Next.js / React)

1. Copy `site/` and `demo/` into your app's `public/` folder.
2. Use the components in [`suggested-components.md`](./suggested-components.md); point `DemoEmbed` at
   `/adleader/demo/index.html` (or the Artifact URL).
3. Add the metadata from [`seo-metadata.md`](./seo-metadata.md) to the project page.

## Option C — keep the Claude Artifacts as the canonical live links

The demo and project page are already published as private Claude Artifacts:

- Demo: `https://claude.ai/artifact/PHTUdUubAyg8sJgdJpPFsg`
- Project page: `https://claude.ai/artifact/5N3aefwnPYKhAHpGjB3FuZ`

Share them from the Artifact's share menu when you want them public. Until then they open only for
your signed‑in Claude account.

---

## Before you publish — checklist

- [ ] Run the **de‑identification checklist** in [`README.md`](./README.md). All names/phones/emails
      are invented; confirm nothing maps to a real person or client.
- [ ] Do **not** publish or link the source repository. The only outbound links are the demo and the
      project page.
- [ ] Keep the "תצוגת דמו · נתונים לדוגמה" badge visible in any screenshot or embed.
- [ ] If you name the client, get their explicit approval first; otherwise keep it generic.
- [ ] Update `{DEMO_URL}` / `{PROJECT_PAGE_URL}` placeholders in the sample components with your final
      URLs.
- [ ] Regenerate the OG image (`seo-metadata.md`) with your name/branding.

## Do NOT

- Do not copy any `.env` values, keys, tokens, or credentials from the product into these assets — the
  code snippets here are pure logic and contain none; keep it that way.
- Do not deploy the actual product from this package. This is a portfolio presentation of it, not a
  runnable copy of the app.
