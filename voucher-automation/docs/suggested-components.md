# Suggested Components — שובר אוטומטי

Ready-to-drop React/Next components for presenting this project on a portfolio site. They
are self-contained (inline styles / Tailwind-friendly), RTL-aware, and use only example
data. Replace the demo URLs with your published Artifact links.

---

## 1. `ProjectHero` — the masthead

```tsx
// components/voucher/ProjectHero.tsx
export function ProjectHero({ demoUrl }: { demoUrl: string }) {
  return (
    <header dir="rtl" className="mx-auto max-w-3xl px-6 py-16">
      <div className="mb-6 flex items-baseline gap-2">
        <b className="text-lg font-extrabold tracking-wide">שובר אוטומטי</b>
        <span className="text-xs font-semibold uppercase tracking-[0.16em] text-[#a2a877]">
          תרומה שהופכת לשובר, לבד
        </span>
      </div>
      <h1 className="text-4xl font-extrabold leading-tight text-[#363a1f] md:text-5xl">
        התורם תורם, השובר יוצא אליו לבד
      </h1>
      <p className="mt-4 max-w-[60ch] text-lg font-medium text-[#5f6440]">
        במקום להכין כל שובר ביד, מערכת שסוגרת את כל המעגל: התורם בוחר לאן, והשובר יוצא
        אליו במייל תוך שניות.
      </p>
      <a
        href={demoUrl}
        target="_blank"
        rel="noopener"
        className="mt-8 flex w-full items-center justify-center gap-3 rounded-2xl border border-[#c3c79f] bg-[#fbfbf3] px-6 py-4 font-bold text-[#363a1f] transition hover:-translate-y-0.5 hover:border-[#8f9f22]"
      >
        ▶ צפייה בדמו החי של המערכת
      </a>
    </header>
  );
}
```

---

## 2. `NumberedSection` — a numbered case-study block

```tsx
// components/voucher/NumberedSection.tsx
export function NumberedSection({
  n,
  title,
  children,
  full = false,
}: {
  n: string;
  title: string;
  children: React.ReactNode;
  full?: boolean;
}) {
  return (
    <div
      dir="rtl"
      className={`border-t border-[#d8dbbe] py-8 ${full ? "col-span-full" : ""}`}
    >
      <div className="mb-2 flex items-baseline gap-2">
        <h3 className="text-lg font-extrabold text-[#363a1f]">{title}</h3>
        <span className="text-sm font-semibold tabular-nums text-[#a2a877]">{n}</span>
      </div>
      <div className="max-w-[46ch] text-[#5f6440]">{children}</div>
    </div>
  );
}
```

Usage:

```tsx
<section dir="rtl" className="mx-auto grid max-w-3xl grid-cols-1 gap-x-16 px-6 md:grid-cols-2">
  <NumberedSection n="01" title="הבעיה">כל תרומה דרשה עבודה ידנית…</NumberedSection>
  <NumberedSection n="02" title="לפני">אישור התרומה נחת במייל…</NumberedSection>
  <NumberedSection n="03" title="הפתרון">מערכת אחת שמקבלת את הפרטים לבד…</NumberedSection>
  <NumberedSection n="04" title="האוטומציה">השובר נוצר ונשלח לבד…</NumberedSection>
</section>
```

---

## 3. `FeatureList` — the "what it does now, by itself" bullets

```tsx
// components/voucher/FeatureList.tsx
const ACTIONS = [
  "קולטת את פרטי התרומה לבד, בלי העתקה מהמייל",
  "שולחת לתורם טופס נעים לבחירת היעד",
  "מפיקה שובר מעוצב עם מספר ייחודי משלה",
  "שולחת את השובר לתורם במייל, מיד",
  "מעדכנת את מלאי המקום שנבחר, אוטומטית",
  "מתריעה כשמלאי של מקום מתקרב לסיום",
];

export function FeatureList() {
  return (
    <ul dir="rtl" className="grid grid-cols-1 gap-2 md:grid-cols-2">
      {ACTIONS.map((a) => (
        <li key={a} className="relative ps-6 font-medium text-[#363a1f]">
          <span className="absolute inset-inline-start-0 top-2 h-2 w-2 rounded-full bg-[#bcd42e] ring-4 ring-[#bcd42e]/20" />
          {a}
        </li>
      ))}
    </ul>
  );
}
```

---

## 4. `DemoCallout` — a reusable "watch the demo" bar

```tsx
// components/voucher/DemoCallout.tsx
export function DemoCallout({ demoUrl }: { demoUrl: string }) {
  return (
    <div dir="rtl" className="mx-auto flex max-w-3xl flex-wrap items-center justify-between gap-3 border-t-2 border-[#363a1f] px-6 py-8">
      <p className="text-lg font-bold text-[#363a1f]">הכי קל להבין כשרואים את זה עובד.</p>
      <a
        href={demoUrl}
        target="_blank"
        rel="noopener"
        className="rounded-full border border-[#bcd42e] bg-[#bcd42e] px-6 py-3 font-extrabold text-[#2c3410] transition hover:-translate-y-0.5"
      >
        צפייה בדמו החי ←
      </a>
    </div>
  );
}
```

---

## 5. `VoucherCard` — a static voucher preview (for a gallery)

```tsx
// components/voucher/VoucherCard.tsx
export function VoucherCard({
  place = "סופר השכונה",
  donor = "יעל אזולאי",
  value = "₪ 500",
  number = "100483",
}: {
  place?: string; donor?: string; value?: string; number?: string;
}) {
  return (
    <div dir="rtl" className="mx-auto max-w-md overflow-hidden rounded-2xl border border-[#e2ece8] bg-white shadow-xl">
      <div className="bg-gradient-to-br from-[#c8892b] to-[#a9741f] p-6 text-white">
        <small className="text-xs font-bold uppercase tracking-widest opacity-80">שובר מתנה</small>
        <h2 className="mt-1 text-2xl font-black">{place}</h2>
      </div>
      <div className="p-6">
        <Row label="לכבוד" value={donor} />
        <Row label="ערך השובר" value={value} />
        <div className="mt-4 rounded-xl border border-dashed border-[#c8892b] bg-[#faf6ee] p-4 text-center">
          <small className="block text-xs tracking-widest text-[#5f7a72]">מספר שובר ייחודי</small>
          <b className="mt-1 block text-2xl font-black tracking-widest text-[#c8892b] tabular-nums">{number}</b>
        </div>
      </div>
    </div>
  );
}
const Row = ({ label, value }: { label: string; value: string }) => (
  <div className="flex justify-between border-b border-dashed border-[#e2ece8] py-2 text-sm last:border-0">
    <span className="text-[#5f7a72]">{label}</span>
    <b className="font-bold text-[#12312b]">{value}</b>
  </div>
);
```

---

## Palette tokens (for a shared theme file)

```ts
// theme.ts — project page (cream / olive / lime)
export const pageTheme = {
  ground: "#f3f4e7", surface: "#fbfbf3",
  ink: "#363a1f", inkSoft: "#5f6440", num: "#a2a877",
  line: "#d8dbbe", lineStrong: "#c3c79f",
  lime: "#bcd42e", limeDeep: "#8f9f22",
};

// theme.ts — app/demo (teal / gold)
export const appTheme = {
  brand: "#0d6e5f", brand2: "#128975", soft: "#3bb39a",
  gold: "#c8892b", bg: "#f4f7f5", line: "#e2ece8",
  ink: "#12312b", muted: "#5f7a72",
};
```
