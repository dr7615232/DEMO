# SEO & Social Metadata — AdLeader

Drop‑in metadata for a portfolio page featuring this project. Replace `https://your-portfolio.example`
and the OG image path with your real values before publishing.

---

## Titles & descriptions

**Primary title (EN):**
> AdLeader — Multi‑Channel Lead Attribution & Advertising P&L (Next.js + Firebase)

**Primary title (HE):**
> אדלידר — מדידת פרסום ושיוך פניות לעסקים

**Meta description (EN, ~155 chars):**
> A production Hebrew‑RTL SaaS that captures every inquiry from every channel and ties it to the ad
> that produced it, turning ad spend into real per‑source profit. Next.js 14 + Firebase.

**Meta description (HE):**
> מערכת SaaS בעברית שאוספת כל פנייה מכל ערוץ ומחברת אותה לפרסום שהביא אותה, כדי לראות איזה פרסום באמת
> מחזיר. נבנתה ב‑Next.js 14 ו‑Firebase.

**Short tagline:** Every inquiry, tied to the ad that brought it. · מפרסום לליד, בלי לאבד אף פנייה.

---

## Open Graph

```html
<meta property="og:type" content="website" />
<meta property="og:site_name" content="Portfolio" />
<meta property="og:title" content="AdLeader — Lead Attribution & Advertising P&L" />
<meta property="og:description" content="A production Hebrew-RTL SaaS that captures every inquiry from every channel and ties it to the ad that produced it. Next.js 14 + Firebase, multi-tenant." />
<meta property="og:url" content="https://your-portfolio.example/projects/adleader" />
<meta property="og:image" content="https://your-portfolio.example/og/adleader.png" />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
<meta property="og:locale" content="he_IL" />
<meta property="og:locale:alternate" content="en_US" />
```

## Twitter / X

```html
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content="AdLeader — Lead Attribution & Advertising P&L" />
<meta name="twitter:description" content="Capture every inquiry from every channel, attribute it to the ad that produced it, and see real return per shekel. Next.js 14 + Firebase." />
<meta name="twitter:image" content="https://your-portfolio.example/og/adleader.png" />
```

## Canonical & language alternates

```html
<link rel="canonical" href="https://your-portfolio.example/projects/adleader" />
<link rel="alternate" hreflang="he" href="https://your-portfolio.example/he/projects/adleader" />
<link rel="alternate" hreflang="en" href="https://your-portfolio.example/en/projects/adleader" />
```

---

## JSON‑LD (SoftwareApplication + CreativeWork)

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "SoftwareApplication",
  "name": "AdLeader",
  "alternateName": "אדלידר",
  "applicationCategory": "BusinessApplication",
  "operatingSystem": "Web",
  "inLanguage": "he",
  "description": "Multi-tenant, Hebrew-RTL SaaS for multi-channel lead attribution and advertising profit-and-loss: captures every inquiry from every channel and ties it to the source and campaign that produced it.",
  "author": { "@type": "Person", "name": "Your Name" },
  "keywords": "lead attribution, multi-channel marketing, advertising ROI, ROAS, CRM, Next.js, Firebase, Hebrew RTL, multi-tenant SaaS",
  "offers": { "@type": "Offer", "category": "Portfolio case study" }
}
</script>
```

Optional wrapper describing the case study itself:

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "CreativeWork",
  "headline": "AdLeader — Lead Attribution & Advertising P&L (case study)",
  "about": ["Lead attribution", "Marketing analytics", "Multi-tenant architecture", "Next.js", "Firebase"],
  "inLanguage": ["he", "en"],
  "creator": { "@type": "Person", "name": "Your Name" }
}
</script>
```

---

## Keywords

**Primary:** lead attribution, advertising ROI, ROAS per channel, multi‑channel marketing, campaign
measurement, lead management CRM, marketing analytics SaaS.

**Technical:** Next.js 14 App Router, Firebase Firestore, multi‑tenant SaaS, Hebrew RTL, next‑intl,
AES‑256‑GCM, serverless rate limiting, webhook idempotency, tenant isolation, Zod, Playwright,
Vitest.

**Hebrew:** מדידת פרסום, שיוך פניות, החזר על פרסום, ניהול לידים, קמפיינים, קישורים מדידים, וואטסאפ
לעסקים, מערכת CRM בעברית.

---

## Suggested OG image content

A 1200×630 card in the AdLeader palette (navy `#07306b` → cyan `#00b7c7`): the AdLeader logo, the
Hebrew tagline "מפרסום לליד, בלי לאבד אף פנייה", and one line of English — "Lead attribution &
advertising P&L · Next.js + Firebase". A screenshot of the demo dashboard also works well.
