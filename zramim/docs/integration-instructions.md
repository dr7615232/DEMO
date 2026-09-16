# Zramim — Integration Instructions (publishing this to a portfolio site)

How to turn this package into a live case-study page — **in a separate portfolio project**,
never inside the original Zramim repo. Everything below assumes a repo/site that is *not* the
application.

---

## Golden rule

> The original Zramim repository stays untouched. Copy from this package **into your portfolio
> project**. Do not add portfolio files, routes, or assets to the application repo, and do not
> import the app's code into your site.

---

## Option A — Next.js portfolio (recommended)

1. In your portfolio project, create a route: `app/work/zramim/page.tsx`.
2. Copy `project-data.json` into your site (e.g. `content/zramim.json`) and import it.
3. Paste the components from `suggested-components.md` into
   `app/work/zramim/_components/` and compose them on the page.
4. Add the `metadata` export from `seo-metadata.md` and the JSON-LD `<script>`.
5. Copy `assets/logo.png` to `public/images/zramim-logo.png`; generate a 1200×630
   `zramim-og.png` for social (see `seo-metadata.md`).
6. Render the narrative: either paste `case-study.md` through your MDX pipeline, or map the
   `highlights` / `features` arrays from the JSON into the feature grid.
7. Embed the Mermaid diagrams from `architecture.md` (if your site renders Mermaid) or export
   them to SVG and drop them in as images.

**Minimal page skeleton:**

```tsx
// app/work/zramim/page.tsx
import data from "@/content/zramim.json";
import { ProjectHero, MetricsStrip, FeatureGrid, StackList, DeepDives } from "./_components";

export { metadata } from "./metadata"; // from seo-metadata.md

export default function ZramimCaseStudy() {
  return (
    <main className="mx-auto max-w-4xl space-y-12 px-4 py-12">
      <ProjectHero />
      <MetricsStrip />
      <section><h2 className="mb-4 text-2xl font-bold text-[#1B6E9B]">What it does</h2><FeatureGrid /></section>
      <section><h2 className="mb-4 text-2xl font-bold text-[#1B6E9B]">Stack</h2><StackList /></section>
      <section><h2 className="mb-4 text-2xl font-bold text-[#1B6E9B]">Engineering deep-dives</h2><DeepDives /></section>
    </main>
  );
}
```

## Option B — Markdown/MDX site (Astro, Eleventy, Hugo, etc.)

1. Copy `case-study.md` into your content collection (add the front-matter your site expects —
   title, date, tags, cover image).
2. Copy `assets/logo.png` into your static/images folder.
3. Paste the diagrams from `architecture.md` as Mermaid code fences (most MD site generators
   support them via a plugin) or as exported SVGs.
4. Add the meta tags / JSON-LD from `seo-metadata.md` to the page head.

## Option C — Résumé / one-pager (no site)

Use the bullet and one-liners in `README.md` and `case-study.md`. The metrics in
`project-data.json` (`26.5k lines`, `71 migrations`, `~48 RLS policies`) make strong résumé
figures.

---

## Rendering the Mermaid diagrams

If your target doesn't render Mermaid, export each diagram to SVG once:

```bash
npm i -g @mermaid-js/mermaid-cli
# paste one diagram into diagram.mmd, then:
mmdc -i diagram.mmd -o zramim-architecture.svg
```

Then reference the SVGs as images.

---

## Pre-publish checklist (repeat of README, because it matters)

- [ ] No individual/client names (keep it "the studio" / "the client").
- [ ] No live phone number, domain, or contact endpoint.
- [ ] Screenshots use seeded demo data only — never real customers.
- [ ] No filled `.env` / secrets anywhere.
- [ ] You've decided deliberately whether code is public (snippets from `code-snippets.md` are
      safe) or private.
- [ ] Diagrams and metrics match the honest scope (see the provenance note in `README.md`).

---

## What NOT to do

- ❌ Add any of these files to the original Zramim repo.
- ❌ Deploy the portfolio from the application's hosting/workspace.
- ❌ Point the portfolio at the production Supabase project or reuse its keys.
- ❌ Publish real customer data, voicemails, or the live phone line.
