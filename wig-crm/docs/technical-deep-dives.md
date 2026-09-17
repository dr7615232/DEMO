# Technical Deep-Dives — Wig CRM (אומנות בפאות)

Four engineering deep-dives for a technical reader. Every excerpt is real code from the
delivered single-file application (lightly trimmed for length; comments translated).

---

## 1. A self-reconciling supplier ledger keyed by provenance

**The problem.** A wig is bought from a supplier at a real cost and sold to a customer at a
price. The owner needs to always know how much she still owes each supplier — but she should
never have to maintain a separate supplier ledger by hand, and edits to a sale must never
leave the books inconsistent.

**The idea.** The supplier ledger is *derived* state. When an appointment is saved, its
supplier charges are deleted and rebuilt from its items, tagged with the appointment's id as
a provenance key (`sourceEvent`). Re-saving is therefore idempotent: the ledger can never
double-count or drift.

```js
function syncEventSuppliers(eventId, items, date, clientId){
  // Drop any prior rows this appointment produced …
  db.supplierLedger = db.supplierLedger.filter(e => e.sourceEvent !== eventId);

  // … then re-derive from the current items, summed per supplier.
  const byMan = {};
  (items || []).forEach(it => {
    const cost = Number(it.cost) || 0;
    const man  = it.manId || "";
    if (cost > 0 && man) byMan[man] = (byMan[man] || 0) + cost;
  });

  Object.keys(byMan).forEach(man => {
    db.supplierLedger.push({
      id: uid(), createdAt: Date.now(), type: "owe",
      manufacturerId: man, amount: byMan[man],
      date: date || todayISO(), clientId: clientId || "",
      note: "עלות מכירה בתור", sourceEvent: eventId,   // provenance key
    });
  });
}
```

The balance is then a pure fold over the ledger, with payments to the supplier subtracted:

```js
function supplierBalance(manId){ return supplierOwed(manId) - supplierPaid(manId); }
```

Because `owe` rows come only from `syncEventSuppliers` and `pay` rows come only from
recorded supplier payments, `owe − pay` is always the truth. A negative result is a credit
(the owner pre-bought stock), which the UI surfaces rather than hides.

**Why it matters.** This turns a bookkeeping chore into a side effect of doing normal work.
The owner records a sale the way she already thinks about it; the supplier account maintains
itself.

---

## 2. Cost lives on the sub-service, so profit survives partial payments

**The problem.** Wigs are often paid in installments. If profit were computed as *paid minus
cost*, a customer who leaves a deposit would make the month look unprofitable until she pays
the balance.

**The idea.** Separate the four quantities completely. Cost and supplier are attached to the
**sub-service** (the product in the price list), copied onto the appointment's items at sale
time; a **payment** is only money in. Profit for a customer is *charge minus cost*; cash-flow
is tracked separately by payments.

```js
// Cost/supplier are resolved from the sub-service definition, never from a payment:
function subCost(sid, subId){ const s = subById(sid, subId); return s ? (Number(s.cost) || 0) : 0; }
function subManufacturer(sid, subId){ const s = subById(sid, subId); return s ? (s.manufacturerId || "") : ""; }

// At sale time the item snapshots price + cost + supplier onto the appointment:
const items = _evPick.map(it => ({
  serviceId: it.serviceId, subId: it.subId,
  price: Number(it.price) || 0,
  cost:  Number(it.cost)  || 0,
  manId: subManufacturer(it.serviceId, it.subId),
}));
```

Debt on an appointment is charge minus the payments explicitly tied to it:

```js
function visitPaid(evId){
  return db.payments.filter(p => p.eventId === evId)
                    .reduce((s, p) => s + (Number(p.amount) || 0), 0);
}
```

**A deliberately-tracked nuance.** The project notes flag that period *income* is counted on
the **payment** date while cost of goods is recorded on the **sale** date, so selling in one
month and collecting the next can skew a single month's picture. It is documented as an open
decision rather than silently "fixed", because the right answer (cash vs. accrual) is a
business call, not a bug.

---

## 3. An idempotent, offline reminder engine

