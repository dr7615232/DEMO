# Suggested Components — featuring SUNSUITE on a portfolio site

Drop-in React/Next components to present this project. They read from
[`project-data.json`](./project-data.json) where practical, use no UI library, and
are safe to restyle. Replace the Artifact URLs with your own if you self-host.

## Constants

```tsx
export const SUNSUITE = {
  name: "SUNSUITE",
  tagline: "Vacation-rental & spa booking and operations platform",
  demoUrl: "https://claude.ai/artifact/CvWsa7cPV7KZwcPJvUrez1",
  pageUrl: "https://claude.ai/artifact/U5ice3pCdtNUEsU4LsTT63",
  stack: ["TanStack Start", "React 19", "Supabase", "TypeScript", "Tailwind", "Cloudflare"],
};
```

## 1. Project card

```tsx
export function SunsuiteCard() {
  return (
    <a
      href={SUNSUITE.pageUrl}
      target="_blank"
      rel="noopener"
      className="group block rounded-2xl border border-neutral-200 p-6 transition hover:shadow-lg"
    >
      <div className="text-xs font-bold tracking-widest text-amber-700">CASE STUDY</div>
      <h3 className="mt-1 text-xl font-bold">{SUNSUITE.name}</h3>
      <p className="mt-2 text-sm text-neutral-600">{SUNSUITE.tagline}</p>
      <ul className="mt-4 flex flex-wrap gap-2">
        {SUNSUITE.stack.map((s) => (
          <li key={s} className="rounded-full border border-neutral-200 px-3 py-1 text-xs font-medium">
            {s}
          </li>
        ))}
      </ul>
      <span className="mt-4 inline-block text-sm font-semibold text-amber-700 group-hover:underline">
        View case study →
      </span>
    </a>
  );
}
```

## 2. Live demo embed

```tsx
export function SunsuiteDemoEmbed() {
  return (
    <figure className="overflow-hidden rounded-2xl border border-neutral-200">
      <iframe
        src={SUNSUITE.demoUrl}
        title="SUNSUITE interactive demo"
        loading="lazy"
        className="h-[720px] w-full"
      />
      <figcaption className="bg-neutral-50 px-4 py-2 text-xs text-neutral-500">
        Interactive demo — example data only.
      </figcaption>
    </figure>
  );
}
```

## 3. Feature grid (from project-data.json)

```tsx
import data from "@/data/sunsuite/project-data.json";

export function SunsuiteFeatures() {
  return (
    <section>
      <h2 className="text-2xl font-bold">What it does</h2>
      <div className="mt-6 grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
        {data.features.map((f: string) => (
          <div key={f} className="rounded-xl border border-neutral-200 p-4 text-sm">
            <span className="mr-2 inline-block h-2 w-2 rounded-full bg-lime-400 align-middle" />
            {f}
          </div>
        ))}
      </div>
    </section>
  );
}
```

## 4. Highlights strip

```tsx
import data from "@/data/sunsuite/project-data.json";

export function SunsuiteHighlights() {
  return (
    <ul className="space-y-3">
      {data.highlights.map((h: string) => (
        <li key={h} className="flex gap-3">
          <span aria-hidden className="mt-1 text-amber-600">◆</span>
          <span className="text-neutral-700">{h}</span>
        </li>
      ))}
    </ul>
  );
}
```

## 5. Case-study hero (dual CTA)

```tsx
export function SunsuiteHero() {
  return (
    <header className="rounded-3xl bg-gradient-to-b from-amber-50 to-white p-10 text-center">
      <h1 className="text-4xl font-black">{SUNSUITE.name}</h1>
      <p className="mx-auto mt-3 max-w-xl text-neutral-600">{SUNSUITE.tagline}</p>
      <div className="mt-6 flex justify-center gap-3">
        <a href={SUNSUITE.demoUrl} target="_blank" rel="noopener"
           className="rounded-full bg-amber-500 px-6 py-3 font-semibold text-white">
          ▶ Live demo
        </a>
        <a href={SUNSUITE.pageUrl} target="_blank" rel="noopener"
           className="rounded-full border border-neutral-300 px-6 py-3 font-semibold">
          Project page
        </a>
      </div>
    </header>
  );
}
```

> Copy `project-data.json` to your app (e.g. `src/data/sunsuite/`) so the imports
> above resolve. All components are framework-agnostic React and work in Next.js
> (app or pages router) as client or server components.
