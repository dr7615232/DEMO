# Demo Instructions — תוכי פרינט · בונה הקטלוג

The demo is a **single, self-contained HTML file** — no build, no server, no data
saved anywhere. It runs offline by double-click.

## Run it

- **Locally:** open [`../demo/index.html`](../demo/index.html) in any modern browser.
- **Live (Artifact):** https://claude.ai/artifact/N6HuWafMG1kLRiFnvZsX6p
- Optional local server (nicer for sharing on a LAN):
  ```bash
  cd tuki/demo && python3 -m http.server 8080   # then open http://localhost:8080
  ```

## A 2-minute walkthrough

The top tab bar switches the three screens.

1. **בונה הקטלוג (Builder)**
   - Products are grouped by category. Use **search** and the **category filter** to narrow.
   - On a card: edit the **title** inline, set **מחיר ליח׳** (per-unit) and **25+** (bulk),
     then tap **הוסף לקטלוג** to include it (card turns blue).
   - Try **בחר הכל / נקה** per category, **+ הוספת מוצר** (add modal), and the contact
     details modal. Watch the **שומר… → נשמר** autosave indicator and the floating
     summary bar (count + sum) at the bottom.
2. **שער הקטלוג (Cover + catalog)** — tap **הפקת קטלוג / תצוגה מקדימה**. See the festive
   cover with the logo, the **Gregorian + Hebrew date**, voucher/CTA badges, and the
   selected products laid out by category with the right prices.
3. **טופס הזמנה (Order form)** — the customer only sets **quantities**; the line price
   switches to the **bulk tier at 25+**, totals update, and there's a **VAT** toggle.

## Notes for presenting

- All products, prices, phone and email are **example data**. Say so if asked.
- State is **in memory only** — refreshing resets the demo. That's intentional for a
  portfolio demo; a production build would persist and export the catalog.
- Fully responsive and RTL; works well on a phone.
