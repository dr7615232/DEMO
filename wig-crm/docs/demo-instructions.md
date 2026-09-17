# Demo Instructions — Wig CRM (אומנות בפאות)

Two ways to demo this project: the **portfolio demo** (a faithful reconstruction, safe for
public sharing) and the **real application** (private, real design, needs care with data).

---

## A. The portfolio demo (recommended for showing off)

`../demo/index.html` in this package is a self-contained reconstruction of the real system's
screens, in Hebrew RTL, with **example data only**. It is completely safe to share.

**To run it:** double-click `demo/index.html`, or open it in any modern browser. No build,
no server, no internet.

**Suggested 90-second walkthrough:**

1. **Dashboard** — point out today's appointments, the auto-reminder strip, and "this
   month's profit" computed live.
2. **Calendar** — note the two-stage booking idea: booked by phone with a service, the exact
   sub-service is captured on arrival (see the "ממתין לבחירת שירות בהגעה" appointment).
3. **Finances → תמונת מצב** — the four cards: income, **supplier cost**, other expenses, and
   **net profit**; plus open debts, the six-month trend, and the by-service breakdown.
4. **Finances → ספקים** — the rolling supplier account: charges minus payments = balance,
   maintained automatically.
5. **Settings** — show that cost & supplier hang off the price list, and call out the line:
   *the customer only ever sees the price, never the cost.*

The demo is also published as a private Claude Artifact — see `project-data.json → links.liveDemo`.

---

## B. The real application (private)

The delivered product is one HTML file. If you have a copy of the real file (kept **outside**
this portfolio repo):

1. Double-click it to open. It loads instantly and works offline.
2. On first open it seeds an example configuration (price list, suppliers, statuses). You can
   explore freely; nothing is sent anywhere.
3. **For a clean demo, do not use a file that contains real client data.** If it does, use
   the in-app reset, or start from a fresh copy, before demoing.
4. Optional file-sync mode uses the browser's File System Access API (Chromium-based
   browsers). You can skip it for a demo and rely on `localStorage`.

> ⚠️ Never screen-share or screenshot the real app while it holds real customer records
> (names, phone numbers, balances). Use the portfolio demo in `demo/` for anything public.

---

## Recording tips

- Set the browser zoom to ~100–110% and use a 1280×800 window for clean framing.
- Record in Hebrew RTL; keep the cursor slow.
- The best single "wow" moment is recording a sale and then flipping to **ספקים** and
  **תמונת מצב** to show the supplier balance and profit already updated, with nothing typed.
