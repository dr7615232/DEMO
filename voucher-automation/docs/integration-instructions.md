# Integration Instructions — Publishing these materials

How to put the project page, the demo, and the docs onto a portfolio site. None of this
touches any original client system — these are standalone portfolio assets.

---

## What you have

```
voucher-automation/
├── site/index.html      # designed Hebrew project page (no build)
├── demo/index.html      # interactive demo (no build)
└── docs/                # this documentation package
    ├── assets/logo.svg  # project logo/wordmark
    └── *.md, project-data.json
```

Both HTML files are **self-contained**: open by double-click, host by copying the file.

---

## Option A — Host the two HTML files as-is (simplest)

Any static host (a `public/` folder, GitHub Pages, Netlify, Vercel static, S3 + CloudFront):

1. Copy `site/index.html` to e.g. `/projects/voucher-automation/index.html`.
2. Copy `demo/index.html` to e.g. `/projects/voucher-automation/demo/index.html`.
3. In `site/index.html`, the demo button points at the published Artifact URL. To point it
   at your self-hosted demo instead, replace the Artifact URL with your demo's path
   (there are two occurrences — the top demo bar and the footer button).

That's it. No build, no dependencies to install.

---

## Option B — Rebuild the page as components (React/Next portfolio)

Use the ready components in [`suggested-components.md`](./suggested-components.md):

1. Drop `ProjectHero`, `NumberedSection`, `FeatureList`, `DemoCallout`, `VoucherCard` into
   your component tree.
2. Feed copy from [`project-data.json`](./project-data.json) so the page and your project
   index share one source of truth.
3. Link the demo button to your hosted `demo/index.html` (or the Artifact).
4. Add the metadata from [`seo-metadata.md`](./seo-metadata.md) to the page `<head>`.

---

## Wiring the demo link

The project page references the demo in two places. Search `site/index.html` for the
Artifact URL and replace both with your chosen target:

```
https://claude.ai/artifact/…   →   /projects/voucher-automation/demo/  (or your URL)
```

---

## SEO / social

- Paste the tags from [`seo-metadata.md`](./seo-metadata.md) into the page head.
- Generate a 1200×630 OG image (art-direction note is in that file) and host it at the
  `og:image` URL. Use example data only.
- Set the canonical URL to the page's real address.

---

## Before you publish — safety checklist

Run the anonymization checklist in [`README.md`](./README.md). In short:

- [ ] No real donor names, emails, partner businesses, amounts, or voucher numbers.
- [ ] No secrets, tokens, SMTP credentials, connection strings, or internal hostnames.
- [ ] No real client brand, logo-with-a-name, phone number, or domain.
- [ ] The "תצוגת דמו · נתונים לדוגמה" tag stays visible on the demo.

---

## What NOT to do

- **Don't** point the demo or page at any real production system, inbox, or database.
- **Don't** embed real screenshots without re-running the anonymization checklist.
- **Don't** copy any file back into an original client repository — these live only in the
  portfolio repo.
