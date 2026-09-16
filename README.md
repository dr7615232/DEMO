# פרויקט דמו · Portfolio

A portfolio repository of project case studies.

## Projects

### 🌊 [Zramim (זרמים)](./zramim/) — studio-management platform with a self-service phone line

A production, Hebrew-RTL management system for a swimming & water-aerobics studio:
subscriptions, class scheduling, a reconciling financial ledger, and a full self-service
IVR phone line (registration, card clearing, automated Hebrew-TTS debt collection).

**Stack:** Next.js 15 · React 19 · Supabase (PostgreSQL + RLS) · TypeScript · Tailwind ·
Yemot HaMashiach (IVR) · Nedarim Plus · Resend/SMTP

Start with **[`zramim/README.md`](./zramim/README.md)** for the full index, or jump to the
**[case study](./zramim/case-study.md)**.

**🌊 Interactive UI demo:** [`demo-app/index.html`](./demo-app/index.html) — a faithful,
self-contained reconstruction of the real system's screens in Hebrew (RTL) with demo data:
dashboard, lessons, customers, customer card, groups, products, debts, payments, calls,
reports, broadcast, finances, settings, phone-line settings, and logs. Open it in a browser
(no build step). Uses example data only — no real customers.

> ℹ️ These are portfolio materials describing a real client system. Before making anything
> public, review the anonymization checklist in [`zramim/README.md`](./zramim/README.md).
