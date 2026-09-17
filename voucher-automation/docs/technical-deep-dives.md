# Technical Deep-Dives — שובר אוטומטי

> **Reference implementation.** The code below is a representative, publish-safe
> implementation of the described workflow (PostgreSQL + Node.js shown for concreteness).
> It contains no secrets and no real data. It illustrates *how* each guarantee is achieved,
> and would need wiring to a real donation source and email provider to run.

---

## Deep-dive 1 — Unique voucher numbers that can never collide

**The manual pain it replaces:** the operator typed each voucher number by hand, and the
whole system's integrity depended on never reusing one. Under load, that is a matter of time
before a mistake.

**The principle:** uniqueness must be enforced by the **database**, not by careful
application code. Two guards work together.

1. A schema-level unique constraint makes a duplicate number physically impossible to store.
2. Numbers are drawn from a **sequence**, so concurrent requests never see the same value.

```sql
-- Numbers come from a gapless-enough, monotonic sequence.
CREATE SEQUENCE voucher_number_seq START 100001;

CREATE TABLE voucher (
  id           uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  number       bigint NOT NULL DEFAULT nextval('voucher_number_seq'),
  donation_id  uuid   NOT NULL REFERENCES donation(id),
  location_id  uuid   NOT NULL REFERENCES location(id),
  value_agorot int    NOT NULL CHECK (value_agorot > 0),
  status       text   NOT NULL DEFAULT 'issuing',
  issued_at    timestamptz NOT NULL DEFAULT now(),
  valid_until  timestamptz NOT NULL,
  -- the guarantee: the DB itself refuses a duplicate number
  CONSTRAINT voucher_number_unique UNIQUE (number)
);
```

Because `number` defaults from `nextval(...)`, the application never chooses a number at all
— it just inserts a row and reads back what the database assigned. Even if two donations are
processed at the exact same instant, the sequence hands each a distinct value. The unique
constraint is the belt to the sequence's suspenders: if anything ever tried to force a
duplicate, the insert fails loudly instead of corrupting the data.

**Why not `MAX(number) + 1`?** That classic pattern has a race: two concurrent requests read
the same max and both write `max + 1`. A sequence (or an atomic counter) removes the race by
construction.

---

## Deep-dive 2 — Issuing a voucher as one atomic step

**The manual pain it replaces:** by hand, "assign a number", "produce the voucher", and
"remember it went out" were separate acts, any of which could be half-done. An automation
that runs unattended cannot leave half-done state behind.

**The principle:** reserving the number, decrementing stock, and creating the voucher happen
inside **one transaction**. Either all of it commits, or none of it does.

```js
// issueVoucher.js — runs when the donor submits their chosen location.
async function issueVoucher(pool, { donationId, locationId }) {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');

    // 1) Guarded stock decrement: only succeeds if stock is actually available.
    //    The WHERE clause is the guard — it cannot go negative.
    const stock = await client.query(
      `UPDATE stock
          SET remaining = remaining - 1
        WHERE location_id = $1 AND remaining > 0
      RETURNING remaining, alert_threshold`,
      [locationId]
    );
    if (stock.rowCount === 0) {
      // Location is sold out. Abort cleanly — no number burned, no voucher created.
      await client.query('ROLLBACK');
      return { ok: false, reason: 'out_of_stock' };
    }

    // 2) Create the voucher. The number is assigned by the sequence (deep-dive 1).
    const voucher = await client.query(
      `INSERT INTO voucher (donation_id, location_id, value_agorot, valid_until)
       SELECT id, $2, amount_agorot, now() + interval '6 months'
         FROM donation WHERE id = $1
       RETURNING id, number, value_agorot`,
      [donationId, locationId]
    );

    // 3) Advance the donation's own status in the same breath.
    await client.query(
      `UPDATE donation SET status = 'issued' WHERE id = $1`,
      [donationId]
    );

    await client.query('COMMIT');

    const { remaining, alert_threshold } = stock.rows[0];
    return {
      ok: true,
      voucher: voucher.rows[0],
      lowStock: remaining <= alert_threshold, // signal for the alerting layer
    };
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
}
```

Two things make this safe to run with nobody watching:

