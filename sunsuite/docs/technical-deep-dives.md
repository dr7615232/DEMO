# Technical Deep-Dives — SUNSUITE

Four engineering problems worth a closer look. Code excerpts are taken from the
real implementation and lightly trimmed for readability.

---

## 1. Availability that accounts for the cleaning buffer

A naive availability check compares a candidate stay against existing stays. That
isn't enough here: a unit isn't truly free if the **cleaning window** after the
previous guest still overlaps the new stay. So the engine computes the cleaning
range for the *candidate* booking, extends the search window to cover it, and
reports **which** kind of conflict occurred (stay vs. cleaning) so the UI can
explain it.

```ts
export const checkAvailability = createServerFn({ method: "POST" })
  .inputValidator((i: unknown) => checkSchema.parse(i))
  .handler(async ({ data }) => {
    const { data: prop } = await supabaseAdmin
      .from("properties")
      .select("overnight_checkin_time, overnight_checkout_time, motzaei_shabbat_checkout_time, cleaning_rules")
      .eq("slug", data.propertySlug)
      .maybeSingle();
    const times = propertyTimes(prop);
    const candidate = bookingToRange({ /* dates, times, extra hours */ }, times);

    // The cleaning window the candidate itself would need afterwards.
    const candidateCleaning = cleaningRangeAfter(candidate, prop?.cleaning_rules, data.guestsCount);
    const checkEnd = candidateCleaning?.end ?? candidate.end;

    const res = await getAvailability({
      data: { propertySlug: data.propertySlug, monthStart: data.startDate,
              monthEnd: localDateISO(checkEnd), excludeBookingId: data.excludeBookingId ?? null },
    });
    const busy = res.busy.map((b) => ({ start: new Date(b.start), end: new Date(b.end) }));

    const stayConflict = busy.some((r) => overlap(candidate, r));
    const cleaningConflict = !stayConflict && !!candidateCleaning && busy.some((r) => overlap(candidateCleaning, r));
    return {
      available: !stayConflict && !cleaningConflict,
      stayConflict, cleaningConflict,
      reason: cleaningConflict ? "cleaning" : stayConflict ? "stay" : null,
    };
  });
```

The same primitive powers three things: the live check in the booking widget, a
gate at booking creation, and a re-check at approval time. It also drives
`getAvailableHourlyDurations`, which maps 2–23 hour options over a start time and
keeps only the durations whose stay **and** cleaning both fit.

**Why it matters:** availability bugs are the worst kind in a booking system —
they lead to real double-bookings. Making the cleaning buffer a first-class part
of "busy" removes an entire class of them.

---

## 2. Cleaning windows sized by guest count

Cleaning time isn't fixed — a 12-person group needs far longer than a couple. Each
property stores an ordered list of rules as JSON, and a small resolver picks the
matching band for a guest count, falling back safely to a sensible default.

```ts
export const DEFAULT_CLEANING_RULES: CleaningRule[] = [
  { min_guests: 1,  max_guests: 2,    duration_minutes: 60 },
  { min_guests: 3,  max_guests: 6,    duration_minutes: 90 },
  { min_guests: 7,  max_guests: 10,   duration_minutes: 120 },
  { min_guests: 11, max_guests: null, duration_minutes: 180 },
];

export function cleaningMinutesForGuests(value: unknown, guests: number) {
  const rules = normalizeCleaningRules(value);
  const n = Math.max(1, Number(guests) || 1);
  const match = rules.find((r) => n >= r.min_guests && (r.max_guests == null || n <= r.max_guests));
  return match?.duration_minutes ?? rules[rules.length - 1]?.duration_minutes ?? 60;
}
```

`normalizeCleaningRules` coerces, validates and sorts whatever is stored (rules are
editable in the admin UI) so a malformed row can never crash the resolver. On
approval, `insertCleaningBuffer` computes the checkout moment (honoring the
motzaei-Shabbat checkout for Saturday departures and any purchased late hours),
adds the resolved minutes, and writes the buffer:

