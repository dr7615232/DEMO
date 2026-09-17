# Suggested Components — featuring תוכי פרינט

Drop-in React/Next components to present this project. No UI library required; restyle
freely. Replace the Artifact URLs if you self-host.

## Constants

```tsx
export const TUKI = {
  name: "תוכי פרינט — בונה הקטלוג",
  tagline: "A self-updating product catalog builder for a print shop",
  demoUrl: "https://claude.ai/artifact/N6HuWafMG1kLRiFnvZsX6p",
  pageUrl: "https://claude.ai/artifact/Q1NbhQTXLwFGPNzXxwSY2c",
  stack: ["HTML", "CSS", "Vanilla JS", "RTL", "Offline"],
};
```

## 1. Project card

```tsx
export function TukiCard() {
  return (
    <a href={TUKI.pageUrl} target="_blank" rel="noopener"
       className="group block rounded-2xl border border-neutral-200 p-6 transition hover:shadow-lg">
      <div className="text-xs font-bold tracking-widest text-blue-600">CASE STUDY</div>
      <h3 className="mt-1 text-xl font-bold">{TUKI.name}</h3>
      <p className="mt-2 text-sm text-neutral-600">{TUKI.tagline}</p>
      <ul className="mt-4 flex flex-wrap gap-2">
        {TUKI.stack.map(s => (
          <li key={s} className="rounded-full border border-neutral-200 px-3 py-1 text-xs font-medium">{s}</li>
        ))}
      </ul>
      <span className="mt-4 inline-block text-sm font-semibold text-blue-600 group-hover:underline">
        View case study →
      </span>
    </a>
  );
}
```

## 2. Live demo embed

```tsx
export function TukiDemoEmbed() {
  return (
    <figure className="overflow-hidden rounded-2xl border border-neutral-200">
      <iframe src={TUKI.demoUrl} title="תוכי פרינט demo" loading="lazy" className="h-[720px] w-full" />
      <figcaption className="bg-neutral-50 px-4 py-2 text-xs text-neutral-500">
        Interactive demo — example data only.
      </figcaption>
    </figure>
  );
}
```

## 3. Feature grid (from project-data.json)

```tsx
import data from "@/data/tuki/project-data.json";

export function TukiFeatures() {
  return (
    <section>
      <h2 className="text-2xl font-bold">What it does</h2>
      <div className="mt-6 grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
        {data.features.map((f: string) => (
          <div key={f} className="rounded-xl border border-neutral-200 p-4 text-sm">
            <span className="mr-2 inline-block h-2 w-2 rounded-full bg-blue-500 align-middle" />
            {f}
          </div>
        ))}
      </div>
    </section>
  );
}
```

## 4. Dual-CTA hero

```tsx
export function TukiHero() {
  return (
    <header className="rounded-3xl bg-gradient-to-b from-blue-50 to-white p-10 text-center">
      <h1 className="text-4xl font-black">{TUKI.name}</h1>
      <p className="mx-auto mt-3 max-w-xl text-neutral-600">{TUKI.tagline}</p>
      <div className="mt-6 flex justify-center gap-3">
        <a href={TUKI.demoUrl} target="_blank" rel="noopener"
           className="rounded-full bg-blue-600 px-6 py-3 font-semibold text-white">▶ Live demo</a>
        <a href={TUKI.pageUrl} target="_blank" rel="noopener"
           className="rounded-full border border-neutral-300 px-6 py-3 font-semibold">Project page</a>
      </div>
    </header>
  );
}
```

> Copy `project-data.json` to your app (e.g. `src/data/tuki/`) so the imports resolve.
> Components are framework-agnostic React and work in Next.js (app or pages router).
