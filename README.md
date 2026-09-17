# פרויקט דמו · Portfolio

A portfolio repository of project case studies.

## Projects

### 🦜 [תוכי פרינט — בונה הקטלוג](./tuki/) — a self-updating product catalog builder for a print shop

A Hebrew-RTL catalog builder for a print business with hundreds of products. Instead of
rebuilding a catalog from scratch each season, the owner ticks which products go in this
month; the system assembles a designed catalog with a festive cover, correct per-unit and
bulk prices, and a ready-to-fill client order form — ready to send in a few clicks.

**Stack:** Self-contained HTML · CSS · JavaScript (single-file demo, no build)

**▶ צפייה חיה / Live:**
- 📄 עמוד הפרויקט (Project page): https://claude.ai/artifact/Q1NbhQTXLwFGPNzXxwSY2c
- 🦜 דמו אינטראקטיבי (Interactive demo): https://claude.ai/artifact/N6HuWafMG1kLRiFnvZsX6p

*(הקישורים פרטיים — נפתחים כשאת מחוברת לחשבון Claude שלך.)*

Start with **[`tuki/docs/README.md`](./tuki/docs/README.md)** for the full index, or jump to
the **[case study](./tuki/docs/case-study.md)**.

**📄 Project page (Hebrew):** [`tuki/site/index.html`](./tuki/site/index.html) — a designed,
RTL Hebrew case-study page (cream/olive/lime style) telling the project story, with a link to the live demo. Open it in a browser (no build).

**🦜 Interactive UI demo:** [`tuki/demo/index.html`](./tuki/demo/index.html) — a self-contained
reconstruction of the catalog builder, the client cover page and the order form, in Hebrew (RTL)
with example data. Open it in a browser (no build step).

---

### 🌞 [SUNSUITE](./sunsuite/) — booking & operations platform for luxury vacation-rental and spa complexes

A production, Hebrew-RTL platform where guests book short-stay suites and a private
spa (overnight or by the hour) online, and the owner runs the whole operation from
one admin panel. Availability accounts for the cleaning buffer between guests; one
click approves a booking and the system auto-schedules the cleaning, syncs the stay
to Google Calendar, and emails the guest. A dedicated **Shabbat mode** closes the
site automatically using real candle-lighting/havdalah times.

**Stack:** TanStack Start (React 19) · TanStack Router/Query · Supabase (PostgreSQL + RLS) ·
TypeScript · Tailwind v4 · Cloudflare · Google Calendar · Cloudinary · Resend/SMTP · Hebcal

**▶ צפייה חיה / Live:**
- 📄 עמוד הפרויקט (Project page): https://claude.ai/artifact/U5ice3pCdtNUEsU4LsTT63
- 🌞 דמו אינטראקטיבי (Interactive demo): https://claude.ai/artifact/CvWsa7cPV7KZwcPJvUrez1

*(הקישורים פרטיים — נפתחים כשאת מחוברת לחשבון Claude שלך.)*

Start with **[`sunsuite/docs/README.md`](./sunsuite/docs/README.md)** for the full index, or jump to
the **[case study](./sunsuite/docs/case-study.md)**.

**📄 Project page (Hebrew):** [`sunsuite/site/index.html`](./sunsuite/site/index.html) — a designed,
RTL Hebrew case-study page telling the project story, with a link to the live demo. Open it in a browser (no build).

**🌞 Interactive UI demo:** [`sunsuite/demo/index.html`](./sunsuite/demo/index.html) — a faithful,
self-contained reconstruction of the real system's screens in Hebrew (RTL) with demo data:
admin overview, bookings, calendar (with Hebrew dates), cleanings, blocks, properties, customers,
reports, settings, and the public booking site with an availability calendar and Shabbat mode.
Open it in a browser (no build step). Uses example data only — no real customers.

> ℹ️ These are portfolio materials describing a real client system. Before making anything
> public, review the anonymization checklist in [`sunsuite/docs/README.md`](./sunsuite/docs/README.md).

---

### 🌊 [Zramim (זרמים)](./zramim/) — studio-management platform with a self-service phone line

A production, Hebrew-RTL management system for a swimming & water-aerobics studio:
subscriptions, class scheduling, a reconciling financial ledger, and a full self-service
IVR phone line (registration, card clearing, automated Hebrew-TTS debt collection).

