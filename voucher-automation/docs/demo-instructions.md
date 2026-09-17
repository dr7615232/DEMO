# Demo Instructions — שובר אוטומטי

How to open, present, and talk through the interactive demo.

---

## What the demo is

A single, self-contained HTML file (`../demo/index.html`) — no build step, no server, no
dependencies beyond a Google Font loaded over the web. It's a faithful reconstruction of the
system's screens in Hebrew (RTL), filled with **example data only**.

**Live Artifact:** open the published link from the repo's main `README.md` (opens when
you're signed in to your Claude account).

---

## Opening it

- **From a browser:** double-click `../demo/index.html`, or drag it into a browser tab.
- **From the published Artifact:** click the demo link in the main README.
- Best viewed on a laptop/desktop width; it also collapses gracefully to mobile.

No login. Nothing is saved. Refreshing resets everything.

---

## The screens (left sidebar)

**ניהול (Management)**
1. **סקירה כללית** — the dashboard: low-stock alert banner, headline figures, the
   donation→voucher flow strip, recent activity, and vouchers-by-location.
2. **תרומות שהתקבלו** — donations that flowed in automatically, with per-donation voucher status.
3. **שוברים שהופקו** — issued vouchers, each with its unique number, location, donor, and status.
4. **מלאי שוברים** — stock per redemption location, with low-stock highlighted.
5. **התראות מלאי** — open and resolved low-stock alerts.

**חוויית התורם (Donor experience)**
6. **טופס בחירת שובר** — what the donor sees: pick a redemption location. *Interactive.*
7. **השובר שנשלח** — the designed voucher that gets emailed.

**מערכת (System)**
8. **הגדרות ואוטומציה** — the automation toggles and the list of locations with thresholds.

---

## A 60-second walkthrough (suggested script)

1. Start on **סקירה כללית**: "One screen shows the whole operation — what came in, what went
   out, and what needs attention. Notice the low-stock banner at the top: the system warns
   *before* anything runs out."
2. Point at the **flow strip** (01→04): "This is the whole loop, and it runs by itself."
3. Open **תרומות שהתקבלו**: "Every donation lands here automatically. No copy-paste from the
   inbox."
4. Open **טופס בחירת שובר**: "This is all the donor does — choose where to redeem." Click a
   different location to show it's live.
5. Click **קבלת השובר במייל**: it jumps to **השובר שנשלח** — "and within seconds, this
   designed voucher, with its own unique number, is emailed to them." (The location you
   picked on the form carries over.)
6. Finish on **מלאי שוברים / התראות**: "Meanwhile the stock updates itself, and the operator
   is alerted early — so the automatic flow never stalls."

---

## Interactive bits worth clicking

- **Donor form → voucher:** the location chosen on screen 6 updates the voucher on screen 7.
- **Settings toggles:** the automation switches on screen 8 flip on click.
- **Sidebar navigation:** every screen is reachable; the active item is highlighted.

---

## Talking points (benefit-first, no jargon)

- "The operator is no longer *in* the process — they only watch and get alerted."
- "Voucher numbers can never collide, because the system assigns them, not a person."
- "A slow moment, a busy day, or a hundred donations at once — the donor still gets their
  voucher in seconds."

---

## Notes for presenting

- Everything is **example data**. Say so if asked — no real donors, businesses, or numbers.
- The permanent tag "תצוגת דמו · נתונים לדוגמה" is intentional; leave it visible.
- If you record a screen capture, the donor-form → voucher click makes the best 5-second clip.
