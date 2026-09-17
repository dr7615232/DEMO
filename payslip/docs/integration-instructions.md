# Integration Instructions — publishing this material

How to put this portfolio package online, **outside** the original application repository. Nothing
here touches, imports, or deploys the original app.

---

## 0. Ground rules

- This package lives in a **separate repo** (`dr7615232/DEMO`, folder `payslip/`). Keep it that way.
- Do the anonymization checklist in `README.md` before anything goes public.
- The two artifact links are **private** (they open when signed in to the owner's Claude account).
  For a fully public site, self-host the HTML instead of relying on the artifact links.

---

## 1. Fastest path — share the artifacts

- **Project page:** https://claude.ai/artifact/EhcaNPusrQKdZjJ5EkGEjD
- **Interactive demo:** https://claude.ai/artifact/JiYZ78FcF8Kb3GWsyr6YN1

Send the links, or embed them. To make an artifact viewable by anyone, open it and use its share menu.

---

## 2. Self-host the static pages (public, no build)

Both HTML files are standalone (fonts load from Google Fonts; no bundler).

```
your-portfolio/
  public/projects/payslip/site/index.html   ← copy of payslip/site/index.html
  public/projects/payslip/demo/index.html   ← copy of payslip/demo/index.html
```

- Serve them as static files (GitHub Pages, Netlify, Vercel static, Cloudflare Pages, S3 + CDN).
- If self-hosting the demo, update the demo links inside `site/index.html` (the `demobar` and footer
  `href`s) to point at your hosted `demo/index.html` instead of the artifact URL.
- Example (Netlify): drop the `public/` folder in, or `netlify deploy --dir=public`.

---

## 3. Embed in a Next.js / React portfolio

1. Copy `project-data.json` into your site (e.g. `content/payslip.json`).
2. Copy the components from `suggested-components.md` into `components/payslip/`.
3. Render a case-study page:

```tsx
import { PayslipHero, StackChips, Highlights, FindingPreview, DemoCallout } from "@/components/payslip";
import Markdown from "your-markdown-renderer";
import caseStudy from "@/content/payslip/case-study.md";

export default function PayslipCaseStudy() {
  return (
    <main className="mx-auto max-w-3xl px-6 py-12 space-y-10">
      <PayslipHero />
      <StackChips />
      <Highlights />
      <FindingPreview />
      <Markdown source={caseStudy} />
      <DemoCallout />
    </main>
  );
}
```

4. Add the metadata from `seo-metadata.md` to the page `<head>` (Open Graph, Twitter, JSON-LD).
5. Render `architecture.md` with a Markdown renderer that supports **Mermaid** (e.g. `rehype-mermaid`,
   or Mermaid's client script) so the diagrams display.

---

## 4. Generate an OG image (optional)

There is no bundled screenshot of real data. To make a social image:

- Screenshot the **demo** (example data only), or
- Render `assets/mark.svg` on the cream background (`#f3f4e7`) with the title text.

Save it where `seo-metadata.md` references it (`/og/payslip.png`) and keep meaningful `alt` text.

---

## 5. Keep it in sync

- The case study, architecture, and deep-dives are prose — safe to edit for tone/length.
- `project-data.json` is the machine-readable source of truth for chips/highlights; update it if the
  project evolves, and the data-driven components follow automatically.
- If you ever update the demo or project-page HTML in this package, re-publish the artifacts (or
  re-copy the files to your host) so the links stay current.

---

## 6. Do not

- Do not add these files to the original application repo.
- Do not publish a filled `.env`, real payslips, real names/IDs, or unverified legal numbers as fact.
- Do not present the tool's output as binding legal advice.
