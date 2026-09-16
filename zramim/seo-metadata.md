# Zramim — SEO & Social Metadata

Copy-paste metadata for a portfolio case-study page. Replace `https://your-domain.example`
and image paths with your real values before publishing.

---

## Page title options

- `Zramim — Studio-Management Platform & Self-Service Phone Line | <Your Name>`
- `Zramim: Full-Stack SaaS with IVR Telephony (Next.js + Supabase) | <Your Name>`

## Meta description (≤160 chars)

> Full-stack case study: a Hebrew-RTL studio-management platform with atomic transactional
> billing, a reconciling deferred-revenue ledger, and a self-service IVR phone line.

Alt (shorter):

> Next.js 15 + Supabase studio-management SaaS with a self-service Hebrew IVR phone system,
> transactional subscription billing, and deferred-revenue accounting.

## Keywords

`Next.js`, `React`, `Supabase`, `PostgreSQL`, `Row-Level Security`, `TypeScript`,
`Server Actions`, `IVR`, `telephony`, `Yemot HaMashiach`, `Nedarim Plus`, `SaaS`,
`vertical SaaS`, `subscription management`, `deferred revenue`, `ledger accounting`,
`Hebrew`, `RTL`, `full-stack engineer`, `case study`

---

## Open Graph / Twitter tags

```html
<!-- Primary -->
<title>Zramim — Studio-Management Platform & Self-Service Phone Line</title>
<meta name="description" content="Full-stack case study: a Hebrew-RTL studio-management platform with atomic transactional billing, a reconciling deferred-revenue ledger, and a self-service IVR phone line." />

<!-- Open Graph -->
<meta property="og:type" content="article" />
<meta property="og:title" content="Zramim — Studio-Management Platform & Self-Service Phone Line" />
<meta property="og:description" content="Next.js 15 + Supabase SaaS with a self-service Hebrew IVR phone system, transactional subscription billing, and deferred-revenue accounting." />
<meta property="og:url" content="https://your-domain.example/work/zramim" />
<meta property="og:image" content="https://your-domain.example/images/zramim-og.png" />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />

<!-- Twitter -->
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="Zramim — Studio-Management Platform & Self-Service Phone Line" />
<meta name="twitter:description" content="Full-stack SaaS with IVR telephony, transactional billing, and deferred-revenue accounting. Next.js + Supabase." />
<meta name="twitter:image" content="https://your-domain.example/images/zramim-og.png" />
```

---

## Next.js App Router `metadata` export

```ts
// app/work/zramim/page.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Zramim — Studio-Management Platform & Self-Service Phone Line",
  description:
    "Full-stack case study: a Hebrew-RTL studio-management platform with atomic transactional billing, a reconciling deferred-revenue ledger, and a self-service IVR phone line.",
  keywords: [
    "Next.js", "Supabase", "PostgreSQL", "Row-Level Security", "IVR", "telephony",
    "SaaS", "subscription management", "deferred revenue", "TypeScript", "full-stack",
  ],
  openGraph: {
    type: "article",
    title: "Zramim — Studio-Management Platform & Self-Service Phone Line",
    description:
      "Next.js 15 + Supabase SaaS with a self-service Hebrew IVR phone system, transactional subscription billing, and deferred-revenue accounting.",
    url: "https://your-domain.example/work/zramim",
    images: [{ url: "https://your-domain.example/images/zramim-og.png", width: 1200, height: 630 }],
  },
  twitter: {
    card: "summary_large_image",
    title: "Zramim — Studio-Management Platform & Self-Service Phone Line",
    description:
      "Full-stack SaaS with IVR telephony, transactional billing, and deferred-revenue accounting.",
    images: ["https://your-domain.example/images/zramim-og.png"],
  },
};
```

---

## JSON-LD structured data

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "CreativeWork",
  "name": "Zramim",
  "alternateName": "זרמים",
  "headline": "Studio-management platform with a self-service Hebrew phone line",
  "about": "Full-stack vertical-SaaS platform for a swimming & water-aerobics studio: subscriptions, class scheduling, deferred-revenue ledger, and a self-service IVR phone system.",
  "author": { "@type": "Person", "name": "<Your Name>" },
  "dateCreated": "2026",
  "inLanguage": "he",
  "keywords": "Next.js, Supabase, PostgreSQL, IVR, telephony, SaaS, subscription management, deferred revenue, TypeScript, full-stack",
  "url": "https://your-domain.example/work/zramim"
}
</script>
```

---

## OG image suggestion

If you generate a 1200×630 social card: put the Zramim logo (`assets/logo.png`) on a deep
water-blue (`#1B6E9B`) → light-blue (`#3FA9DD`) gradient, with the title in white and a
one-line subtitle. Keep the pink accent (`#E05A79`) for one small highlight. No screenshots
with real customer data.