- **No partial state.** A crash between steps rolls the whole thing back. You never get a
  consumed stock unit with no voucher, or a voucher with no stock decrement.
- **No overselling.** `remaining > 0` in the `WHERE` clause means the decrement is atomic
  with the availability check. Two simultaneous last-voucher requests cannot both win.

Rendering and emailing happen **after** the commit, so a slow email provider never holds a
database transaction open (see deep-dive 4).

---

## Deep-dive 3 — Edge-triggered, de-duplicated low-stock alerts

**The manual pain it replaces:** stock was invisible until it was empty. The automation must
warn *ahead of time*, but without burying the operator in a notification per voucher.

**The principle:** an alert opens **once** when stock *crosses* the threshold downward, and
resolves automatically when replenishment lifts it back above. It is a state transition, not
a level check repeated on every issue.

```js
// After a successful issueVoucher() that reported lowStock:
async function reconcileStockAlert(pool, locationId) {
  // Is there already an unresolved alert for this location?
  const open = await pool.query(
    `SELECT 1 FROM stock_alert
      WHERE location_id = $1 AND resolved_at IS NULL`,
    [locationId]
  );

  const stock = await pool.query(
    `SELECT remaining, alert_threshold FROM stock WHERE location_id = $1`,
    [locationId]
  );
  const { remaining, alert_threshold } = stock.rows[0];

  if (remaining <= alert_threshold && open.rowCount === 0) {
    // Crossed downward and no alert open yet → open exactly one.
    await pool.query(
      `INSERT INTO stock_alert (location_id, remaining_at_open)
       VALUES ($1, $2)`,
      [locationId, remaining]
    );
    await notifyOperatorLowStock(locationId, remaining); // email, once
  }

  if (remaining > alert_threshold && open.rowCount > 0) {
    // Replenished above threshold → auto-resolve.
    await pool.query(
      `UPDATE stock_alert
          SET resolved_at = now()
        WHERE location_id = $1 AND resolved_at IS NULL`,
      [locationId]
    );
  }
}
```

The `open.rowCount === 0` check is what turns "stock is low" (true on every subsequent issue)
into "stock *just became* low" (true once). The operator gets a single, timely heads-up per
location, and the dashboard reflects the same open/resolved state.

---

## Deep-dive 4 — Delivering the voucher reliably, outside the transaction

**The manual pain it replaces:** the operator personally attached and sent each voucher. The
automation sends it — but email providers are flaky, and a failed send must not lose the
voucher or block the next donor.

**The principle:** **render and send after commit**, track delivery as its own state, and
**retry** transient failures with backoff. The voucher record is the source of truth; the
email is a delivery attempt against it.

```js
// deliverVoucher.js — invoked after issueVoucher commits.
async function deliverVoucher(pool, mailer, voucherId) {
  const v = await loadVoucherForRender(pool, voucherId); // donor, location, number, value
  const pdf = await renderVoucherPdf(v);                  // designed HTML → PDF

  try {
    await mailer.send({
      to: v.donorEmail,
      subject: `השובר שלך · ${v.locationName}`,
      html: voucherEmailBody(v),
      attachments: [{ filename: `voucher-${v.number}.pdf`, content: pdf }],
    });
    await pool.query(
      `UPDATE voucher SET status = 'sent', sent_at = now() WHERE id = $1`,
      [voucherId]
    );
  } catch (err) {
    // Mark for retry rather than losing it. A job runner picks 'failed' back up.
    await pool.query(
      `UPDATE voucher
          SET status = 'failed', last_error = $2, retry_count = retry_count + 1
        WHERE id = $1`,
      [voucherId, String(err.message).slice(0, 240)]
    );
    if (isTransient(err)) throw err; // let the runner retry with backoff
  }
}
```

Design choices worth noting:

- **Commit first, send second.** The donor's voucher exists and is numbered before any email
  is attempted. A delivery failure is recoverable; a lost voucher record is not.
- **Delivery is a tracked state** (`issuing → sent | failed`), so the dashboard can show
  "in production" versus "sent", and failed sends are visible and retryable rather than
  silently gone.
- **Retries with backoff** handle the normal flakiness of email without human intervention —
  which is the whole point of the automation.
