# Technical Deep-Dives — תוכי פרינט · בונה הקטלוג

Four notes on how the demo works. All snippets are real, taken from
`../demo/index.html` and lightly trimmed.

---

## 1. One selection drives three screens

There is a single working list; the `include` flag on each product is the only thing
that decides what appears on the catalog cover and in the order form. The builder,
cover and order screens all read the same `products` array, so they can never drift
apart.

```js
let products = DATA.products.map(p => ({
  id:p.id, t:p.t, c:p.c, cat:p.cat, img:p.img,
  price:p.price, priceQty:p.priceQty, include:false, deleted:false
}));
const live = list => list.filter(p => !p.deleted);
// cover/order both do: live(products).filter(p => p.include)
```

**Why it matters:** the owner ticks a product once and it is instantly consistent
everywhere — no copy-paste, no page/price mismatch.

---

## 2. Quantity-aware pricing (bulk from 25)

Each product carries a per-unit price and a bulk price. A single pure function picks
the right one by quantity, with safe fallbacks when a price is missing.

```js
const TH = 25; // bulk threshold
function priceFor(p, q){
  const u = parseFloat(p.price) || 0, qq = parseFloat(p.priceQty) || 0;
  if (q >= TH && qq > 0) return qq;   // 25+ → bulk price
  if (u > 0) return u;                // else per-unit
  return qq || u || 0;                // last-resort fallback
}
```

The order form highlights which tier is active for the current quantity, so the
customer sees exactly why a line costs what it does.

**Why it matters:** correct pricing is automatic and transparent; the owner sets two
numbers and never recomputes tiers by hand.

---

## 3. Category grouping and stable ordering

The builder and both output screens group products by category and render categories
in a configured order, appending any stragglers so nothing is ever hidden.

```js
const byCat = {};
list.forEach(p => (byCat[p.cat] = byCat[p.cat] || []).push(p));
const order = CATS.filter(c => byCat[c] && byCat[c].length);
Object.keys(byCat).forEach(c => { if (!order.includes(c)) order.push(c); });
// then render each category section in `order`
```

Search and a category filter narrow what's shown without changing the underlying
selection, so filtering never loses ticked products.

**Why it matters:** a catalog of hundreds of items stays readable and predictable,
and the printed order matches the on-screen order.

---

## 4. Dual Gregorian + Hebrew dates on the cover

The festive cover stamps the current month in both calendars, using the browser's
built-in Hebrew calendar — no date library.

```js
function hebDate(d){
  try {
    return new Intl.DateTimeFormat('he-u-ca-hebrew',
      { day:'numeric', month:'long', year:'numeric' }).format(d);
  } catch(e){ return ''; }
}
const now = new Date();
const g = now.toLocaleDateString('he-IL', { month:'long', year:'numeric' });
coverDate.textContent = g + ' · ' + hebDate(now);   // e.g. ספטמבר 2026 · ...
```

**Why it matters:** the catalog reads naturally to an Israeli audience, and the date
is always current with zero maintenance.
