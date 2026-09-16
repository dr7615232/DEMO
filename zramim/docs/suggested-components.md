# Zramim — Suggested Portfolio-Site Components

Ready-to-adapt React/Next components for **presenting** this project on your portfolio site.
These are for *your* portfolio, not the original app. They use plain Tailwind and no
dependencies, and echo the Zramim brand palette so the case-study page feels on-theme.

**Brand tokens (from the app's `tailwind.config.ts`):**

```
brand         #1B6E9B  (deep water blue — headings, primary)
brand.soft    #3FA9DD  (light blue — hover, tags, focus)
brand.accent  #E05A79  (petal pink — highlights, active state)
```

---

## 1. Hero header

```tsx
export function ProjectHero() {
  return (
    <header className="relative overflow-hidden rounded-3xl bg-gradient-to-br from-[#1B6E9B] to-[#3FA9DD] px-8 py-14 text-white">
      <p className="mb-2 text-sm font-semibold uppercase tracking-widest text-white/70">
        Full-stack · Vertical SaaS · Telephony
      </p>
      <h1 className="text-4xl font-extrabold sm:text-5xl">Zramim</h1>
      <p className="mt-4 max-w-2xl text-lg text-white/90">
        A production, Hebrew-RTL management platform for a swimming &amp; water-aerobics
        studio — subscriptions, a reconciling financial ledger, and a full self-service
        phone line.
      </p>
      <div className="mt-6 flex flex-wrap gap-2">
        {["Next.js 15", "Supabase", "PostgreSQL + RLS", "TypeScript", "IVR / Yemot"].map((t) => (
          <span key={t} className="rounded-full bg-white/15 px-3 py-1 text-sm font-medium">
            {t}
          </span>
        ))}
      </div>
      <span aria-hidden className="pointer-events-none absolute -right-8 -top-8 h-40 w-40 rounded-full bg-[#E05A79]/30 blur-2xl" />
    </header>
  );
}
```

## 2. Metrics strip

```tsx
const METRICS = [
  { value: "26.5k", label: "lines of TypeScript" },
  { value: "71",    label: "SQL migrations" },
  { value: "~24",   label: "database tables" },
  { value: "~48",   label: "RLS policies" },
  { value: "9",     label: "scheduled jobs" },
];

export function MetricsStrip() {
  return (
    <dl className="grid grid-cols-2 gap-4 sm:grid-cols-5">
      {METRICS.map((m) => (
        <div key={m.label} className="rounded-2xl border border-[#e3edf2] bg-white p-5 text-center">
          <dt className="text-3xl font-extrabold text-[#1B6E9B]">{m.value}</dt>
          <dd className="mt-1 text-xs text-gray-500">{m.label}</dd>
        </div>
      ))}
    </dl>
  );
}
```

## 3. Feature card grid

```tsx
const FEATURES = [
  { icon: "💳", title: "Atomic billing", body: "Subscription + N class bookings + ledger charge in one Postgres transaction." },
  { icon: "📊", title: "Reconciling ledger", body: "Customer-level cash ledger plus a deferred-revenue waterfall that sums to the bank." },
  { icon: "📞", title: "Self-service IVR", body: "Phone registration, card clearing, class swaps, and debt calls — no staff needed." },
  { icon: "🔒", title: "Defense in depth", body: "Role invariants enforced at both the app layer and Row-Level Security." },
  { icon: "🗣️", title: "Hebrew TTS", body: "Nikud-aware text-to-speech with validated automatic vowelization." },
  { icon: "⏱️", title: "Idempotent automation", body: "An hourly dispatcher runs timed, deduped debt and renewal jobs." },
];

export function FeatureGrid() {
  return (
    <div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
      {FEATURES.map((f) => (
        <article key={f.title} className="rounded-2xl border border-[#e3edf2] bg-white p-6 transition hover:-translate-y-1 hover:border-[#3FA9DD]">
          <span className="text-2xl" aria-hidden>{f.icon}</span>
          <h3 className="mt-3 font-bold text-[#1B6E9B]">{f.title}</h3>
          <p className="mt-1 text-sm text-gray-600">{f.body}</p>
        </article>
      ))}
    </div>
  );
}
```

## 4. Tech-stack list (grouped)

```tsx
const STACK = {
  "Frontend":   ["Next.js 15 (App Router)", "React 19", "Server Actions", "Tailwind CSS"],
  "Data":       ["Supabase / PostgreSQL", "Row-Level Security", "PL/pgSQL RPCs"],
  "Telephony":  ["Yemot HaMashiach IVR", "Dicta Nakdan (nikud TTS)"],
  "Integrations": ["Nedarim Plus (payments)", "Resend / SMTP (email)"],
  "Ops":        ["Vercel / Render", "GitHub Actions (cron)"],
};

export function StackList() {
  return (
    <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-3">
      {Object.entries(STACK).map(([group, items]) => (
        <div key={group}>
          <h4 className="mb-2 text-xs font-bold uppercase tracking-wider text-[#E05A79]">{group}</h4>
          <ul className="space-y-1 text-sm text-gray-700">
            {items.map((i) => <li key={i} className="flex gap-2"><span className="text-[#3FA9DD]">▸</span>{i}</li>)}
          </ul>
        </div>
      ))}
    </div>
  );
}
```

## 5. Deep-dive accordion (details/summary — no JS)

```tsx
const DIVES = [
  { q: "How does registration stay consistent?", a: "One transactional Postgres function writes the subscription, books N class occurrences, and posts the ledger charge together — capacity enforced by a trigger, so no half-written state is possible." },
  { q: "How does the money reconcile?", a: "Payments are tracked at the customer level (not per subscription), and a waterfall allocates cash into earned / deferred / frozen / credit buckets that always sum to cash received." },
  { q: "How does a stateless phone line have back-navigation?", a: "Yemot re-sends every keypress on each request; the app keeps a per-call navigation stack keyed by call-ID so * and 0 pop the menu correctly, while the handler stays a pure function." },
];

export function DeepDives() {
  return (
    <div className="space-y-3">
      {DIVES.map((d) => (
        <details key={d.q} className="group rounded-2xl border border-[#e3edf2] bg-white p-5">
          <summary className="cursor-pointer list-none font-semibold text-[#1B6E9B] marker:content-none">
            {d.q}
          </summary>
          <p className="mt-3 text-sm leading-relaxed text-gray-600">{d.a}</p>
        </details>
      ))}
    </div>
  );
}
```

---

### Data-driven option

Every list above can be sourced from `project-data.json` instead of being hard-coded — import
it and map over `highlights`, `features`, and `stack`. That keeps the page and the structured
metadata in sync.
