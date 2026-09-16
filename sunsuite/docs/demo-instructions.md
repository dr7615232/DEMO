# Demo Instructions — SUNSUITE

The interactive demo is a **single, self-contained HTML file** — no build, no
server, no dependencies beyond web fonts. It reconstructs the real system's
screens (admin panel + public booking site) in Hebrew (RTL) with example data.

## Run it

- **Locally:** open [`../demo/index.html`](../demo/index.html) in any modern browser
  (double-click, or `File → Open`). Everything runs client-side.
- **Live (Artifact):** the published link is in the top-level repository `README.md`.
- Optional local server (nicer for sharing on a LAN):
  ```bash
  cd sunsuite/demo && python3 -m http.server 8080
  # then open http://localhost:8080
  ```

## What to show (a 2-minute walkthrough)

The header has a **mode switch** (top-right): *מערכת ניהול* (admin) and *אתר ההזמנות* (public site).

**Admin — the operations story:**
1. **סקירה (Overview)** — pending requests, approved bookings, this week's
   check-ins and upcoming cleanings (dates shown in both Gregorian and Hebrew).
2. **הזמנות (Bookings)** — filter/search the queue; note the amber row flagged
   *"בקשה חופפת"* (overlap detected). Click **אשר** on a pending row and watch the
   toast: the booking is approved, a cleaning is auto-created, it syncs to the
   calendar, and the guest is emailed.
3. **יומן (Calendar)** — month grid with color-coded stays, cleanings and blocks;
   each day shows its Hebrew date. Legend at the top.
4. **נקיונות (Cleanings)** — auto-generated windows, sized by guest count (see the
   rule chart at the bottom).
5. **סוגי נופש (Properties)** — per-property editor with tabs (general / cleaning
   rules / media & pricing), check-in/checkout & motzaei-Shabbat times.
6. **דוחות (Reports)** — KPIs and charts by property, type and month.
7. **הגדרות (Settings)** — Google Calendar, email, and the Shabbat-mode card
   (there's a **"תצוגה מקדימה של מסך שבת"** button).

**Public site — the guest story:**
1. Switch to **אתר ההזמנות**. Scroll to the property showcase and gallery.
2. In **בדיקת זמינות**, pick a **check-in** then **check-out** date on the calendar —
   greyed dates are unavailable. Selected dates appear below in Gregorian **and**
   Hebrew.
3. Click **בדוק זמינות** → the "available" banner appears and the booking form opens
   with a live price summary. Toggle **הזמנה לפי שעות** to see the hourly time picker.
4. From Settings (or the same view) you can preview the **Shabbat screen** that
   replaces the site during Shabbat.

## Notes for presenting

- All names, phones and emails are **fictional** (`example.com`). Say so if asked.
- The demo simulates the flows client-side; in production these are real server
  functions against Supabase with Google Calendar and email integrations.
- Works great on a phone — it's fully responsive and RTL.
