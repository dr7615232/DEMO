# Architecture — תוכי פרינט · בונה הקטלוג

The demo is a single self-contained HTML/CSS/JS file with three screens driven by one
in-memory product list. Selecting a product in the builder flows straight into the
catalog cover and the order form.

## Screen flow

```mermaid
flowchart LR
  build["בונה הקטלוג\n(builder: pick products, set prices)"]
  cover["שער הקטלוג\n(festive cover + catalog pages)"]
  order["טופס הזמנה\n(client order form)"]

  build -->|"tick 'הוסף לקטלוג'"| cover
  build -->|"same selection"| order
  cover -->|"back"| build
  order -->|"back"| build

  subgraph state["one in-memory list"]
    p["products[] · include flag · price · priceQty"]
  end
  build --- state
  cover --- state
  order --- state
```

## Product data model (demo)

```mermaid
classDiagram
  class Product {
    id : string
    t  : title (inline-editable)
    c  : SKU
    cat: category
    img: image (data/URL)
    price    : per-unit price
    priceQty : bulk price (25+)
    include  : in this catalog?
    deleted  : soft-deleted?
  }
  class Contact {
    phone : string (example)
    email : string (example)
  }
  Product "many" --o "1" Catalog : grouped by cat
  Contact "1" --o "1" Catalog : shown on cover + order
```

## Key rules

- **Selection is the single source of truth.** `include` on each product drives what
  appears on the cover, the catalog pages and the order form — no duplicated lists.
- **Grouping & order.** Products are bucketed by `cat`; categories render in a fixed
  configured order, with any extra categories appended.
- **Bulk pricing.** A threshold (`TH = 25`) chooses the price: quantity ≥ 25 uses the
  bulk price when set, otherwise the per-unit price.
- **Dates.** The cover shows both the Gregorian month/year and the Hebrew-calendar date
  via `Intl.DateTimeFormat('he-u-ca-hebrew', …)`.
- **Autosave UX.** Every edit flips a small "שומר… → נשמר" indicator (the demo keeps
  state in memory; production would persist).
- **Client-side only.** No network calls; the demo runs offline by double-click.
