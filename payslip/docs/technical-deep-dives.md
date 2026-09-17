# Technical Deep-Dives — Payslip Check

Four engineering topics that show how the system is put together. All code excerpts are real,
lightly trimmed for length. Paths are relative to `src/`.

---

## 1. An extensible rules engine that grows by addition

The domain — Israeli labour law — is large and keeps growing. The engine is designed so that adding a
new check never means editing the core: it is one new file plus one registry line. Every rule
implements the same interface and receives the same context.

```ts
// lib/engine/types.ts
export interface RuleContext {
  payslip: PayslipData;
  attendance: AttendanceData | null;
  hasContract: boolean;
  answers: Record<string, string | number | boolean>;
  periodDate: Date;            // the payslip month, for date-versioned params
  params: LegalParamSet | null; // admin-managed values, override code constants
  sector: string | null;       // e.g. "teaching" activates sector rules
}

export interface Rule {
  id: string;
  title: string;
  category: "minimum_wage" | "pension" | "benefits" | "overtime"
    | "attendance" | "deductions" | "consistency" | "sector" | "anomaly";
  isApplicable?(ctx: RuleContext): boolean;
  run(ctx: RuleContext): RuleOutput; // { findings, questions? }
}
```

The runner is deliberately dumb: it iterates the registry, skips rules that don't apply, collects
findings and de-duplicated questions, guards each rule in a `try/catch` so one bad rule can't crash
the check, then sorts by severity and confidence and computes the summary.

```ts
// lib/engine/runEngine.ts (core loop, trimmed)
for (const rule of RULES) {
  try {
    if (rule.isApplicable && !rule.isApplicable(ctx)) continue;
    const out = rule.run(ctx);
    findings.push(...out.findings);
    for (const q of out.questions ?? []) {
      if (!questionMap.has(q.id) && !(q.id in ctx.answers)) questionMap.set(q.id, q);
    }
  } catch (err) {
    findings.push({ ruleId: rule.id, title: `שגיאה בבדיקה: ${rule.title}`,
      severity: "info", confidence: "low",
      message: "אירעה שגיאה בהרצת הבדיקה. נא לבדוק את הנתונים שהוזנו.",
      legalSource: "-", explanation: String(err instanceof Error ? err.message : err) });
  }
}

findings.sort((a, b) =>
  severityOrder[a.severity] - severityOrder[b.severity] ||
  confOrder[a.confidence] - confOrder[b.confidence]);
```

**Why it matters:** the interesting behaviour lives in small, independently testable units. The
`__tests__` folders exercise rules in isolation; the registry is the single source of truth for what
runs. Growth is linear and safe.

---

## 2. Honest uncertainty: "cannot determine" instead of a false positive

A compliance tool that guesses is worse than none. The minimum-wage rule shows the pattern used
across the engine: it looks for the actual hourly rate by descending order of certainty, and if it
cannot establish one, it **asks a question and returns an informational finding** rather than
declaring a violation.

```ts
// lib/engine/rules/minimumWage.ts (trimmed)
let actualHourly: number | null = null, basis = "", confidence: Finding["confidence"] = "high";

if (ctx.payslip.hourlyRate && ctx.payslip.hourlyRate > 0) {
  actualHourly = ctx.payslip.hourlyRate; basis = "תעריף שעתי מהתלוש"; confidence = "high";
} else if (answeredRate && answeredRate > 0) {
  actualHourly = answeredRate; basis = "תעריף שעתי שהוזן על ידי המשתמש"; confidence = "high";
} else if (ctx.payslip.baseSalary && ctx.payslip.hoursWorked) {
  actualHourly = round2(ctx.payslip.baseSalary / ctx.payslip.hoursWorked);
  basis = "שכר בסיס חלקי שעות עבודה"; confidence = "medium";
}

if (actualHourly === null) {
  questions.push({ id: "minwage.hourlyRate",
    text: "מהו התעריף השעתי שלך (₪ לשעה)? אם השכר חודשי, ניתן להשאיר ריק.",
    type: "number", askedBy: "minimum_wage" });
  findings.push({ ruleId: "minimum_wage", title: "שכר מינימום — לא ניתן לקבוע",
    severity: "info", confidence: "low",
    message: "לא זוהה תעריף שעתי ולא ניתן לחשב שכר שעתי בפועל.",
    expectedText: `שכר מינימום לשעה: ${ils(minHourly)}`, legalSource: source,
    explanation: "כדי לבדוק עמידה בשכר מינימום נדרש התעריף השעתי בפועל...",
    dependsOnQuestions: ["minwage.hourlyRate"] });
  return { findings, questions };
}
```

Note that `confidence` is derived from *how* the value was obtained (a value read straight off the
payslip is `high`; one inferred from base salary ÷ hours is `medium`). That confidence rides along on
the finding and becomes a visible badge in the UI. When a real gap is found from an inferred value,
its monetary impact is tagged `estimate`, not `high` — the tool never overstates what it knows.

---

## 3. Date-versioned legal parameters, with managed overrides

Labour-law values change most years, and a payslip must be judged by the values valid **for its
month**, not today's. Every parameter is stored as a dated, source-tagged, confidence-tagged record,
and resolved against the payslip date.

