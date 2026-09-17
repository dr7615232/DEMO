# Architecture — Payslip Check

This document describes the system as built. All claims map to the codebase
(`src/app`, `src/lib/engine`, `src/lib/law`, `src/lib/extraction`, `src/lib/store`,
`src/lib/security`, `src/lib/auth`).

---

## 1. System context

```mermaid
flowchart TB
  subgraph Client["Browser (Hebrew, RTL)"]
    U["Employee / Professional / Admin"]
  end

  subgraph App["Next.js 14 App Router"]
    Pages["Pages: home, new check, wizard,\nreport, history, compare,\nclients, settings, admin"]
    API["Route handlers (/api/*)"]
    MW["middleware.ts\n(owner cookie / identity)"]
  end

  subgraph Core["Domain core (src/lib)"]
    EX["Extraction pipeline"]
    EN["Rules engine (17 rules)"]
    LAW["Legal parameters\n(date-versioned)"]
    SEC["Security\n(crypto / privacy / auth)"]
    STORE["Repository interface"]
  end

  subgraph Ext["External services (optional / pluggable)"]
    SB["Supabase Auth"]
    AI["AI providers\n(Gemini / OpenAI / Anthropic)"]
    PG[("PostgreSQL")]
    FS[("File store + AES-GCM")]
  end

  U --> Pages --> API
  API --> MW
  API --> EX --> EN
  EN --> LAW
  API --> STORE
  STORE --> PG
  STORE --> FS
  EX --> AI
  API --> SEC
  Pages --> SB
```

The browser talks only to Next.js pages and route handlers. The domain core is pure TypeScript and
has no framework coupling: the engine, the extraction pipeline, and the legal parameters can be unit
tested in isolation (and are, under Vitest).

---

## 2. The check lifecycle (state machine)

A `Check` moves through four states; the wizard mirrors them.

```mermaid
stateDiagram-v2
  [*] --> draft: create (mode + label)
  draft --> extracted: upload payslip → auto-extract
  extracted --> verified: user confirms / corrects fields
  verified --> completed: run engine
  completed --> completed: answer follow-ups → re-run
  completed --> [*]: report / compare / delete
```

- **draft** — created, no documents yet.
- **extracted** — a document was uploaded and parsed; awaiting human verification.
- **verified** — the user confirmed or corrected the extracted data.
- **completed** — the engine ran; findings, follow-up questions, and a summary exist. Answering a
  follow-up re-runs the engine on the same check.

---

## 3. Data model (core domain types)

```mermaid
classDiagram
  class Check {
    id
    ownerId
    mode  // employee | professional
    clientId?
    label
    status  // draft|extracted|verified|completed
    hasContract
    answers
  }
  class PayslipData {
    period
    employeeName
    employerName
    seniorityMonths
    grossPay
    netPay
    hourlyRate
    hoursWorked
    daysWorked
    balances
    cumulative
  }
  class PayComponent { label; amount; quantity?; rate?; kind }
  class Deduction { label; amount; kind }
  class EmployerContribution { label; amount; kind; rate? }
  class AttendanceData { period; totalHours; totalDays; days }
  class StoredDocument { id; kind; originalName; storedPath; size }
  class Finding {
    ruleId; title; severity; confidence
    message; actualText; expectedText; gapText
    legalSource; explanation; monetary
  }
  class FollowUpQuestion { id; text; type; choices?; askedBy }
  class CheckResultSummary {
    status; findingsCount; errorsCount; warningsCount
    totalEstimatedGap; highCertaintyGap
  }

  Check "1" --> "1" PayslipData
  Check "1" --> "0..1" AttendanceData
  Check "1" --> "*" StoredDocument
  Check "1" --> "*" Finding
  Check "1" --> "*" FollowUpQuestion
  Check "1" --> "1" CheckResultSummary
  PayslipData "1" --> "*" PayComponent
  PayslipData "1" --> "*" Deduction
  PayslipData "1" --> "*" EmployerContribution
```

The `Check` is the aggregate root; the repository persists it whole (as a file, or as a JSONB row in
PostgreSQL). Raw documents are stored separately and referenced by `storedPath`.

---

## 4. Extraction pipeline

```mermaid
flowchart LR
  F["Uploaded file"] --> T{"MIME type?"}
  T -->|PDF with text| P["pdf-parse → text"]
  T -->|scanned PDF / image| O["OcrProvider.recognize()"]
  O --> OM{"provider mode"}
  OM -->|off| MAN["fall back to manual entry"]
  OM -->|tesseract| TES["tesseract.js (local)"]
  OM -->|ai / auto| AIP["AI provider (Gemini/OpenAI/Anthropic)"]
  P --> HP["Hebrew heuristic parser"]
  TES --> HP
  AIP --> STR["Structured JSON extraction\n(typed PayslipData) + cost log"]
  HP --> V["Verify screen (editable)"]
  STR --> V
```

Text-layer PDFs are parsed directly; scanned documents go through an injected `OcrProvider`. When an
AI provider is configured, extraction can return typed JSON directly (more accurate than parsing
text) and logs the model and cost per use. **Every path lands on the editable verify screen** — the
human is always in the loop before the engine judges anything.

---

## 5. Rules engine

```mermaid
flowchart TB
  RC["RuleContext\n(payslip, attendance, answers,\nperiodDate, resolved params, sector)"]
  RC --> LOOP["runEngine(): for each Rule in registry"]
  LOOP --> APP{"rule.isApplicable(ctx)?"}
  APP -->|no| SKIP["skip"]
  APP -->|yes| RUN["rule.run(ctx) → findings + questions"]
  RUN --> AGG["aggregate"]
  AGG --> SORT["sort by severity, then confidence"]
  SORT --> SUM["summary: counts + totalEstimatedGap\n+ highCertaintyGap"]
  SUM --> OUT["EngineResult"]
```

Rules are registered in `engine/registry.ts`; the runner is agnostic to them. Each rule pulls the
legal values it needs from the date-resolved parameter set (managed values override code constants),
emits findings with confidence and monetary impact, and may emit follow-up questions that are
de-duplicated across rules.

**The 17 rules:** minimum wage · split-rate · teaching-sector degree · teaching-sector seniority ·
overtime · pension · convalescence · vacation · sick pay · severance · holiday pay · contract bonus
· study fund · income-tax sanity · travel · attendance cross-check · gross/net/deduction
consistency.

---

## 6. Storage abstraction

```mermaid
flowchart LR
  CODE["Application code"] --> IFACE["CheckRepository (interface)"]
  IFACE --> SEL{"DATABASE_URL set?"}
  SEL -->|yes| PGR["PostgresCheckRepository\n(checks: JSONB, docs: encrypted BYTEA)"]
  SEL -->|no| FR["FileCheckRepository\n(JSON files + encrypted document blobs)"]
```

One interface, two implementations, chosen at runtime by the presence of `DATABASE_URL`. The rest of
the system depends only on the interface. A parallel `adminStore` (file + PostgreSQL) holds profiles,
the audit log, and usage records.

---

## 7. Security & identity

- **Encryption at rest** — documents (and stored AI keys) are encrypted with AES-256-GCM
  (`security/crypto.ts`); the key comes from `PAYSLIP_ENC_KEY` in production.
- **Owner isolation** — every `Check` carries an `ownerId`; access is denied to non-owners.
- **ID masking** — national IDs are masked to their last four digits in list views
  (`security/privacy.ts`).
- **Auth & roles** — Supabase Auth (Google / email-password); roles `admin` / `professional` /
  `employee`, with an admin "view-as" read-only mode and an audit trail.
- **Retention** — a retention window underpins proactive/automatic document deletion.
