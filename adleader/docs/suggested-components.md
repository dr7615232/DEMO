# Suggested Components — feature AdLeader on a portfolio site

Ready‑to‑adapt React/Next components for showing this project. They use the AdLeader palette and are
framework‑light (Tailwind classes are illustrative; swap for your own system). All copy is
placeholder‑safe.

---

## Design tokens

```css
:root {
  --al-navy: #07306b;
  --al-secondary: #15508f;
  --al-cyan: #00b7c7;
  --al-cyan-soft: #e6fbfd;
  --al-ink: #071827;
  --al-muted: #52687a;
}
/* fonts: Assistant (body), Rubik (headings), Heebo (numbers) */
```

---

## 1. `ProjectHero`

```tsx
export function ProjectHero() {
  return (
    <section className="relative overflow-hidden rounded-2xl bg-[#07306b] p-10 text-white">
      <div className="max-w-2xl">
        <p className="text-sm font-semibold tracking-wide text-[#7fe3ec]">Case study · 2026</p>
        <h1 className="mt-3 text-4xl font-extrabold leading-tight">
          AdLeader — every inquiry, tied to the ad that brought it
        </h1>
        <p className="mt-4 text-lg text-white/80">
          A production, Hebrew‑RTL, multi‑tenant SaaS that captures leads from every channel,
          attributes each one to its source and campaign, and turns ad spend into a real
          per‑source profit‑and‑loss.
        </p>
        <div className="mt-6 flex flex-wrap gap-3">
          <a href="{DEMO_URL}" className="rounded-lg bg-[#00b7c7] px-5 py-2.5 font-semibold text-[#07306b]">
            ▶ Live demo
          </a>
          <a href="{PROJECT_PAGE_URL}" className="rounded-lg border border-white/30 px-5 py-2.5 font-semibold">
            Project page (Hebrew)
          </a>
        </div>
      </div>
      <div aria-hidden className="pointer-events-none absolute -left-16 -bottom-16 h-64 w-64 rounded-full bg-[#00b7c7]/20 blur-2xl" />
    </section>
  );
}
```

## 2. `StackChips`

```tsx
const STACK = ["Next.js 14", "TypeScript", "Firebase / Firestore", "Tailwind RTL",
  "next-intl", "Zod", "TanStack Query", "Recharts", "Vitest", "Playwright"];

export function StackChips() {
  return (
    <ul className="flex flex-wrap gap-2">
      {STACK.map((s) => (
        <li key={s} className="rounded-full border border-[#d5e8ee] bg-[#e6fbfd] px-3 py-1 text-sm font-medium text-[#0a5a62]">
          {s}
        </li>
      ))}
    </ul>
  );
}
```

## 3. `HighlightList` — engineering highlights

```tsx
const HIGHLIGHTS = [
  "Automatic attribution: every inquiry bound to its source at capture time",
  "Race-safe lead dedup in a single Firestore transaction (leadKeys index)",
  "Per-source & per-campaign advertising profit-and-loss",
  "AES-256-GCM at rest for all third-party credentials",
  "Firestore-backed rate limiting, idempotency and deferred jobs (no Redis)",
  "Tenant isolation proven at rules AND handler layers",
];

export function HighlightList() {
  return (
    <ul className="space-y-3">
      {HIGHLIGHTS.map((h) => (
        <li key={h} className="flex gap-3">
          <span className="mt-2 h-2.5 w-2.5 flex-none rounded-full bg-[#bcd42e]" />
          <span className="text-[#334155]">{h}</span>
        </li>
      ))}
    </ul>
  );
}
```

## 4. `DemoEmbed` — inline the live demo

```tsx
export function DemoEmbed({ url = "{DEMO_URL}" }: { url?: string }) {
  return (
    <figure className="overflow-hidden rounded-2xl border border-[#d5e8ee] shadow-lg">
      <iframe
        src={url}
        title="AdLeader interactive demo"
        loading="lazy"
        className="h-[720px] w-full"
      />
      <figcaption className="bg-[#07306b] px-4 py-2 text-center text-xs text-white/70">
        תצוגת דמו · נתונים לדוגמה בלבד
      </figcaption>
    </figure>
  );
}
```

> Note: the demo is published as a private Claude Artifact; if embedding fails due to frame policy,
> link out to it instead of iframing.

## 5. `MetricTile` (optional, for an internal deck — not for the client‑facing page)

```tsx
export function MetricTile({ label, value, sub }: { label: string; value: string; sub?: string }) {
  return (
    <div className="rounded-xl border border-[#d5e8ee] bg-white p-5">
      <div className="text-sm text-[#52687a]">{label}</div>
      <div className="mt-1 text-3xl font-extrabold text-[#07306b] [font-family:Heebo]">{value}</div>
      {sub && <div className="mt-1 text-xs text-[#718698]">{sub}</div>}
    </div>
  );
}
```

## 6. Next.js `generateMetadata`

```tsx
// app/projects/adleader/page.tsx
export function generateMetadata() {
  return {
    title: "AdLeader — Lead Attribution & Advertising P&L",
    description:
      "A production Hebrew-RTL SaaS that captures every inquiry from every channel and ties it to the ad that produced it. Next.js 14 + Firebase, multi-tenant.",
    openGraph: {
      title: "AdLeader — Lead Attribution & Advertising P&L",
      description: "Every inquiry, tied to the ad that brought it.",
      images: ["/og/adleader.png"],
      locale: "he_IL",
    },
    alternates: { canonical: "/projects/adleader" },
  };
}
```

---

### Composition example

```tsx
export default function AdLeaderProjectPage() {
  return (
    <main className="mx-auto max-w-4xl space-y-10 p-6">
      <ProjectHero />
      <section><h2 className="mb-3 text-xl font-bold">Stack</h2><StackChips /></section>
      <section><h2 className="mb-3 text-xl font-bold">Engineering highlights</h2><HighlightList /></section>
      <section><h2 className="mb-3 text-xl font-bold">See it live</h2><DemoEmbed /></section>
    </main>
  );
}
```