**The problem.** The owner wants the system to remind her — wash a wig the day before a
styling appointment, greet a customer before her birthday — but there is no server and no
cron. The engine runs in the browser whenever the app is open, which means it may run many
times a day and must never create the same reminder twice.

**The idea.** Each potential reminder has a deterministic key. A per-document `autoLog`
records which keys have fired; `add()` is a no-op if the key already exists.

```js
function runAutomations(){
  if(!db.config.automations || !db.config.automations.length) return 0;
  db.autoLog = db.autoLog || {};
  const today = todayISO(); let created = 0;

  const add = (rule, key, due, ctx, clientId) => {
    if (db.autoLog[key]) return;                       // idempotency guard
    const title = fillTemplate(rule.titleTemplate, ctx);
    const kind  = rule.action === "reminder" ? "reminder" : "task";
    db.tasks.push({
      id: uid(), title, dueDate: due,
      priorityId: rule.priorityId || autoDefaultPriority(),
      done:false, createdAt:Date.now(), auto:true, kind, ruleKey:key,
    });
    db.autoLog[key] = true; created++;
  };

  db.config.automations.filter(r => r.enabled).forEach(r => {
    if (r.trigger === "before_appt" || r.trigger === "after_appt") {
      db.events.filter(e => e.type === "appt" && (!r.serviceId || evHasService(e, r.serviceId)))
        .forEach(e => {
          // window-guard so we only look at the relevant horizon …
          if (r.trigger === "before_appt" && (e.date < today || e.date > addDays(today,120))) return;
          const due = addDays(e.date, -(r.offsetDays || 1));
          add(r, /* deterministic key from rule + event */ r.id + ":" + e.id, due, {/*ctx*/}, e.clientId);
        });
    }
    // birthday trigger handled similarly, keyed by rule + client + year
  });
  return created;
}
```

Templates are filled with a tiny, safe substitution — only three named tokens are ever
interpolated, so there is no risk of arbitrary expansion:

```js
function fillTemplate(t, ctx){
  return String(t || "").replace(/\{(לקוחה|שירות|תאריך)\}/g,
    (m, k) => (ctx[k] != null && ctx[k] !== "") ? ctx[k] : m);
}
```

**Why it matters.** It gives a cron-like feature to a program that has no cron and no server,
with correctness (exactly-once) guaranteed by construction rather than by timing.

---

## 4. Real local-file persistence, safely, in a static file

**The problem.** `localStorage` is convenient but invisible and easy to lose. A non-technical
owner wants her data to live in a real file she can see and back up — without a server and
without re-picking the file every session.

**The idea.** Always write to `localStorage`; additionally, when the user opts in, mirror
every change to a chosen file via the File System Access API, and persist the returned file
**handle** in IndexedDB so the connection survives restarts.

```js
function save(){
  try { localStorage.setItem(KEY, JSON.stringify(db)); }
  catch(e){ if (storageMode === "local") toast("שמירה נכשלה – כדאי לגבות", true); }
  if (fileHandle) queueFileWrite();     // debounced mirror to the real file
  updateStorageIndicator();
}

let fileHandle = null, storageMode = "local";
function fsaOK(){ return !!(window.showSaveFilePicker && window.showOpenFilePicker); }

// The file handle itself is stored in IndexedDB so it reconnects on next launch:
const IDB_NAME = "crm_fsa_v1", IDB_STORE = "handles";
function idbReady(){
  return new Promise((res, rej) => {
    const r = indexedDB.open(IDB_NAME, 1);
    r.onupgradeneeded = () => { if(!r.result.objectStoreNames.contains(IDB_STORE))
                                  r.result.createObjectStore(IDB_STORE); };
    r.onsuccess = () => res(r.result);
    r.onerror   = () => rej(r.error);
  });
}
```

Two defensive touches:

- **Writes are debounced** (`queueFileWrite`) so a burst of edits produces one disk write,
  not dozens.
- **Forward-compatible loading.** On startup, `migrate()` backfills any `config` sections and
  fields added since a document was last saved (for example defaulting `cost` and
  `manufacturerId` on older sub-services), so an old backup never crashes a newer build.

**Why it matters.** It gives a static, serverless HTML file the durability story of a real
app — the owner's data is a plain JSON file she owns, and the app quietly keeps it in sync.
