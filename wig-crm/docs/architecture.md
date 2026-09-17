# Architecture — Wig CRM (אומנות בפאות)

A single-file, offline-first, dependency-free web application. There is no server, no
database engine, and no build step. This document describes how the one HTML file is
organized, how data flows, and how the core money logic stays consistent.

---

## 1. System context

The entire product is one HTML file the owner opens in a browser. The only optional external
touch points are the browser's own storage APIs and a user-initiated email client (for
sending a greeting). Nothing leaves the machine automatically.

```mermaid
graph TD
  subgraph Browser["Owner's browser (offline, file://)"]
    APP["index.html<br/>(HTML + CSS + ~1,950 lines vanilla JS)"]
    LS[("localStorage<br/>crm_omanut_bapeot_v1")]
    IDB[("IndexedDB<br/>crm_fsa_v1 — file handle")]
    APP -->|"save() on every change"| LS
    APP -->|"optional mirror"| FILE["Local .json file<br/>(File System Access API)"]
    APP -->|"persist handle"| IDB
  end
  USER(["Studio owner"]) --> APP
  APP -.->|"user clicks 'greeting'"| MAIL["mailto: / Gmail compose"]
  APP -.->|"import / export"| CSV["CSV & Excel files"]
```

Key properties:

- **No network dependency.** Fonts degrade gracefully to system fonts; the logo and favicon
  are base64-inlined; there are no script or style CDNs required for the app to function.
- **Single source of truth.** All state is one in-memory object, `db`, serialized to JSON.

---

## 2. Data model

All data lives in one object. `config` holds the editable "shape" of the business (price
list, suppliers, statuses, automations, branding); the rest are records.

```mermaid
erDiagram
  DB ||--|| CONFIG : has
  DB ||--o{ CLIENTS : has
  DB ||--o{ EVENTS : has
  DB ||--o{ PAYMENTS : has
  DB ||--o{ EXPENSES : has
  DB ||--o{ TASKS : has
  DB ||--o{ DEALS : has
  DB ||--o{ PROJECTS : has
  DB ||--o{ SUPPLIERLEDGER : has

  CONFIG {
    array services "each sub has price cost supplier isProduct variable"
    array manufacturers "suppliers"
    array paymentMethods
    array clientStatuses
    array groups
    array automations
    object business "name descriptor mark currency"
  }
  CLIENTS {
    string id
    string name
    string phone
    string statusId
    string groupId
    string birthday
  }
  EVENTS {
    string id
    string type "appt or event"
    string clientId
    array items "serviceId subId price cost manId"
    string date
    string time
    bool done
  }
  PAYMENTS {
    string id
    string clientId
    number amount
    string methodId
    string eventId "optional link to a specific appointment"
  }
  SUPPLIERLEDGER {
    string id
    string manufacturerId
    string type "owe or pay"
    number amount
    string sourceEvent "provenance key for auto-regeneration"
  }
```

The crucial modelling choice: **cost and supplier are properties of the sub-service** (the
product line in the price list), and are copied onto an appointment's `items[]` at sale
time. Payments carry no cost/supplier fields at all — a payment is purely money in.

---

## 3. Appointment → supplier ledger flow

When an appointment is saved, its supplier charges are **rebuilt deterministically** from
its items. This delete-then-rebuild keyed by `sourceEvent` guarantees the ledger always
matches what was actually sold, even after edits.

```mermaid
sequenceDiagram
  participant U as Owner
  participant A as saveEvent()
  participant S as syncEventSuppliers()
  participant L as supplierLedger
  participant P as save()

  U->>A: Save appointment (items with price+cost+supplier)
  A->>S: syncEventSuppliers(eventId, items, date, clientId)
  S->>L: remove all rows where sourceEvent === eventId
  S->>S: sum cost per supplier for items with cost>0
  S->>L: push one 'owe' row per supplier (keyed by sourceEvent)
  A->>P: save()
  P->>P: localStorage.setItem(...)
  P-->>P: if file handle connected → debounced file write
```

Supplier balance is then simply:

```
balance(supplier) = Σ owe(supplier) − Σ pay(supplier)
```

A negative balance is a credit (prepaid stock), which the model supports.

---

## 4. Money semantics

Four independent quantities, deliberately kept separate so partial payments never distort
anything:

```mermaid
graph LR
  SALE["Appointment items<br/>(charge + cost)"] -->|charge| REV["Revenue owed by customer"]
  SALE -->|cost| COGS["Cost of goods"]
  PAY["Payments"] -->|amount| CASH["Cash received"]
  EXP["Expenses"] -->|amount| OUT["Other money out"]
  REV --> PROFIT["Customer profit = charge − cost"]
  COGS --> PROFIT
  CASH --> NET["Net profit (period) = income − supplier cost − expenses"]
  COGS --> NET
  OUT --> NET
```

- **Customer profit** uses *charge* minus cost, not *paid* minus cost — so a deposit today
  and the balance next month does not misstate profit.
- **Debt** for an appointment = `evTotal(appointment) − paymentsTiedToIt`.

---

## 5. Rendering & routing

No framework. A tiny router swaps the visible screen and calls that screen's render
function, which returns an HTML string built with template literals.

```mermaid
graph TD
  NAV["Sidebar button / go(view)"] --> ROUTER["Router: set view, update title"]
  ROUTER --> R{"render(view)"}
  R --> D["renderDashboard()"]
  R --> C["renderCalendar()"]
  R --> CL["renderClients()"]
  R --> F["renderFinances() → 4 tabs"]
  R --> S["renderSettings()"]
  D --> HTML["innerHTML = template string"]
```

Rendering is idempotent: every screen is a pure function of `db`, so any mutation followed by
a re-render reflects the new state. Modals (forms) are shown with a `showModal(html, size)`
helper and never close on an outside click, to prevent accidental data loss.

---

## 6. Persistence

```mermaid
graph TD
  MUT["Any mutation"] --> SAVE["save()"]
  SAVE --> LS["localStorage (always)"]
  SAVE --> H{"file handle connected?"}
  H -->|yes| Q["queueFileWrite() — debounced"]
  Q --> W["write JSON to local file"]
  H -->|no| DONE["done"]
  BOOT["Startup: load()"] --> MIG["migrate() — backfill new config keys"]
  IDB[("IndexedDB crm_fsa_v1")] -->|restore handle| SAVE
```

- `storageMode` is `'local'` or `'file'`.
- `migrate()` makes older saved documents forward-compatible by backfilling any missing
  `config` sections and defaulting new fields (e.g. `cost`, `manufacturerId`).
- JSON backup/restore and CSV/Excel import-export provide additional safety and portability.
