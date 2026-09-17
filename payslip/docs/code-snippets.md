# Code Snippets — Payslip Check

Curated, real excerpts (lightly trimmed) with commentary. Safe to publish: no secrets, no real
personal data. Paths are relative to `src/`.

---

## 1. The `Finding` — a self-explaining result

Every rule speaks the same rich vocabulary. A finding is not just "pass/fail": it carries the
evidence, the law, the confidence, and the money.

```ts
// lib/domain/types.ts
export interface MonetaryImpact {
  actual?: number;    // paid in practice
  computed?: number;  // what was expected
  gap?: number;       // computed - actual, positive = owed to the employee
  certainty: "high" | "estimate";
}

export interface Finding {
  ruleId: string;
  title: string;                          // what was found
  severity: "ok" | "info" | "warning" | "error";
  confidence: "high" | "medium" | "low";
  message: string;
  actualText?: string;                    // display: in practice
  expectedText?: string;                  // display: expected
  gapText?: string;                       // display: the gap
  legalSource: string;                    // law / order / agreement
  explanation: string;
  monetary?: MonetaryImpact;
  dependsOnQuestions?: string[];
}
```

---

## 2. A rule end-to-end — overtime premium (125%)

Reads the actual overtime rate off the payslip, compares to 125% of the base hourly rate, and if it
is short, emits an `error` with a **high-certainty** monetary gap. (The same rule also handles 150%,
attendance-derived expectations, and global overtime as an *estimate*.)

```ts
// lib/engine/rules/overtime.ts (trimmed)
if (ctx.payslip.hourlyRate && ot125Comp?.rate) {
  const expectedRate = round2(ctx.payslip.hourlyRate * OVERTIME.firstHoursRate); // ×1.25
  if (ot125Comp.rate + 0.1 < expectedRate) {
    findings.push({
      ruleId: "overtime",
      title: "תעריף שעה נוספת (125%) נמוך מהנדרש",
      severity: "error",
      confidence: "high",
      message: `תעריף השעה הנוספת (${ils(ot125Comp.rate)}) נמוך מ־125% מהתעריף הרגיל.`,
      actualText: ils(ot125Comp.rate),
      expectedText: `${ils(expectedRate)} (125% מ־${ils(ctx.payslip.hourlyRate)})`,
      gapText: ils(round2(expectedRate - ot125Comp.rate)),
      legalSource: OVERTIME.source, // "חוק שעות עבודה ומנוחה, התשי״א-1951"
      explanation: "שעתיים נוספות ראשונות מזכות ב־125% לפחות מהתעריף הרגיל.",
      monetary: {
        actual: ot125Comp.rate * (ot125Comp.quantity ?? 1),
        computed: expectedRate * (ot125Comp.quantity ?? 1),
        gap: round2((expectedRate - ot125Comp.rate) * (ot125Comp.quantity ?? 1)),
        certainty: "high",
      },
    });
  }
}
```

---

## 3. The storage interface — one contract, two backends

Application code never imports a concrete store. It depends on this interface; the implementation is
chosen at runtime.

```ts
// lib/store/repository.ts
export interface CheckRepository {
  create(check: Check): Promise<Check>;
  get(id: string): Promise<Check | null>;
  update(id: string, patch: Partial<Check>): Promise<Check | null>;
  delete(id: string): Promise<boolean>;
  list(ownerId: string, opts?: { query?: string; limit?: number }): Promise<Check[]>;
  saveDocument(checkId: string, docId: string, data: Buffer, originalName: string): Promise<string>;
  readDocument(storedPath: string): Promise<Buffer | null>;
  deleteDocument(storedPath: string): Promise<boolean>;
  purgeDocuments(checkId: string): Promise<void>;
}
```

```ts
// lib/store/index.ts — runtime selection, memoized
export function getRepository(): CheckRepository {
  if (!repo) {
    const url = process.env.DATABASE_URL;
    repo = url ? new PostgresCheckRepository(url) : new FileCheckRepository();
  }
  return repo;
}
```

---

## 4. Summary aggregation — separating certainty from estimate

The engine keeps two running totals: the full estimated gap, and the subset it is highly confident
about. Both surface in the UI so the number is never oversold.

```ts
// lib/engine/runEngine.ts (trimmed)
let totalEstimatedGap = 0, highCertaintyGap = 0;
for (const f of findings) {
  if (f.monetary?.gap && f.monetary.gap > 0) {
    totalEstimatedGap += f.monetary.gap;
    if (f.monetary.certainty === "high") highCertaintyGap += f.monetary.gap;
  }
}
const summary: CheckResultSummary = {
  status: flaggedCount > 0 ? "needs_review" : "ok",
  findingsCount: flaggedCount,
  errorsCount, warningsCount,
  totalEstimatedGap: totalEstimatedGap > 0 ? round2(totalEstimatedGap) : undefined,
  highCertaintyGap: highCertaintyGap > 0 ? round2(highCertaintyGap) : undefined,
};
```

---

## 5. Cross-month trends for comparison

Comparison is pure and testable: normalise each check to a row, sort by period, diff first vs. last.

```ts
// lib/domain/compare.ts
export function sortRowsByPeriod(rows: CompareRow[]): CompareRow[] {
  return [...rows].sort((a, b) => (a.period ?? "").localeCompare(b.period ?? ""));
}

export function trend(rows: CompareRow[], field: keyof CompareRow): number | null {
  const sorted = sortRowsByPeriod(rows).filter(r => typeof r[field] === "number");
  if (sorted.length < 2) return null;
  return (sorted[sorted.length - 1][field] as number) - (sorted[0][field] as number);
}
```

---

## 6. The finding card — confidence and money in the UI

The React card mirrors the finding one-to-one: a severity dot and label, a confidence chip, and, when
there is a real gap, a "monetary meaning" block that labels the number as high-certainty or estimate.

```tsx
// components/FindingCard.tsx (trimmed)
{finding.monetary && (finding.monetary.gap ?? 0) > 0 && (
  <div className="rounded-xl bg-gray-50 border border-gray-200 p-3 text-sm">
    <div className="font-medium mb-1">משמעות כספית</div>
    <div className="flex flex-wrap gap-x-6 gap-y-1">
      {finding.monetary.actual !== undefined && <span>בפועל: {ils(finding.monetary.actual)}</span>}
      {finding.monetary.computed !== undefined && <span>מחושב: {ils(finding.monetary.computed)}</span>}
      <span className="font-semibold text-brand-700">פער משוער: {ils(finding.monetary.gap)}</span>
      <span className="chip bg-gray-200 text-gray-700">
        {finding.monetary.certainty === "high" ? "ודאות גבוהה" : "אומדן"}
      </span>
    </div>
  </div>
)}
```

---

## 7. Currency & period formatting (Hebrew locale)

Small, shared formatters keep every screen consistent and correctly localized.

```ts
// lib/format.ts
export function ils(n: number | null | undefined): string {
  if (n == null || !Number.isFinite(n)) return "—";
  return new Intl.NumberFormat("he-IL", { style: "currency", currency: "ILS", maximumFractionDigits: 2 }).format(n);
}

export function formatPeriod(period: string | null): string {
  if (!period) return "—";
  const [y, m] = period.split("-");
  const months = ["ינואר","פברואר","מרץ","אפריל","מאי","יוני","יולי","אוגוסט","ספטמבר","אוקטובר","נובמבר","דצמבר"];
  return months[Number(m) - 1] ? `${months[Number(m) - 1]} ${y}` : period;
}
```
