# Selected Code Snippets — שובר אוטומטי

> Publish-safe excerpts from the reference implementation. No secrets, no real data. Each is
> a small, self-contained illustration of one idea. See
> [`technical-deep-dives.md`](./technical-deep-dives.md) for the full context.

---

## 1. Money as integers (agorot), formatted for display

Currency is stored and computed as integers; formatting happens only at the edge.

```js
// money.js — never store or math currency as a float.
const toAgorot = (shekels) => Math.round(shekels * 100);

const formatILS = (agorot) =>
  new Intl.NumberFormat('he-IL', {
    style: 'currency',
    currency: 'ILS',
    maximumFractionDigits: 0,
  }).format(agorot / 100);

formatILS(50000); // "‏₪ 500"
```

---

## 2. The stock guard, in one line of SQL

The entire "don't oversell" guarantee lives in the `WHERE` clause.

```sql
UPDATE stock
   SET remaining = remaining - 1
 WHERE location_id = $1
   AND remaining > 0      -- ← the guard: atomic check-and-decrement
RETURNING remaining, alert_threshold;
```

If `rowCount === 0`, the location was already at zero — the update simply did nothing, and
the caller aborts without burning a voucher number.

---

## 3. Voucher validity window

Vouchers are issued with a fixed validity period, computed by the database at insert time.

```sql
INSERT INTO voucher (donation_id, location_id, value_agorot, valid_until)
SELECT id, $2, amount_agorot, now() + interval '6 months'
  FROM donation
 WHERE id = $1
RETURNING id, number, value_agorot, valid_until;
```

---

## 4. Masking donor emails for any operator-facing list

Operators can see enough to recognize a donor, never the full address in a shared view.

```js
// privacy.js
function maskEmail(email) {
  const [user, domain] = String(email).split('@');
  if (!domain) return '•••';
  const head = user.slice(0, Math.min(4, user.length));
  return `${head}•••@${domain}`;
}

maskEmail('yael.azoulay@example.com'); // "yael•••@example.com"
```

---

## 5. Edge-trigger check (open an alert only on the downward crossing)

The predicate that turns "is low" into "just became low".

```js
const justCrossedDown = remaining <= threshold && !alertAlreadyOpen;
const justRecovered   = remaining >  threshold &&  alertAlreadyOpen;
```

---

## 6. Voucher email subject & body (Hebrew, RTL-safe)

```js
const voucherEmailBody = (v) => `
  <div dir="rtl" style="font-family:Heebo,Arial,sans-serif;color:#12312b">
    <h2>תודה על תרומתך!</h2>
    <p>מצורף השובר שלך למימוש ב<strong>${v.locationName}</strong>.</p>
    <p>מספר שובר: <strong>${v.number}</strong> · ערך: <strong>${formatILS(v.valueAgorot)}</strong></p>
    <p>בתוקף עד ${new Intl.DateTimeFormat('he-IL').format(v.validUntil)}.</p>
  </div>
`;
```

---

## 7. Idempotent intake (the same donation never creates two records)

Incoming donations carry a source reference; intake upserts on it so a re-delivered
notification is harmless.

```sql
INSERT INTO donation (donor_name, donor_email, amount_agorot, source_ref, status)
VALUES ($1, $2, $3, $4, 'awaiting_choice')
ON CONFLICT (source_ref) DO NOTHING
RETURNING id;
```

If the same `source_ref` arrives twice, the second insert is a no-op — no duplicate donor,
no duplicate voucher down the line.
