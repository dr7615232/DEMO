# Code Snippets — תוכי פרינט · בונה הקטלוג

Selected, publish-safe excerpts from `../demo/index.html`. No secrets; the demo is
client-side only.

---

## 1. Inline-editable product card (builder)

Title is `contenteditable`, prices are number inputs, one toggle adds the product to
the catalog. Everything is wired by `data-*` hooks.

```js
function card(p){
  const el = document.createElement('div');
  el.className = "pcard" + (p.include ? " on" : "");
  el.innerHTML = `
    <div class="imgwrap">${p.img ? `<img src="${p.img}">` : `<div>אין תמונה</div>`}</div>
    <div class="pbody">
      <div class="ptitle" contenteditable="true" data-t="${p.id}">${esc(p.t)}</div>
      <div class="pmeta">${p.c ? `<span class="sku">${esc(p.c)}</span>` : ''}</div>
      <div class="prices">
        <div class="pricewrap"><span class="tag">ליח׳</span>
          <input type="number" value="${p.price}" data-p="${p.id}"></div>
        <div class="pricewrap"><span class="tag q">25+</span>
          <input type="number" value="${p.priceQty}" data-pq="${p.id}"></div>
      </div>
      <div class="incl" data-i="${p.id}">
        <span class="box">${p.include ? '✓' : ''}</span>
        ${p.include ? 'נבחר לקטלוג' : 'הוסף לקטלוג'}
      </div>
    </div>`;
  return el;
}
```

## 2. Event delegation for the whole grid

One listener handles include-toggle, delete, select-all/none and collapse — no
per-card handlers.

```js
main.addEventListener('click', e => {
  const t = e.target.closest('[data-i],[data-del],[data-all],[data-none],[data-toggle]');
  if (!t) return;
  if (t.dataset.i != null){ const p = find(t.dataset.i); p.include = !p.include; save(); render(); }
  else if (t.dataset.del != null){ if (confirm('למחוק את המוצר מהרשימה?')){ find(t.dataset.del).deleted = true; save(); render(); } }
  else if (t.dataset.all != null){ live(products).filter(p => p.cat === t.dataset.all).forEach(p => p.include = true); save(); render(); }
  else if (t.dataset.none != null){ live(products).filter(p => p.cat === t.dataset.none).forEach(p => p.include = false); save(); render(); }
  else if (t.dataset.toggle != null){ closed[t.dataset.toggle] = !closed[t.dataset.toggle]; render(); }
});
```

## 3. Live selection summary (floating bar)

```js
let totalSel = 0, sumSel = 0;
live(products).forEach(p => { if (p.include){ totalSel++; sumSel += parseFloat(p.price) || 0; } });
fSel.textContent = totalSel;
fSum.textContent = fmt(sumSel) + ' ₪';
document.getElementById('floatbar').classList.toggle('show', totalSel > 0);
```

## 4. Quantity-aware price + Israeli number formatting

```js
const TH = 25;
function priceFor(p, q){
  const u = parseFloat(p.price) || 0, qq = parseFloat(p.priceQty) || 0;
  if (q >= TH && qq > 0) return qq;
  if (u > 0) return u;
  return qq || u || 0;
}
const fmt = n => (Math.round(n*100)/100).toLocaleString('he-IL');
```

## 5. Design tokens (Rubik, brand palette, RTL)

```css
:root{
  --ink:#1d2b36; --ink2:#5a6b78; --line:#e7ecef; --bg:#f4f6f8; --card:#fff;
  --brand:#2a7de1; --brand2:#e8542f; --green:#22a06b; --gold:#f4b400;
  --radius:14px;
}
html[dir="rtl"] body{ font-family:'Rubik','Heebo','Assistant',sans-serif; background:var(--bg); color:var(--ink); }
```
