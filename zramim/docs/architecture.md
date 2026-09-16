# Zramim — Architecture

All diagrams are [Mermaid](https://mermaid.js.org/). GitHub, GitLab, Obsidian, and most
static-site generators render them inline. For a portfolio page, either keep them as Mermaid
code blocks (if your site renders Mermaid) or export them to SVG.

---

## 1. System context

How the app sits between the people who use it and the external services it integrates.

```mermaid
graph TB
    subgraph Users
        MGR["Manager (מנהלת)"]
        SEC["Secretary (מזכירה)"]
        DEV["Developer (hidden identity)"]
        CUST["Customer — on the phone"]
    end

    subgraph App["Zramim — Next.js 15 (App Router)"]
        WEB["Web dashboard<br/>Server Components + Server Actions"]
        IVRAPI["/api/ivr<br/>stateless IVR endpoint"]
        CRONAPI["/api/cron/*<br/>secured job endpoints"]
    end

    subgraph Data["Supabase (PostgreSQL)"]
        DB[("Tables + RLS<br/>~24 tables · ~48 policies")]
        RPC["Postgres functions<br/>register / freeze / reschedule ..."]
        AUTH["Supabase Auth"]
    end

    subgraph External["External services (client-owned)"]
        YEMOT["Yemot HaMashiach<br/>telephony / IVR / TTS"]
        NEDARIM["Nedarim Plus<br/>card clearing"]
        EMAIL["Email<br/>Resend / SMTP"]
        DICTA["Dicta Nakdan API<br/>Hebrew vowelization"]
    end

    GHA["GitHub Actions<br/>hourly cron"]

    MGR --> WEB
    SEC --> WEB
    DEV --> WEB
    CUST -->|"phone call"| YEMOT
    YEMOT <-->|"HTTP per keypress<br/>(shared secret)"| IVRAPI

    WEB --> AUTH
    WEB --> DB
    WEB --> RPC
    IVRAPI --> DB
    IVRAPI --> RPC
    IVRAPI --> YEMOT
    IVRAPI --> DICTA
    IVRAPI -->|"card extension"| NEDARIM
    CRONAPI --> DB
    CRONAPI --> EMAIL
    CRONAPI --> YEMOT
    GHA -->|"x-cron-secret"| CRONAPI

    WEB --> EMAIL
```

---

## 2. Core data model

The relational spine. A **group** is the recurring weekly schedule; a **class_instance** is
one concrete occurrence; a **booking** ties a subscription to an occurrence; the **ledger**
tracks money at the customer level.

```mermaid
erDiagram
    CUSTOMERS ||--o{ SUBSCRIPTIONS : has
    CUSTOMERS ||--o{ LEDGER_ENTRIES : "billed / pays"
    CUSTOMERS ||--o{ BOOKINGS : "attends"
    CUSTOMERS ||--o{ WAITLIST_ENTRIES : "waits for"

    SUBSCRIPTION_TYPES ||--o{ SUBSCRIPTIONS : "priced from (snapshot)"
    SUBSCRIPTIONS ||--o{ BOOKINGS : "schedules N"
    SUBSCRIPTIONS ||--o{ LEDGER_ENTRIES : "charged to"

    POOLS ||--o{ GROUPS : "hosts"
    GROUPS ||--o{ CLASS_INSTANCES : "recurs into"
    GROUPS ||--o{ WAITLIST_ENTRIES : "queued for"
    GROUPS ||--o{ GROUP_CODES : "registered via"
    CLASS_INSTANCES ||--o{ BOOKINGS : "booked by"
    CLASS_INSTANCES ||--o| CLASS_INSTANCES : "rescheduled_to"

    CUSTOMERS {
        uuid id PK
        text full_name
        text phone "not unique"
        text id_number
        text email
        enum notify_channel "email|phone|both"
        enum status
        timestamptz health_declared_at
    }
    SUBSCRIPTIONS {
        uuid id PK
        int lesson_count "snapshot"
        numeric price "snapshot"
        date started_on "first real lesson"
        enum status "active|frozen|completed|cancelled"
        int frozen_balance
        timestamptz freeze_expires_at
    }
    BOOKINGS {
        uuid id PK
        enum status "scheduled|attended|cancelled|replaced"
        enum attendance_kind "regular|trial|replacement"
        bool is_override
    }
    LEDGER_ENTRIES {
        uuid id PK
        enum direction "charge|payment|credit|refund"
        numeric amount
        uuid subscription_id "nullable"
    }
    GROUPS {
        uuid id PK
        enum department "swimming|water_aerobics"
        enum kind "group|private"
        int capacity "default 12"
        smallint weekday
        time start_time
        bool is_future
    }
```

*Not shown (peripheral):* `phone_settings`, `calls`, `voicemails`, `ivr_nav`,
`debt_acknowledgments`, `debt_escalation_calls`, `email_settings`, `email_templates`,
`report_recipients`, `automation_schedules`, `notifications_log`, `business_errors`,
`app_logs`, `app_developers`, `products`, `product_sales`, `expenses`, `holidays`,
`broadcasts`, `pending_phone_messages`, `group_codes`.

---

## 3. Atomic subscription registration (request flow)

Why registration is one transactional RPC rather than several client writes.

```mermaid
sequenceDiagram
    autonumber
    participant UI as Server Action
    participant JS as schedule.ts (TS)
    participant PG as register_subscription() (Postgres)
    participant T as bookings trigger

    UI->>JS: compute next N occurrences<br/>(Asia/Jerusalem, skip holidays)
    JS-->>UI: timestamptz[] occurrences
    UI->>PG: register_subscription(customer, type, group, occurrences[])
    activate PG
    Note over PG: BEGIN (single transaction)
    PG->>PG: load subscription_type (price, lesson_count)
    PG->>PG: derive started_on = first occurrence (Jerusalem)
    PG->>PG: INSERT subscription (price/count snapshotted)
    loop each occurrence
        PG->>PG: get-or-create class_instance<br/>(ON CONFLICT group_id, starts_at)
        PG->>T: INSERT booking
        T-->>PG: enforce_group_capacity()<br/>(reject if full unless override)
    end
    PG->>PG: INSERT ledger charge = price
    Note over PG: COMMIT — all or nothing
    deactivate PG
    PG-->>UI: new subscription id
```

---

## 4. Stateless IVR call flow

Yemot keeps no server-side memory; it re-sends every captured variable on each request. The
app reconstructs navigation from a per-call stack keyed by call-ID.

```mermaid
sequenceDiagram
    autonumber
    participant C as Caller
    participant Y as Yemot (PBX)
    participant API as /api/ivr
    participant NAV as ivr_nav (per-call stack)
    participant DB as Supabase

    C->>Y: dials the line / presses a key
    Y->>API: HTTP GET/POST (all prior vars + ApiCallId + secret)
    API->>API: verifyIvrSecret (timingSafeEqual)
    API->>NAV: load stack for ApiCallId
    alt "*" pressed
        API->>NAV: clear stack (→ main menu)
    else "0" pressed
        API->>NAV: pop (→ previous menu)
    else digit
        API->>NAV: push {var, value}
    end
    API->>DB: resolve caller / data (identify, balance, groups...)
    DB-->>API: rows
    API-->>Y: text/plain commands<br/>(id_list_message / read / go_to_folder)
    Y-->>C: speaks Hebrew TTS, captures next keypress
    Note over API,Y: For card payment / voicemail / terms,<br/>API routes to a native Yemot extension and<br/>stamps a marker so the return is handled, not re-run.
```

---

## 5. Automation & scheduling

One hourly external trigger fans out to timed, idempotent jobs.

```mermaid
graph LR
    GHA["GitHub Actions<br/>cron: 0 * * * * (hourly)"] -->|"x-cron-secret"| DISP["/api/cron/dispatch"]
    DISP -->|"reads"| SCHED[("automation_schedules<br/>+ code defaults")]
    DISP -->|"jerusalemNow() gate<br/>+ last_run_on dedup"| FIRE{fire?}
    FIRE -->|daily 03:00| EF["expire-freezes → RPC"]
    FIRE -->|daily 06:00| DA["debt-alerts"]
    FIRE -->|daily 06:00| RR["renewal-reminders"]
    FIRE -->|Sun 07:00| WD["weekly-debts"]
    FIRE -->|Thu 07:00| TR["thursday-report"]
    FIRE -->|day-1 07:00| MI["monthly-income"]
    DISP -->|every hour| DE["debt-escalation<br/>(self-throttling)"]
    DISP -->|at 03:00| PURGE["log/retention purge"]
    DA --> CH{per-customer<br/>notify_channel}
    CH -->|email| EMAIL["Resend / SMTP"]
    CH -->|phone| YEMOT["Yemot tzintuk + queued message"]
    DA --> LOG[("notifications_log")]
```

---

## 6. Authorization — defense in depth

```mermaid
graph TB
    REQ["Request"] --> MW["middleware.ts<br/>refresh session · redirect /login"]
    MW --> GUARD["Server-side guards<br/>requireProfile / requireManager / requireDeveloper"]
    GUARD -->|"no session"| LOGIN["/login"]
    GUARD -->|"blocked / inactive"| BLOCKED["/blocked"]
    GUARD -->|"secretary hits manager page"| DASH["/dashboard"]
    GUARD --> QUERY["Supabase query (anon key + user JWT)"]
    QUERY --> RLS{"Row-Level Security"}
    RLS -->|"is_staff()"| OPS["operational tables ✓"]
    RLS -->|"is_manager()"| FIN["expenses / finances ✓ (manager only)"]
    RLS -->|"fails is_active"| DENY["0 rows"]

    style RLS fill:#1B6E9B,color:#fff
    style DENY fill:#E05A79,color:#fff
```

**Key point:** the same `is_active` + role invariants are enforced at *both* the application
layer (redirects) and the database layer (RLS). API routes (`/api/*`) are excluded from
session middleware and secured by their own shared-secret tokens (`IVR_SECRET`, `CRON_SECRET`)
with constant-time comparison.
