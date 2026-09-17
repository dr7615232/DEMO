# פרויקט דמו · Portfolio

A portfolio repository of project case studies.

## Projects

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
