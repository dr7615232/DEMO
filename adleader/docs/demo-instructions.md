# Demo Instructions — AdLeader

Two self‑contained deliverables reconstruct the product for a portfolio. Neither needs a build step,
a server, or the original repository.

---

## 1. Interactive demo — `../demo/index.html`

A single, offline HTML file that reconstructs AdLeader's real screens in Hebrew (RTL) with the actual
design system (navy `#07306b` + cyan `#00b7c7`, Assistant/Rubik/Heebo fonts) and example data.

**Open it:**
- Locally: double‑click `adleader/demo/index.html` (or open it in any modern browser). It ships with
  `adleader-logo.png` and `adleader-icon.png` beside it — keep those two files next to the HTML.
- Live (Claude Artifact): **https://claude.ai/artifact/PHTUdUubAyg8sJgdJpPFsg**
  *(private link — opens when you're signed in to your Claude account.)*

**What to click through (suggested 90‑second tour):**
1. **מסך ראשי (Dashboard)** — the headline story: pipeline KPIs, "where leads come from", a money
   summary, and the attribution breakdown (how many leads were auto‑matched to a source).
2. **דוחות (Reports)** — the core value. Toggle **לפי מקור פרסום / לפי קמפיין** and point at the
   green/red "מדד" column: one source returns ₪‑per‑shekel, the printed‑ad phone source is red and
   hasn't closed a deal. Honest measurement, not clicks.
3. **מסלול הפנייה (CRM pipeline)** — drag‑ready cards across stages, each showing its source.
4. **הפניות שלי (Leads)** — every inquiry with its source, campaign, attribution method and status.
5. **הודעות (Communications)** — WhatsApp + email threads attached to a lead.
6. **פרסומים / קישורים / חיבורים** — campaigns, smart tracking links, and integrations.

Everything is client‑side; navigation switches screens with no network calls. A fixed badge reads
**"תצוגת דמו · נתונים לדוגמה"** so viewers know the data is illustrative.

---

## 2. Hebrew project page — `../site/index.html`

A designed, RTL Hebrew case‑study page (cream/olive/lime minimalist style) that tells the story to a
**business owner**, benefit‑first, with a single call‑to‑action to the live demo.

**Open it:**
- Locally: double‑click `adleader/site/index.html` (fully self‑contained, only Google Fonts loaded).
- Live (Claude Artifact): **https://claude.ai/artifact/5N3aefwnPYKhAHpGjB3FuZ**

Read it top to bottom: masthead → the demo bar → numbered sections `01 הבעיה` … `06 התוצאה`, with the
strongest feature (automatic attribution) as the full‑width section `05`.

---

## Presenting it

- **Lead with the demo.** "The easiest way to understand it is to see it" — open the dashboard, then
  the reports tab, and let the per‑source ROI make the point.
- **Tell the one‑sentence story:** *every inquiry, from every channel, tied automatically to the ad
  that produced it — so you can see which advertising actually pays.*
- **For a technical audience,** jump to [`technical-deep-dives.md`](./technical-deep-dives.md) and the
  [architecture diagrams](./architecture.md).
- **Data is fictional.** All names, phones, and businesses are invented examples — see the
  de‑identification checklist in [`README.md`](./README.md).

## Regenerating screenshots (optional)

Any headless browser works. Example with Playwright:

```js
const { chromium } = require("playwright");
(async () => {
  const b = await chromium.launch();
  const p = await b.newPage({ viewport: { width: 1300, height: 900 }, deviceScaleFactor: 2 });
  await p.goto("file:///ABSOLUTE/PATH/adleader/demo/index.html");
  await p.screenshot({ path: "dashboard.png" });
  await p.click('[data-screen="reports"]');
  await p.screenshot({ path: "reports.png" });
  await b.close();
})();
```
