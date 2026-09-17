# Suggested Portfolio Components — Wig CRM (אומנות בפאות)

Drop-in React/Next components to present this project on a portfolio site. They read from
`project-data.json`. Styling is intentionally minimal (Tailwind classes + the project's
rose-gold accents) so you can adapt to your own design system.

Accent tokens used below:

```css
--wig-plum:#5E4642;  --wig-rose:#C88E7F;  --wig-rose-wash:#F6E8E2;  --wig-bg:#FAF5F2;
```

---

## 1. Project hero card

```tsx
import data from "@/content/wig-crm/project-data.json";

export function WigCrmHero() {
  return (
    <section dir="rtl" className="rounded-2xl bg-[#FAF5F2] p-8 ring-1 ring-[#EDE1DB]">
      <p className="text-sm font-semibold tracking-widest text-[#A18A83]">
        {data.nameHebrew} · {data.year}
      </p>
      <h1 className="mt-2 text-3xl font-bold text-[#48342F]">{data.tagline}</h1>
      <p className="mt-3 max-w-2xl text-[#5f5048]">{data.oneLiner}</p>
      <div className="mt-5 flex flex-wrap gap-2">
        {data.stack.storage.map((s) => (
          <span key={s} className="rounded-full bg-[#F6E8E2] px-3 py-1 text-sm text-[#5E4642]">{s}</span>
        ))}
      </div>
      <a href={data.links.liveDemo} target="_blank" rel="noopener"
         className="mt-6 inline-flex items-center gap-2 rounded-lg bg-[#5E4642] px-5 py-2.5 font-semibold text-white">
        ▶ צפייה בדמו החי
      </a>
    </section>
  );
}
```

---

## 2. Highlights list

```tsx
import data from "@/content/wig-crm/project-data.json";

export function WigCrmHighlights() {
  return (
    <ul className="grid gap-3 sm:grid-cols-2">
      {data.highlights.map((h) => (
        <li key={h} className="flex gap-3 rounded-xl bg-white p-4 ring-1 ring-[#EDE1DB]">
          <span className="mt-1 h-2 w-2 shrink-0 rounded-full bg-[#C88E7F]" />
          <span className="text-[#4A3833]">{h}</span>
        </li>
      ))}
    </ul>
  );
}
```

---

## 3. "At a glance" fact strip

This project's story is about *constraints*, not raw scale — so the strip highlights the
zero-infrastructure facts rather than vanity metrics.

```tsx
const facts = [
  { k: "קובץ אחד",      v: "כל המערכת" },
  { k: "0 תלויות",     v: "בלי build, בלי שרת" },
  { k: "אופליין מלא",  v: "עובד מ‑file://" },
  { k: "עברית RTL",    v: "8 מסכים" },
];

export function WigCrmFacts() {
  return (
    <div dir="rtl" className="grid grid-cols-2 gap-px overflow-hidden rounded-xl bg-[#EDE1DB] sm:grid-cols-4">
      {facts.map((f) => (
        <div key={f.k} className="bg-[#FAF5F2] p-5 text-center">
          <b className="block text-lg font-bold text-[#48342F]">{f.k}</b>
          <small className="text-[#A18A83]">{f.v}</small>
        </div>
      ))}
    </div>
  );
}
```

---

## 4. Live demo embed

```tsx
export function WigCrmDemoFrame({ src }: { src: string }) {
  return (
    <div className="overflow-hidden rounded-2xl ring-1 ring-[#EDE1DB] shadow-lg">
      <iframe
        src={src}                       /* the demo Artifact URL */
        title="Wig CRM — interactive demo"
        loading="lazy"
        className="h-[720px] w-full border-0"
      />
    </div>
  );
}
```

> Note: the demo is published as a private Claude Artifact and opens when you are signed in
> to the owning account. For a fully public portfolio, self-host `demo/index.html` instead
> and point the iframe at that URL.
