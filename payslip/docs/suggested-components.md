# Suggested Portfolio Components — Payslip Check

Drop-in React/Next components to present this project on a portfolio site. They are self-contained,
dependency-light (Tailwind classes, no UI library required), and read from `project-data.json`.
Colours below use the project page's cream/olive/lime palette so the card matches the live page.

---

## 1. Project hero card

```tsx
export function PayslipHero() {
  return (
    <section dir="rtl" className="rounded-2xl border p-8"
      style={{ background: "#fbfbf3", borderColor: "#c3c79f", color: "#363a1f" }}>
      <div className="text-xs font-semibold tracking-widest" style={{ color: "#a2a877" }}>
        בדיקת תלוש שכר · PAYSLIP CHECK
      </div>
      <h2 className="mt-3 text-3xl font-extrabold">לדעת שהתלוש שלך תקין</h2>
      <p className="mt-2 max-w-xl" style={{ color: "#5f6440" }}>
        מערכת שקוראת תלוש שכר, בודקת אותו מול דיני העבודה בישראל, ומחזירה תשובה ברורה עם הסבר
        ואומדן כספי לכל ממצא.
      </p>
      <div className="mt-5 flex gap-3">
        <a href="https://claude.ai/artifact/JiYZ78FcF8Kb3GWsyr6YN1" target="_blank" rel="noopener"
           className="rounded-full px-5 py-2 font-bold"
           style={{ background: "#bcd42e", color: "#2c3410" }}>▶ דמו חי</a>
        <a href="https://claude.ai/artifact/EhcaNPusrQKdZjJ5EkGEjD" target="_blank" rel="noopener"
           className="rounded-full px-5 py-2 font-bold border" style={{ borderColor: "#c3c79f" }}>
          עמוד הפרויקט
        </a>
      </div>
    </section>
  );
}
```

---

## 2. Tech-stack chips (data-driven)

```tsx
import data from "./project-data.json";

export function StackChips() {
  const stack = Object.values(data.stack).flat() as string[];
  return (
    <ul className="flex flex-wrap gap-2">
      {stack.map((t) => (
        <li key={t} className="rounded-full border px-3 py-1 text-sm"
            style={{ borderColor: "#c3c79f", color: "#8f9f22", background: "#f3f4e7" }}>
          {t}
        </li>
      ))}
    </ul>
  );
}
```

---

## 3. Highlights list

```tsx
import data from "./project-data.json";

export function Highlights() {
  return (
    <ul className="space-y-2">
      {data.highlights.map((h: string, i: number) => (
        <li key={i} className="flex gap-2">
          <span aria-hidden style={{ color: "#8f9f22" }}>◆</span>
          <span style={{ color: "#5f6440" }}>{h}</span>
        </li>
      ))}
    </ul>
  );
}
```

---

## 4. "How a finding looks" mini demo

A static illustration of the product's core output — useful on the case-study page without embedding
the whole app.

```tsx
export function FindingPreview() {
  return (
    <div dir="rtl" className="rounded-2xl border bg-white overflow-hidden"
         style={{ borderColor: "#e5e7eb", borderRight: "4px solid #fecaca" }}>
      <div className="px-4 py-3">
        <div className="flex items-center gap-2 flex-wrap">
          <span className="h-2.5 w-2.5 rounded-full" style={{ background: "#ef4444" }} />
          <span className="font-semibold">תעריף שעה נוספת (125%) נמוך מהנדרש</span>
          <span className="rounded-full px-2 py-0.5 text-xs" style={{ background: "#fee2e2", color: "#991b1b" }}>ליקוי</span>
          <span className="rounded-full px-2 py-0.5 text-xs" style={{ background: "#f3f4f6", color: "#4b5563" }}>ודאות גבוהה</span>
        </div>
        <div className="mt-3 grid grid-cols-3 gap-3 text-sm">
          <div><div className="text-xs text-gray-500">בפועל</div><div>₪39.00</div></div>
          <div><div className="text-xs text-gray-500">צפוי</div><div>₪42.75</div></div>
          <div><div className="text-xs text-gray-500">פער</div><div className="font-semibold">₪3.75</div></div>
        </div>
        <div className="mt-3 rounded-xl border p-3 text-sm" style={{ background: "#f9fafb", borderColor: "#e5e7eb" }}>
          <span className="font-semibold" style={{ color: "#1548e1" }}>פער משוער: ₪45.00</span>
        </div>
      </div>
    </div>
  );
}
```

---

## 5. Live-demo callout

```tsx
export function DemoCallout() {
  return (
    <a href="https://claude.ai/artifact/JiYZ78FcF8Kb3GWsyr6YN1" target="_blank" rel="noopener"
       dir="rtl" className="flex items-center justify-center gap-3 rounded-2xl border p-4 font-bold"
       style={{ background: "#fbfbf3", borderColor: "#c3c79f", color: "#363a1f" }}>
      <span aria-hidden style={{ color: "#8f9f22" }}>▶</span>
      צפייה בדמו החי (נתונים לדוגמה בלבד)
    </a>
  );
}
```

---

### Integration notes

- Copy `project-data.json` next to these components (or fetch it) so the chips/highlights stay in sync.
- Both artifact links are private and open when signed in to the owner's Claude account; for a public
  site, self-host the demo/site HTML from `payslip/demo/` and `payslip/site/` instead.
- Keep the product's disclaimer ("preliminary check, not legal advice") anywhere you show real output.