**Stack:** Next.js 15 · React 19 · Supabase (PostgreSQL + RLS) · TypeScript · Tailwind ·
Yemot HaMashiach (IVR) · Nedarim Plus · Resend/SMTP

**▶ צפייה חיה / Live:**
- 📄 עמוד הפרויקט (Project page): https://claude.ai/artifact/P5vpD3q4C5aaeQJw3VJ8KK
- 🌊 דמו אינטראקטיבי (Interactive demo): https://claude.ai/artifact/18W19Dv89LWLWV4A2rrtuM

*(הקישורים פרטיים — נפתחים כשאת מחוברת לחשבון Claude שלך.)*

Start with **[`zramim/docs/README.md`](./zramim/docs/README.md)** for the full index, or jump to
the **[case study](./zramim/docs/case-study.md)**.

**📄 Project page (Hebrew):** [`zramim/site/index.html`](./zramim/site/index.html) — a designed,
RTL Hebrew case-study page telling the project story: the problem, the features, and the phone
system, with a link to the live demo. Open it in a browser (no build).

**🌊 Interactive UI demo:** [`zramim/demo/index.html`](./zramim/demo/index.html) — a faithful,
self-contained reconstruction of the real system's screens in Hebrew (RTL) with demo data:
dashboard, lessons, customers, customer card, groups, products, debts, payments, calls,
reports, broadcast, finances, settings, phone-line settings, and logs. Open it in a browser
(no build step). Uses example data only — no real customers.

> ℹ️ These are portfolio materials describing a real client system. Before making anything
> public, review the anonymization checklist in [`zramim/docs/README.md`](./zramim/docs/README.md).

---

### 🌸 [Wig CRM (אומנות בפאות)](./wig-crm/) — a whole wig-studio business in one offline HTML file

A production, Hebrew-RTL business-management system for a wig studio, delivered as a **single,
self-contained HTML file** that runs offline by double-click — no install, no server, no
internet, zero dependencies. One interface runs the whole business: clients, a two-stage
appointment calendar, tasks, a sales pipeline, and a finance suite. Its heart is the money
model: a product's cost and supplier hang off the price list, so every sale computes true
profit, auto-posts a charge to a self-reconciling supplier ledger, and never shows the cost on
the screen the customer can see.

**Stack:** Vanilla JavaScript (no framework, no build) · HTML5 · CSS3 · localStorage ·
File System Access API · IndexedDB · fully offline

**▶ צפייה חיה / Live:**
- 📄 עמוד הפרויקט (Project page): https://claude.ai/artifact/3BpMvdsk3X7rgx3A9m1yAu
- 🌸 דמו אינטראקטיבי (Interactive demo): https://claude.ai/artifact/3dWkLcgWaeR2ThY3n5XnEq

*(הקישורים פרטיים — נפתחים כשאת מחוברת לחשבון Claude שלך.)*

Start with **[`wig-crm/docs/README.md`](./wig-crm/docs/README.md)** for the full index, or jump to
the **[case study](./wig-crm/docs/case-study.md)**.

**📄 Project page (Hebrew):** [`wig-crm/site/index.html`](./wig-crm/site/index.html) — a designed,
RTL Hebrew case-study page telling the project story: the problem, the features, and the money
that adds up by itself, with a link to the live demo. Open it in a browser (no build).

**🌸 Interactive UI demo:** [`wig-crm/demo/index.html`](./wig-crm/demo/index.html) — a faithful,
self-contained reconstruction of the real system's screens in Hebrew (RTL) with demo data:
dashboard, calendar, clients, tasks, deals, finances (overview, transactions, upcoming, suppliers),
and settings. Open it in a browser (no build step). Uses example data only — no real customers.

> ℹ️ These are portfolio materials describing a real client system. The client's real logo
> contains a personal name and is deliberately excluded; the pages use a neutral wordmark.
> Before making anything public, review the anonymization checklist in
> [`wig-crm/docs/README.md`](./wig-crm/docs/README.md).

---

### 🎟️ [Voucher Automation (שובר אוטומטי)](./voucher-automation/) — donations that become vouchers, automatically

An automation that closes the whole loop between a received donation and a delivered
thank-you voucher, with zero manual work. Donation details flow in on their own; the donor
gets a short form to choose **where** to redeem; the system generates a designed voucher with
a **unique, never-reused number** and emails it in seconds. Stock per redemption location is
tracked continuously, and the operator is alerted **before** any location runs low. One
dashboard shows the whole operation.