```ts
const durationMinutes = cleaningMinutesForGuests(prop?.cleaning_rules, Number(booking.guests_count) || 1);
await supabaseAdmin.from("cleaning_buffers").delete().eq("booking_id", booking.id);
if (durationMinutes <= 0) return;
const endISO = new Date(startMs + durationMinutes * 60_000).toISOString();
await supabaseAdmin.from("cleaning_buffers").insert({
  property_id: booking.property_id, booking_id: booking.id,
  start_datetime: startISO, end_datetime: endISO,
});
```

**Why it matters:** the owner configures policy once (per property), and every
future booking schedules the right amount of cleaning with zero manual thought.

---

## 3. The approval automation, made resilient

Approving a booking is a small orchestration: re-verify availability, flip status
(guarded so a double-click can't double-fire), regenerate the cleaning buffer,
sync both events to Google Calendar, and email the guest. The external steps are
deliberately **best-effort** — a Calendar outage or a bounced email must never
prevent an approval from being recorded.

```ts
if (data.status === "approved") {
  const avail = await checkAvailability({ data: { /* … */ excludeBookingId: booking.id } });
  if (!avail.available) throw new Error(availabilityError(avail, "…זמן חופף"));
}

const { data: updatedRows } = await supabaseAdmin
  .from("bookings")
  .update({ status: data.status, approved_at: /* … */, approved_by: userId })
  .eq("id", data.id)
  .neq("status", data.status)   // idempotency guard: no-op if already in this status
  .select("id");
if (!updatedRows?.length) return { ok: true, skipped: true };

if (data.status === "approved") {
  await supabaseAdmin.from("cleaning_buffers").delete().eq("booking_id", data.id);
  const cleaning = await insertCleaningBuffer(approvedBooking);
  await syncApprovedBookingToGoogleCalendar(approvedBooking, userId);
  if (cleaning) await syncCleaningBufferToGoogleCalendar(cleaning, userId);
  try {
    const { notifyCustomerBookingStatus } = await import("./notifications.server");
    await notifyCustomerBookingStatus(approvedBooking, "approved");
  } catch (e) {
    console.warn("[updateBookingStatus] customer approval email failed (ignored)", e);
  }
}
```

Two details worth calling out: the `.neq("status", data.status)` guard makes the
update **idempotent**, and the notification module is dynamically `import()`-ed so
the email/SMTP dependency stays out of the hot path until it's actually needed.

**Why it matters:** the owner clicks once and trusts that the guest, the calendar
and the cleaning crew are all in sync — without the whole action being hostage to
a third-party's uptime.

---

## 4. Automatic Shabbat mode from real astronomical times

The business must not take bookings on Shabbat. Rather than a manual switch,
SUNSUITE asks Hebcal for candle-lighting and havdalah times **at the property's
coordinates**, caches them briefly, and closes the public site when "now" falls
inside the window. Any failure fails **open** to the normal site (never blocking
legitimate bookings by mistake).

```ts
export async function getShabbatModeStatus(now = new Date()): Promise<ShabbatModeStatus> {
  const settings = await getSiteSettings();
  const message = settings.shabbatModeMessage;
  if (cache && cache.expiresAt > now.getTime()) return cache.value;

  try {
    const response = await fetch(hebcalUrl(), { headers: { accept: "application/json" } });
    if (!response.ok) return inactive(message);
    const data = await response.json();
    const candles  = data.items?.find((i) => i.category === "candles" && i.date);
    const havdalah = data.items?.find((i) => i.category === "havdalah" && i.date);
    if (!candles?.date || !havdalah?.date) return inactive(message);

    const startsAt = new Date(candles.date), endsAt = new Date(havdalah.date);
    const active = startsAt.getTime() <= now.getTime() && now.getTime() <= endsAt.getTime();
    const value = { active, message, startsAt: startsAt.toISOString(), endsAt: endsAt.toISOString() };
    cache = { expiresAt: now.getTime() + CACHE_MS, value };
    return value;
  } catch {
    return inactive(message);   // fail open
  }
}
```

The home route awaits this in its loader and short-circuits to a Shabbat screen
before it even fetches properties — so during Shabbat the site does no booking
work at all.

**Why it matters:** the product respects the client's values automatically, with
correct times year-round and no reliance on anyone remembering to flip a switch.
