# Code Snippets — SUNSUITE

Selected, publish-safe excerpts. All are trimmed for clarity; none contain
secrets — server code reads credentials from environment variables only.

---

## 1. A type-safe, validated server function (booking creation)

Every mutation is a TanStack `createServerFn` with a Zod validator at the
boundary. Creation re-checks availability, computes the price **on the server**
(never trusting the client), inserts as `pending`, and notifies the owner.

```ts
export const createBooking = createServerFn({ method: "POST" })
  .inputValidator((i: unknown) => parseServerInput(bookingCreateSchema, i))
  .handler(async ({ data }) => {
    const { data: prop } = await supabaseAdmin
      .from("properties").select("id, slug, name, supports_hourly")
      .eq("id", data.propertyId).maybeSingle();
    if (!prop) throw new Error("סוג הנופש לא נמצא");
    if (data.bookingType === "hourly" && !prop.supports_hourly)
      throw new Error("סוג הנופש לא תומך בהזמנה לפי שעות");

    const avail = await checkAvailability({ data: { /* property, dates, guests, extra hours */ } });
    if (!avail.available) throw new Error(availabilityError(avail, "התאריך או השעה שנבחרו אינם זמינים"));

    // Server is the source of truth for price:
    const serverPrice = await getServerPrice({ /* … */ });

    const { data: inserted, error } = await supabaseAdmin.from("bookings").insert({
      property_id: data.propertyId, booking_type: data.bookingType,
      customer_name: data.customerName, phone: data.phone, email: data.email,
      guests_count: data.guestsCount, start_date: data.startDate, end_date: data.endDate,
      start_time: data.startTime || null, end_time: data.endTime || null,
      status: "pending", terms_accepted: true, terms_accepted_at: new Date().toISOString(),
      digital_signature: data.digitalSignature, signed_terms_version: data.signedTermsVersion,
      calculated_price: serverPrice.calculatedPrice, price_breakdown: serverPrice.priceBreakdown,
      extras_selection: extrasSelection(data),
    }).select("id").single();
    if (error) throw new Error(error.message);

    try {
      const { notifyAdminOfNewBookingV2 } = await import("./notifications.server");
      await notifyAdminOfNewBookingV2({ id: inserted.id, /* summary fields */ });
    } catch (e) {
      console.warn("[createBooking] notify failed (ignored)", e);
    }
    return { id: inserted.id, status: "pending" as const };
  });
```

**Notes:** hourly bookings are validated to 15-minute steps and a 2-hour minimum;
overnight requires `end_date > start_date`. The digital signature and terms
version are stored with the booking for a clear audit trail.

---

## 2. Live admin screens via Supabase Realtime

Admin views subscribe to Postgres changes and invalidate their React Query cache,
so the dashboard, bookings queue and calendar stay live without polling.

```ts
useEffect(() => {
  const ch = supabase
    .channel("admin-overview-rt")
    .on("postgres_changes", { event: "*", schema: "public", table: "bookings" },
        () => qc.invalidateQueries({ queryKey: ["admin-overview"] }))
    .on("postgres_changes", { event: "*", schema: "public", table: "cleaning_buffers" },
        () => qc.invalidateQueries({ queryKey: ["admin-overview"] }))
    .subscribe();
  return () => { supabase.removeChannel(ch); };
}, [qc]);
```

---

## 3. Overlap detection surfaces conflicting pending requests

The bookings queue flags pending requests that overlap each other (including
across the linked suite/spa units), so the owner never silently approves a clash.

```ts
const overlapMap = useMemo(() => {
  const pendings = bookings.filter((b) => b.status === "pending");
  const map = new Map<string, any[]>();
  for (let i = 0; i < pendings.length; i++) {
    const a = pendings[i], ar = bookingRange(a);
    const aGroup = a.booking_type === "overnight"
      ? linkedSlugs(a.properties?.slug || "") : [a.properties?.slug || ""];
    for (let j = 0; j < pendings.length; j++) {
      if (i === j) continue;
      const b = pendings[j];
      if (b.property_id !== a.property_id &&
          !(b.booking_type === "overnight" && a.booking_type === "overnight"
            && aGroup.includes(b.properties?.slug || ""))) continue;
      const br = bookingRange(b);
      if (ar && br && ar[0] < br[1] && br[0] < ar[1]) {
        (map.get(a.id) ?? map.set(a.id, []).get(a.id))!.push(b);
      }
    }
  }
  return map;
}, [data]);
```

---

## 4. Chronological calendar bucketing across mixed time formats

Bookings store naive local strings; cleanings/blocks store UTC. To render each day
in true chronological order, events are bucketed and sorted by their *local* time
on that specific day — a checkout in the morning sorts before a later check-in.

```ts
function dayEventMinutes(ev: any, dayKey: string): number {
  const s = new Date(ev.start), e = new Date(ev.end);
  const isBooking = ev.kind === "approved" || ev.kind === "pending";
  // Overnight booking on its checkout day → order by the morning checkout time.
  if (isBooking && ev.bookingType !== "hourly" && toISODate(s) !== toISODate(e) && dayKey === toISODate(e))
    return e.getHours() * 60 + e.getMinutes();
  if (toISODate(s) === dayKey) return s.getHours() * 60 + s.getMinutes();
  return 0; // ongoing from an earlier day → all-day, show first
}
```

---

## 5. Design tokens (Tailwind v4, oklch) driving the RTL UI

The warm "sun" palette is defined once as CSS custom properties and mapped to
Tailwind utilities; the document is RTL by default.

```css
@theme inline {
  --color-primary: var(--primary);
  --color-background: var(--background);
  /* … */
}
:root {
  --background: oklch(0.985 0.012 85);  /* warm cream */
  --foreground: oklch(0.22 0.04 60);    /* dark warm brown */
  --primary:    oklch(0.74 0.13 84);    /* sun gold */
  --gradient-sunset: linear-gradient(180deg, oklch(0.66 0.13 78), oklch(0.82 0.13 88));
}
@layer base {
  html { direction: rtl; }
  body { background: var(--color-background); font-family: "Assistant", system-ui, sans-serif; }
  h1, h2, h3, h4 { font-family: "Rubik", "Assistant", sans-serif; }
}
```