**Stack:** Reference implementation — PostgreSQL (transactions, unique sequences) · Node.js ·
server-side HTML→PDF · transactional email · a lightweight job runner for retries

**▶ צפייה חיה / Live:**
- 📄 עמוד הפרויקט (Project page): https://claude.ai/artifact/MTozqShyt4TCYcDwX5fdjZ
- 🎟️ דמו אינטראקטיבי (Interactive demo): https://claude.ai/artifact/E8qpZ9VRKBggksd5sUFntd

*(הקישורים פרטיים — נפתחים כשאת מחוברת לחשבון Claude שלך.)*

Start with **[`voucher-automation/docs/README.md`](./voucher-automation/docs/README.md)** for the full index, or jump to
the **[case study](./voucher-automation/docs/case-study.md)**.

**📄 Project page (Hebrew):** [`voucher-automation/site/index.html`](./voucher-automation/site/index.html) — a designed,
RTL Hebrew case-study page telling the project story: the manual process before, the automation,
the feature that watches stock, and the result, with a link to the live demo. Open it in a browser (no build).

**🎟️ Interactive UI demo:** [`voucher-automation/demo/index.html`](./voucher-automation/demo/index.html) — a faithful,
self-contained reconstruction of the system's screens in Hebrew (RTL) with demo data:
dashboard, donations, issued vouchers, voucher stock, low-stock alerts, the donor choice form,
the delivered voucher, and settings. Open it in a browser (no build step). Uses example data only — no real donors.

> ℹ️ This project was reconstructed from the operator's description of the workflow, not from a
> shared source repository; the architecture and code are a representative reference implementation.
> All donors, emails, partner locations, amounts, and voucher numbers are examples. Before making
> anything public, review the anonymization checklist in
> [`voucher-automation/docs/README.md`](./voucher-automation/docs/README.md).

---

### 🧾 [Payslip Check (בדיקת תלוש שכר)](./payslip/) — automated Israeli payslip auditing in plain Hebrew

A Hebrew-RTL web application that tells a person whether their Israeli payslip is correct. The user
uploads a payslip (PDF or image), optionally an attendance report and a contract; the system extracts
the data, shows it for verification, and runs a **rules engine grounded in Israeli labour law and
extension orders**. It returns a clear verdict, valid or needs review, with per-finding legal source,
confidence level, plain-language explanation, and an estimated monetary gap (split into high-certainty
vs. estimate). One codebase serves two audiences: a private employee, and a payroll professional with
per-client setting cards.

**Stack:** Next.js 14 (App Router, React 18) · TypeScript · Tailwind · PostgreSQL (with a file-based
fallback) · Supabase Auth · pdf-parse / tesseract.js / multi-provider AI extraction (Gemini · OpenAI ·
Anthropic) · Zod · Vitest · GitHub Actions

**▶ צפייה חיה / Live:**
- 📄 עמוד הפרויקט (Project page): https://claude.ai/artifact/EhcaNPusrQKdZjJ5EkGEjD
- 🧾 דמו אינטראקטיבי (Interactive demo): https://claude.ai/artifact/JiYZ78FcF8Kb3GWsyr6YN1

*(הקישורים פרטיים — נפתחים כשאת מחוברת לחשבון Claude שלך.)*

Start with **[`payslip/docs/README.md`](./payslip/docs/README.md)** for the full index, or jump to
the **[case study](./payslip/docs/case-study.md)**.

**📄 Project page (Hebrew):** [`payslip/site/index.html`](./payslip/site/index.html) — a designed,
RTL Hebrew case-study page (cream/olive/lime style) telling the project story: the problem, the
automatic check, and the result, with a link to the live demo. Open it in a browser (no build).

**🧾 Interactive UI demo:** [`payslip/demo/index.html`](./payslip/demo/index.html) — a faithful,
self-contained reconstruction of the real system's screens in Hebrew (RTL) with demo data: home,
new check, the four-step wizard (upload, verify, follow-up questions, results), the detailed report,
history, comparison, client cards, settings, and the admin usage & cost screen. Open it in a browser
(no build step). Uses example data only — no real payslips or people.

> ℹ️ These are portfolio materials describing a working application. All names, employers, amounts,
> and legal parameter values are examples (several legal values are marked "to verify" in the code),
> and the tool provides a preliminary check, not legal advice. Before making anything public, review
> the anonymization checklist in [`payslip/docs/README.md`](./payslip/docs/README.md).