```ts
// lib/law/constants.ts (trimmed)
export interface Dated<T> { effectiveFrom: string; confidence: Confidence; source: string; value: T; }

export function resolveDated<T>(list: Dated<T>[], date: Date): Dated<T> | null {
  const ts = date.getTime();
  return list
    .filter(d => new Date(d.effectiveFrom).getTime() <= ts)
    .sort((a, b) => new Date(b.effectiveFrom).getTime() - new Date(a.effectiveFrom).getTime())[0] ?? null;
}

export const MINIMUM_WAGE: Dated<MinimumWage>[] = [
  { effectiveFrom: "2023-04-01", confidence: "high",
    source: "חוק שכר מינימום, התשמ״ז-1987", value: { monthly: 5571.75, standardMonthlyHours: 182 } },
  { effectiveFrom: "2024-04-01", confidence: "high",
    source: "חוק שכר מינימום — עדכון אפריל 2024", value: { monthly: 5880.02, standardMonthlyHours: 182 } },
  { effectiveFrom: "2025-01-01", confidence: "medium",  // marked "to verify"
    source: "עדכון שכר מינימום 2025 (לאימות)", value: { monthly: 6247.67, standardMonthlyHours: 182 } },
];
```

Rules prefer **admin-managed** values (editable in `/admin/legal`, persisted per store) and fall back
to these code constants:

```ts
// lib/engine/rules/minimumWage.ts (source selection)
if (ctx.params?.minimumWageMonthly && ctx.params?.standardMonthlyHours) {
  minMonthly = ctx.params.minimumWageMonthly; minHours = ctx.params.standardMonthlyHours;
} else {
  const mw = resolveDated(MINIMUM_WAGE, ctx.periodDate);
  if (mw) { minMonthly = mw.value.monthly; minHours = mw.value.standardMonthlyHours; source = mw.source; }
}
```

**Why it matters:** yearly legal updates become a data edit in the admin UI, not a code change and
redeploy — and the `confidence` on each record flows through to the finding, so a value the developer
hasn't verified for the current year is surfaced as an estimate rather than presented as fact.

---

## 4. Structured AI extraction with graceful degradation and cost logging

Parsing Hebrew payslips from raw OCR text is brittle — layouts vary wildly. When an AI provider is
configured, the pipeline extracts the payslip **directly into typed JSON**, which is far more
accurate. Crucially, extraction is an *injected capability*: with no provider, the app degrades to the
heuristic parser and manual entry rather than failing.

```ts
// lib/extraction/index.ts (trimmed) — one pipeline, provider injected
async function documentToText(buffer, mimeType, opts) {
  const ocr = opts.ocr ?? null;
  if (mimeType === "application/pdf") {
    const text = await extractPdfText(buffer);
    if (text.trim().length > 0) return { text, ocrUsed: false, warnings: [] };
    const r = ocr ? await ocr.recognize(buffer, mimeType) : { text: "", ocrUsed: false };
    if (r.ocrUsed && r.text.trim()) return { text: r.text, ocrUsed: true,
      warnings: ["ה-PDF סרוק — הטקסט חולץ אוטומטית באמצעות OCR. מומלץ לבדוק ולתקן את הנתונים."] };
    return { text: r.text, ocrUsed: false, warnings: [failureWarning("pdf", !!opts.isAdmin, r.error)] };
  }
  // ...image / text/plain branches
}
```

The failure message itself is role-aware — an admin gets a technical diagnosis pointing at
`/admin/ai`, an ordinary user gets a friendly "enter the data manually" note:

```ts
function failureWarning(kind: "pdf" | "image", isAdmin: boolean, error?: string): string {
  const what = kind === "pdf" ? "ה-PDF הסרוק" : "התמונה";
  const base = `לא ניתן היה לקרוא את ${what} אוטומטית. נא להזין את הנתונים ידנית.`;
  return isAdmin
    ? `${base} (אבחון: OCR נכשל${error ? " — " + error : ""}. בדקו הגדרות ב-/admin/ai.)`
    : `${base} אם הבעיה חוזרת, ניתן לפנות לתמיכה.`;
}
```

AI keys are stored **encrypted** (same AES-256-GCM primitive as documents), providers are selectable
per mode (`off` / `tesseract` / `ai` / `auto`), and each AI call records the model used and its cost
— surfaced per user and per model in the admin usage-and-cost screen. This makes an otherwise
invisible operating expense measurable and attributable.

---

## Bonus: privacy as a build requirement

Payslips are sensitive personal data, so the security primitives are small, explicit, and reused. The
document cipher is a textbook AES-256-GCM envelope (`iv | tag | ciphertext`), and the same functions
encrypt stored AI keys:

```ts
// lib/security/crypto.ts (trimmed)
export function encryptBuffer(plain: Buffer): Buffer {
  const iv = crypto.randomBytes(12);
  const cipher = crypto.createCipheriv("aes-256-gcm", key(), iv);
  const enc = Buffer.concat([cipher.update(plain), cipher.final()]);
  return Buffer.concat([iv, cipher.getAuthTag(), enc]); // 12 + 16 + n
}
```

National IDs are masked to their last four digits everywhere they appear in lists
(`maskId` → `••••4821`), each check is owner-isolated, and a retention window supports deletion.
Security here is a set of composable functions the whole app leans on, not a bolt-on.
