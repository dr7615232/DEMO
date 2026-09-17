# Code Snippets — Wig CRM (אומנות בפאות)

Curated, real excerpts from the delivered single-file app, chosen to show the design taste
and the core ideas. All are safe to publish: no secrets, no customer data. Comments
translated from Hebrew where helpful.

---

## 1. The design system, as CSS custom properties

The whole rose-gold theme is a handful of tokens. Everything else references them.

```css
:root{
  --bg:#FAF5F2; --surface:#FFFFFF; --ink:#4A3833; --muted:#A18A83;
  --line:#EDE1DB; --line-soft:#F4ECE8;
  --plum:#5E4642; --plum-deep:#48342F;
  --rose:#C88E7F; --rose-soft:#D8A79A; --rose-wash:#F6E8E2;
  --danger:#B5675C; --ok:#7C9A6E;
  --shadow:0 1px 2px rgba(94,70,66,.05),0 8px 24px rgba(94,70,66,.07);
  --r:14px; --r-sm:9px;
}
body{ font-family:"Rubik","Assistant","Heebo",system-ui,sans-serif;
  background:var(--bg); color:var(--ink); }
.app{ display:grid; grid-template-columns:238px 1fr; height:100vh; }  /* sidebar + main */
```

A named colour swatch table drives every status/category badge, so a new status only needs a
colour name, never new CSS:

```js
const SWATCHES = {
  rose:{bg:"#F2C4B7",fg:"#8A3F2E",dot:"#C86A54"}, gold:{bg:"#EFD6A9",fg:"#7E5A16",dot:"#C79A44"},
  mauve:{bg:"#E8C2D3",fg:"#6A3550",dot:"#B0648A"}, sage:{bg:"#C8DCBD",fg:"#47643A",dot:"#7C9A66"},
  clay:{bg:"#EDC6B2",fg:"#8A452A",dot:"#C1764F"}, taupe:{bg:"#D8C6B8",fg:"#6A5140",dot:"#9A7F68"},
  // …
};
function badgeHtml(list, id){
  const o = opt(list, id); if(!o) return "";
  const s = SWATCHES[o.color] || SWATCHES.rose;
  return `<span class="badge" style="background:${s.bg};color:${s.fg}">
            <span class="b-dot" style="background:${s.dot}"></span>${esc(o.label)}</span>`;
}
```

---

## 2. The business rule, as data

The price list *is* the business logic. A sub-service that is a product carries a cost and a
supplier; a plain service does not. `variable` means the price is typed at sale time.

```js
services:[
  { id:"combing", label:"סירוק", color:"rose", subs:[
      { id:"sc1", label:"סירוק רגיל",   price:150, cost:0 },
      { id:"sc2", label:"סירוק + עיצוב", price:220, cost:0 } ]},
  { id:"sale", label:"מכירה", color:"sage", subs:[
      { id:"sl1", label:"פאה חדשה",     price:3600, cost:2000, manufacturerId:"m1" },
      { id:"sl2", label:"פאה יד שנייה", price:1800, cost:900,  manufacturerId:"m1" } ]},
]
```

---

## 3. Events are the source of truth for what was sold

An appointment holds line items; helpers derive labels and totals so the rest of the app
never re-implements the math.

```js
function evItems(e){
  if (e.items && e.items.length) return e.items;
  if (e.serviceId) return [{ serviceId:e.serviceId, subId:null, price:e.price||0 }];
  return [];
}
function evTotal(e){
  const its = evItems(e);
  return its.length ? its.reduce((a,i) => a + (Number(i.price)||0), 0) : (Number(e.price)||0);
}
```

---

## 4. Deriving the whole finance overview from records

Net profit for a period is three folds and a subtraction — income (payments), cost of goods
(supplier `owe` rows in-period), and other expenses:

```js
const income  = sumKeys(db.payments, keys);
const expense = sumKeys(db.expenses, keys);
const cogs    = db.supplierLedger
  .filter(e => e.type === "owe" && kset.has((e.date||"").slice(0,7)))
  .reduce((s,e) => s + (Number(e.amount)||0), 0);
const net = income - cogs - expense;
```

Open customer debts are computed on the fly, never stored:

```js
db.events.filter(e => e.type === "appt").forEach(v => {
  const owed = evTotal(v) - visitPaid(v.id);
  if (owed > 0.5) { /* group by client → debts panel */ }
});
```

---

## 5. Forward-compatible loading

New builds must open old backups. `migrate()` backfills anything a saved document predates.

```js
function migrate(d){
  d.config = d.config || defaultConfig();
  const dc = defaultConfig();
  ["services","paymentMethods","clientStatuses","groups","manufacturers","automations"]
    .forEach(k => { if(!d.config[k]) d.config[k] = dc[k]; });
  ["clients","projects","deals","events","tasks","expenses","payments","supplierLedger"]
    .forEach(k => d[k] = d[k] || []);
  (d.config.services||[]).forEach(s => (s.subs||[]).forEach(sub => {
    if (sub.cost == null) sub.cost = 0;
    if (sub.manufacturerId == null) sub.manufacturerId = "";
  }));
  d.autoLog = d.autoLog || {};
  return d;
}
```

---

## 6. The pre-delivery safety check

Because it is one file with one script, correctness is verified by extracting the script,
syntax-checking it, and simulating the screens headlessly before delivery.

```bash
# Extract the single <script> block and syntax-check it
python3 -c "import re;h=open('index.html',encoding='utf-8').read();\
open('_c.js','w',encoding='utf-8').write(re.findall(r'<script>(.*?)</script>',h,re.S)[0])"
node --check _c.js
# then a jsdom simulation walks every screen and the changed flow; deliver only at 0 errors
```
