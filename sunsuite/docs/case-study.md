# Case Study — SUNSUITE Booking & Operations Platform

## Summary

SUNSUITE is a production, Hebrew-RTL platform that turns a manually-run group of
luxury vacation-rental and spa complexes into a self-service booking business.
Guests discover a property, check real availability, price a stay (overnight or
by-the-hour), sign the terms, and submit a request — all online, in Hebrew,
around the clock. The owner approves with one click, and the system takes care
of everything that used to be manual: it schedules the cleaning window, pushes
the booking to the owner's Google Calendar, and emails the guest.

---

## The problem

Running several short-stay complexes means juggling dozens of small facts at
once: who requested which dates, who paid, which unit is free, and when it needs
cleaning between guests. When all of that lives in a personal calendar, a chat
thread and the owner's head, things slip:

- Requests arrive by phone/WhatsApp at all hours and get lost.
- It's easy to accidentally approve two guests for overlapping dates.
- Cleaning time between stays is forgotten or double-booked.
- Every "is this date free?" needs a manual check — and only during working hours.
- A spa rented **by the hour** and suites rented **by the night** need different
  availability and pricing logic, which is hard to track by hand.
- The business is religiously observant: it must **not** take bookings on Shabbat.

## The solution

A single platform with two faces:

1. **A public booking site** (Hebrew, RTL, mobile-first) with a hero, per-property
   showcases, gallery, FAQ, and a booking widget that checks availability, prices
   the stay live, and captures a digital signature on the terms.
2. **An admin operations panel** — overview dashboard, bookings queue with overlap
   detection, month calendar, auto-generated cleanings, manual blocks, property &
   pricing editor, customers, reports, and settings.

The core idea: **the guest self-serves, the owner approves, and the system does
the rest.** Availability accounts not just for existing stays but for the
**cleaning buffer** each stay needs. On approval, the system re-verifies
availability, creates the cleaning window sized to the guest count, syncs both
the stay and the cleaning to Google Calendar, and notifies the guest — each step
wrapped so a single integration hiccup never blocks the booking.

A dedicated **Shabbat mode** closes the public site automatically from candle-
lighting to havdalah, using real astronomical times for the property's location.

## My role

End-to-end design and implementation:

- Modeled the domain (properties, bookings, cleaning buffers, manual blocks,
  pricing rules, extras) on Supabase/PostgreSQL with Row-Level Security.
- Built the SSR app on TanStack Start + Router with type-safe **server functions**
  and Zod validation at every boundary.
- Implemented the availability engine (overlap + cleaning-buffer conflict, hourly
  and overnight, motzaei-Shabbat checkout, recurring series, weekly notice windows).
- Wired the automation chain (approval → cleaning → Google Calendar → email).
- Designed the RTL design system (Tailwind v4 tokens, shadcn/ui) and both UIs.
- Integrated Google Calendar, Cloudinary media, Resend/SMTP, and Hebcal.

## Outcome

- Guests book unattended, day or night; requests no longer get lost in chat.
- Overlapping approvals are caught before they happen — including the cleaning gap.
- Cleaning is scheduled automatically and appears in the owner's own calendar.
- Guests get instant, consistent confirmations.
- The business stays true to its values: no bookings are taken on Shabbat, with
  no manual toggling.

## Selected challenges

- **Availability that respects cleaning.** A date isn't "free" if the previous
  stay's cleaning window still overlaps it. The engine computes the cleaning
  range for the candidate booking and checks both.
- **Two booking modes.** Overnight (check-in/checkout times, motzaei-Shabbat
  checkout, multi-night) and hourly (15-minute steps, 2-hour minimum, crossing
  midnight) share one model and one availability path.
- **Resilient side effects.** Calendar sync and email are best-effort: they're
  isolated so an approval still succeeds if an external service is briefly down.
- **Correct time handling.** Naive local booking strings vs. UTC timestamps for
  cleanings/blocks are reconciled carefully so the calendar reads chronologically.
