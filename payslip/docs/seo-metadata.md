# SEO & Social Metadata — Payslip Check

Ready-to-use metadata for a portfolio page presenting this project. Replace `https://example.com/...`
with your real URLs. Keep the "not legal advice" framing out of ad copy but present on the product.

---

## Titles & descriptions

**Primary (English)**
- **Title:** Payslip Check — Automated Israeli Payslip Auditing (Case Study)
- **Meta description:** A Hebrew-RTL web app that reads a payslip and audits it against Israeli
  labour law: a 17-rule engine returning legal sources, confidence levels, and estimated monetary
  gaps. Full-stack case study (Next.js + TypeScript).

**Primary (Hebrew)**
- **Title:** בדיקת תלוש שכר — מערכת אוטומטית לבדיקת תלושים (סקירת פרויקט)
- **Meta description:** מערכת עברית שקוראת תלוש שכר ובודקת אותו מול דיני העבודה בישראל: מנוע של 17
  כללים המחזיר מקור משפטי, רמת ודאות ואומדן פער כספי לכל ממצא.

**Short variants**
- Payslip Check: a rules engine for Israeli payroll compliance.
- בדיקת תלוש שכר: תשובה ברורה אם התלוש שלך תקין.

---

## Open Graph

```html
<meta property="og:type" content="website">
<meta property="og:site_name" content="Portfolio">
<meta property="og:title" content="Payslip Check — Automated Israeli Payslip Auditing">
<meta property="og:description" content="A Hebrew-RTL web app that audits payslips against Israeli labour law: a 17-rule engine with legal sources, confidence levels, and estimated monetary gaps.">
<meta property="og:url" content="https://example.com/projects/payslip">
<meta property="og:image" content="https://example.com/og/payslip.png">
<meta property="og:image:alt" content="Payslip Check — payslip auditing result screen">
<meta property="og:locale" content="he_IL">
<meta property="og:locale:alternate" content="en_US">
```

## Twitter / X

```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Payslip Check — Israeli payslip auditing (case study)">
<meta name="twitter:description" content="17-rule labour-law engine, date-versioned legal parameters, structured AI extraction. Next.js + TypeScript, Hebrew RTL.">
<meta name="twitter:image" content="https://example.com/og/payslip.png">
<meta name="twitter:image:alt" content="Payslip Check result screen with findings and estimated gap">
```

---

## JSON-LD (SoftwareApplication + CreativeWork)

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "SoftwareApplication",
  "name": "Payslip Check",
  "alternateName": "בדיקת תלוש שכר",
  "applicationCategory": "BusinessApplication",
  "operatingSystem": "Web",
  "inLanguage": "he",
  "description": "A Hebrew-RTL web application that extracts an Israeli payslip and audits it against labour law and extension orders, returning findings with legal sources, confidence levels, and estimated monetary gaps.",
  "featureList": [
    "Payslip extraction (PDF text, OCR, structured AI extraction)",
    "17-rule labour-law engine",
    "Date-versioned legal parameters with in-app overrides",
    "Per-finding legal source, confidence, and estimated monetary gap",
    "Printable report, history, and multi-month comparison"
  ],
  "author": { "@type": "Person", "name": "Your Name" },
  "url": "https://example.com/projects/payslip"
}
</script>
```

Optional, to describe the case-study page itself:

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "CreativeWork",
  "headline": "Payslip Check — Case Study",
  "about": "Automated Israeli payslip auditing",
  "inLanguage": "en",
  "keywords": "payslip audit, Israeli labour law, rules engine, Next.js, TypeScript, RTL",
  "author": { "@type": "Person", "name": "Your Name" }
}
</script>
```

---

## Keywords

`payslip check`, `בדיקת תלוש שכר`, `Israeli labour law`, `דיני עבודה`, `payroll compliance`,
`rules engine`, `minimum wage check`, `overtime 125% 150%`, `pension contribution audit`,
`convalescence pay דמי הבראה`, `Next.js`, `TypeScript`, `Tailwind`, `RTL Hebrew`, `OCR`,
`structured AI extraction`, `PostgreSQL`, `AES-256-GCM`, `full-stack case study`.

---

## Accessibility & i18n notes for the page

- Set `<html lang="he" dir="rtl">` for the Hebrew page; `lang="en" dir="ltr"` for the English writeup.
- Ensure the OG image has meaningful `alt` text.
- Provide both Hebrew and English titles/descriptions where the platform supports alternates.
