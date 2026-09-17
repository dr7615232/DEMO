# Demo Instructions — Payslip Check

Two ways to demo this project: the **self-contained HTML demo** in this package (instant, safe,
example data only), or the **original application** running locally.

---

## Option A — the self-contained demo (recommended for interviews & links)

No build, no server, no data.

- **File:** `payslip/demo/index.html` (in this package).
- **Live artifact:** https://claude.ai/artifact/JiYZ78FcF8Kb3GWsyr6YN1
- **Project page:** https://claude.ai/artifact/EhcaNPusrQKdZjJ5EkGEjD

Open the HTML file in any browser, or share the artifact link (private — opens when signed in to the
owner's Claude account). It faithfully reconstructs the real screens with example data.

**Suggested 90-second walkthrough**

1. **Home** — dual audience: private employee vs. payroll professional.
2. **New check** — toggle to "בעל/ת מקצוע" to reveal the per-client selector.
3. **Wizard → Upload** — payslip is required; attendance/contract optional; no upfront questionnaire.
4. **Wizard → Verify** — every extracted field and pay/deduction line is editable (the human is in
   the loop before anything is judged).
5. **Run the check**, then **Follow-up questions** — note these appear only when data is missing.
6. **Results** — the verdict, the total estimated gap, and expandable findings. Open the red
   "overtime 125%" finding to show *actual / expected / gap*, the monetary block, the confidence
   badge, and the legal source.
7. **Report** — the printable, per-category view (try the browser's print/PDF).
8. **Compare** — cross-month trends and total estimated gap.
9. **Admin → usage** — per-user, per-model cost accounting.

---

## Option B — run the original application locally

> Do this from a **separate clone** of the original repository. This portfolio package must not be
> added to, or run from inside, that repo.

**Prerequisites:** Node.js 18+ and npm.

```bash
# in a separate working copy of the original repo
npm install
npm run dev            # http://localhost:3000
```

For a production-style run:

```bash
npm run build && npm run start
```

**Tests & type-checking** (good to show engineering rigour):

```bash
npm test               # Vitest — engine rules, extraction, comparison, law params
npm run typecheck
```

**Zero-config defaults:** with no environment variables the app runs in open dev mode (a `dev` user
with the admin role) and uses the **file-based store** — perfect for a local demo. No database, no
keys required.

**What to show in the real app**

- Upload a **sample payslip you own** (never someone else's), or type values on the verify screen.
- Run the engine and open the detailed report.
- If you want to demo AI extraction/OCR, set a provider key in `/admin/ai` (see
  `integration-instructions.md`) — otherwise the heuristic parser + manual entry handles it.

**Optional environment (see the repo's `.env.example`)**

- `DATABASE_URL` — switch to the PostgreSQL backend (checks as JSONB, encrypted document BYTEA).
- `PAYSLIP_ENC_KEY` — document encryption key (required in production).
- `NEXT_PUBLIC_SUPABASE_URL` / `NEXT_PUBLIC_SUPABASE_ANON_KEY` — enable real auth.
- `OCR_PROVIDER` + a provider key (`GEMINI_API_KEY` recommended) — enable OCR / AI extraction.

---

## Demo hygiene

- Use **example data only**. Never upload a real person's payslip, name, ID, or salary.
- Keep the product disclaimer visible: this is a preliminary check, not legal advice.
- Legal values shown are illustrative and some are marked "to verify" in the code.
