# Architecture — SUNSUITE

SUNSUITE is a server-rendered React application (TanStack Start) whose backend is
a set of type-safe **server functions** talking to Supabase (PostgreSQL) and a
handful of external services. Everything runs on Cloudflare.

## System context

```mermaid
flowchart TB
  guest([Guest / visitor])
  owner([Owner / admin])

  subgraph app["SUNSUITE app — TanStack Start on Cloudflare"]
    site["Public site<br/>(hero, properties, booking widget, Shabbat mode)"]
    admin["Admin panel<br/>(overview, bookings, calendar, cleanings, properties, reports, settings)"]
    fns["Server functions<br/>(Zod-validated, requireAdmin middleware)"]
  end

  subgraph supa["Supabase"]
    db[("PostgreSQL<br/>+ Row-Level Security")]
    auth["Auth"]
    rt["Realtime"]
    storage["Storage"]
  end

  gcal["Google Calendar API"]
  cloud["Cloudinary<br/>(images / video)"]
  mail["Resend / SMTP"]
  hebcal["Hebcal API<br/>(Shabbat times)"]

  guest --> site
  owner --> admin
  site --> fns
  admin --> fns
  fns --> db
  fns --> auth
  admin -. live updates .- rt
  rt --- db
  fns --> gcal
  fns --> mail
  fns --> cloud
  site --> hebcal
  fns --> cloud
```

## Data model (core tables)

```mermaid
erDiagram
  PROPERTIES ||--o{ BOOKINGS : "has"
  PROPERTIES ||--o{ CLEANING_BUFFERS : "has"
  PROPERTIES ||--o{ MANUAL_BLOCKS : "has"
  PROPERTIES ||--o{ PRICING_RULES : "priced by"
  BOOKINGS ||--o| CLEANING_BUFFERS : "generates"

  PROPERTIES {
    uuid id PK
    text slug
    text name
    text tagline
    numeric base_price
    time overnight_checkin_time
    time overnight_checkout_time
    time motzaei_shabbat_checkout_time
    bool supports_hourly
    jsonb cleaning_rules
    text[] images
    text[] gallery_images
    text[] videos
    bool active
  }
  BOOKINGS {
    uuid id PK
    uuid property_id FK
    text booking_type "overnight | hourly"
    text customer_name
    text phone
    text email
    int guests_count
    date start_date
    date end_date
    time start_time
    time end_time
    text status "pending | approved | declined | cancelled"
    numeric calculated_price
    jsonb price_breakdown
    jsonb extras_selection
    int extra_early_hours
    int extra_late_hours
    bool terms_accepted
    text digital_signature
    timestamptz approved_at
  }
  CLEANING_BUFFERS {
    uuid id PK
    uuid property_id FK
    uuid booking_id FK
    timestamptz start_datetime
    timestamptz end_datetime
  }
  MANUAL_BLOCKS {
    uuid id PK
    uuid property_id FK
    text reason
    timestamptz start_datetime
    timestamptz end_datetime
  }
  PRICING_RULES {
    uuid id PK
    uuid property_id FK
    text kind "overnight | extra | hourly-group"
    jsonb config
  }
```

> Notes: linked units (`sunsuite` + `sunspa`) are treated as one availability
> group for overnight bookings so the shared complex can't be double-booked.
> `cleaning_rules` is stored per-property as JSON (guest-count → minutes).

## Key flow 1 — Guest booking request

```mermaid
sequenceDiagram
  participant G as Guest
  participant S as Booking widget
  participant F as createBooking (server fn)
  participant DB as Supabase
  participant M as Email

  G->>S: pick property, dates/times, guests, extras
  S->>F: checkAvailability (live)
  F->>DB: read stays + cleanings + blocks
  F-->>S: available? (stay & cleaning conflicts)
  S->>G: show live price + availability
  G->>S: fill details, sign terms, submit
  S->>F: createBooking (Zod-validated)
  F->>F: re-check availability + server-side price
  F->>DB: insert booking (status = pending)
  F-)M: notify owner of new request (best-effort)
  F-->>G: request received (pending)
```

## Key flow 2 — Owner approves (the automation)

```mermaid
sequenceDiagram
  participant A as Admin
  participant U as updateBookingStatus (server fn)
  participant DB as Supabase
  participant GC as Google Calendar
  participant M as Email

  A->>U: approve booking
  U->>U: requireAdmin + re-check availability (excl. this booking)
  U->>DB: set status = approved (guard: only if changed)
  U->>DB: create cleaning_buffer (minutes by guest count)
  U-)GC: sync stay + cleaning events (best-effort)
  U-)M: send guest approval email (best-effort)
  U-->>A: done
  Note over DB,A: Admin views update live via Supabase Realtime
```

## Key flow 3 — Shabbat mode

```mermaid
flowchart LR
  req([Page load]) --> chk{"getShabbatModeStatus()"}
  chk -->|fetch| hc["Hebcal API<br/>candle-lighting + havdalah<br/>(property coordinates)"]
  hc --> cache["5-min in-memory cache"]
  cache --> active{"now between<br/>candles &amp; havdalah?"}
  active -->|yes| closed["Public site shows<br/>Shabbat screen<br/>(no bookings)"]
  active -->|no| open["Normal booking site"]
```

## Cross-cutting concerns

- **Validation & auth:** every server function validates input with Zod; admin
  actions run behind a `requireAdmin` middleware plus per-property scope checks.
- **Realtime:** admin screens subscribe to Postgres changes so the queue, calendar
  and dashboard refresh without reloads.
- **Time correctness:** overnight bookings use naive local date/time; cleanings and
  blocks use UTC timestamps. Calendar bucketing reads local hours from both so a
  day renders chronologically.
- **Resilience:** external calls (Calendar, email, media) are wrapped so failures
  are logged and ignored rather than failing the core transaction.
