# Zramim — Technical Deep-Dives

Four topics that show engineering depth beyond CRUD. Each is self-contained; pick whichever
fits the audience. All claims trace to real files in the codebase (cited inline).

---

## 1. Atomic subscription registration

**Problem.** Buying a subscription is not one write — it's three: create the subscription,
book it into the next *N* concrete class occurrences, and post a charge to the ledger. If any
of these fails independently you get corruption the original Airtable system actually suffered
from: a subscription with no classes, classes with no charge, a charge with no subscription.

**Solution.** A single PL/pgSQL function, `register_subscription`
(`supabase/migrations/0013_register_subscription.sql`), does all three inside one
transaction. `SECURITY INVOKER` keeps RLS and role permissions in force.

```sql
create or replace function public.register_subscription(
  p_customer_id uuid, p_subscription_type_id uuid, p_group_id uuid,
  p_occurrences timestamptz[], p_override boolean default false,
  p_attendance_kind public.attendance_kind default 'regular'
) returns uuid language plpgsql as $$
declare v_type public.subscription_types%rowtype; v_sub_id uuid; ...
begin
  select * into v_type from public.subscription_types where id = p_subscription_type_id;
  -- price & lesson_count are SNAPSHOTTED onto the subscription, so later catalog
  -- edits never retroactively change a purchase already made.
  insert into public.subscriptions (..., lesson_count, price, started_on, status)
    values (..., v_type.lesson_count, v_type.price, v_started_on, 'active')
    returning id into v_sub_id;
  foreach ts in array p_occurrences loop
    insert into public.class_instances (group_id, starts_at) values (p_group_id, ts)
      on conflict (group_id, starts_at) do update set group_id = excluded.group_id
      returning id into v_instance_id;                 -- idempotent get-or-create
    insert into public.bookings (...) values (...);    -- capacity enforced by trigger
  end loop;
  insert into public.ledger_entries (customer_id, subscription_id, direction, amount, note)
    values (p_customer_id, v_sub_id, 'charge', v_type.price, 'חיוב מנוי: ' || v_type.name);
  return v_sub_id;
end $$;
```

**Design choices worth explaining:**

- **The when is computed in TypeScript, the write happens in SQL.** Generating "the next 4
  Tuesdays at 18:00, skipping holidays, correct across a DST change" is calendar logic, so it
  lives in `lib/domain/schedule.ts` using `Intl.DateTimeFormat` pinned to `Asia/Jerusalem`
  (no date library). The resulting `timestamptz[]` is passed to SQL, which owns only the
  atomic write and capacity enforcement. Clean separation: business calendar in app code,
  integrity in the database.
- **Capacity is a trigger, not an `if`.** `enforce_group_capacity()`
  (`0007_bookings.sql`) runs `BEFORE INSERT/UPDATE` on `bookings` and raises a
  `check_violation` with a Hebrew message when a group is full — unless `is_override` is set
  (manager exception). No screen can forget the rule.
- **Idempotent occurrences.** `class_instances` has a unique index on `(group_id, starts_at)`,
  so the `ON CONFLICT ... RETURNING` makes "create the class if it doesn't exist yet" safe
  under concurrency and re-entry.
- **Snapshotting.** `price` and `lesson_count` are copied onto the subscription at purchase.
  Editing the catalog later never rewrites history.

A sibling function `register_subscription_multi` (`0020`) applies the same pattern to
bi-weekly subscriptions that span two groups.

---

## 2. A ledger that always reconciles (cash + deferred revenue)

**Problem.** The original system mis-allocated payments and invented credit balances. And the
owner couldn't answer the accountant's question: of the cash in the bank, how much have we
*earned* (lessons delivered) versus *owe as future service* (prepaid)?

**Solution — two layers.**

**Layer 1: a customer-level cash ledger.** `ledger_entries.direction` is one of
`charge | payment | credit | refund`. Charges attach to a subscription; **payments are
recorded at the customer level only** (no `subscription_id`). Because payment isn't linked to a
specific subscription, it *can't* be mis-allocated to one. Balance is a pure function
(`lib/domain/ledger.ts`):

```ts
const net = paid - refunded;                  // paid = payments + credits
return {
  owed:   Math.max(0, charged - net),          // open debt
  credit: Math.max(0, net - charged),          // overpayment → reusable credit
};
```

Overpay by ₪100 and it becomes credit automatically — the exact bug (phantom balances) the
rebuild was meant to kill, turned into a defined behavior.

**Layer 2: accrual / deferred-revenue accounting.** A Postgres snapshot function
`finance_deferred_snapshot()` (`0044`) returns, per customer, the building blocks:
`net_cash` (P), `charged` (C), `recognized_to_date` (Rp = value of lessons already passed),
`future_active` (Rf = value of future booked lessons), `frozen_value` (FZ = frozen balance ×
per-lesson price). A separate function `recognized_revenue_by_month()` (`0037`) recognizes
`price / lesson_count` per delivered lesson, bucketed by Jerusalem month.

The **allocation waterfall runs in TypeScript** (`lib/domain/deferred.ts`) so it's readable
and unit-tested in one place — every shekel of cash is assigned in priority order:

```ts
let remaining = nz(c.netCash);
const earned         = Math.min(remaining, nz(c.recognizedToDate)); remaining -= earned;
const deferredActive = Math.min(remaining, nz(c.futureActive));     remaining -= deferredActive;
const frozenHeld     = Math.min(remaining, nz(c.frozenValue));      remaining -= frozenHeld;
const credit         = remaining;                 // surplus beyond all obligations
const debt           = Math.max(0, c.charged - c.netCash);
```

**Why it's a strong story:** the buckets always sum to cash received, so the system reconciles
against the bank statement, and the owner gets an earned-vs-prepaid view per month.
Accrual/deferred revenue is real accounting, not a toy — and it was driven by a concrete
client need, not gold-plating.

---

## 3. Making a stateless phone protocol behave like an app

**Problem.** The IVR runs on Yemot HaMashiach, whose API extension is **stateless**: on every
keypress Yemot issues a fresh HTTP request and **re-sends all previously captured variables**.
There is no session. Worse, a phone menu needs *back-navigation* ("press 0 for the previous
menu, * for main") — but you can't "un-press" a key that Yemot is now echoing back to you
forever.

**Solution — a server-side navigation stack keyed by call-ID.**
`handleIvrWithNav` (`lib/ivr/menu.ts`) wraps the pure menu function `handleIvr(params)`:

1. Load the per-call stack for `ApiCallId` (table `ivr_nav` in prod, an in-process `Map` in
   dev/simulator — `lib/ivr/data.ts`).
2. Read which variable the last screen asked for (regex over the previous `read=` command).
3. Apply the new keypress: `*` clears the stack (→ main), `0` pops (→ previous), a digit
   pushes `{var, value}`.
4. Rebuild the full parameter object *from the stack* and call the pure handler.

So a stateless protocol gets real, correct back-navigation, and the handler itself stays a
pure function of its inputs (easy to test and simulate).

**Two more details that make it robust:**

- **The marker mechanism.** Some steps hand the caller to a *native* Yemot extension —
  credit-card clearing, voicemail, the non-skippable terms/health playback. When the app emits
  a `go_to_folder=` it stamps a sentinel (`PAY_MARKER`, `VM_MARKER`, `TERMS_MARKER`…). When
  Yemot brings the caller back to the API extension, the marker tells the app "this is a
  *return*, finalize it" (record the payment in the ledger, save the voicemail, capture the
  terms confirmation) instead of re-running the routing and looping forever.
- **Response builders + protocol hygiene.** `lib/ivr/response.ts` builds Yemot's command
  language (`&`-joined commands, `.`-joined message segments) with typed helpers
  (`say`, `menu`, `readDigits`, `recordWithConfirm`, `goToFolder`). A `sanitize()` strips the
  characters that would break the protocol (`. - " ' & = | ,`) from all free text — but
  deliberately **preserves Hebrew nikud**, because vowel points improve TTS pronunciation.

**Hebrew text-to-speech.** Static menus are hand-vocalized with nikud in
`lib/ivr/static-menus.json`. Dynamic content is vocalized at runtime: hand-written nikud
lookup tables for weekdays/hours/months/amounts (so "בְּשָׁעָה אַרְבַּע אַחַר הַצָּהֳרַיִם" is
spoken correctly), plus a best-effort call to **Dicta's Nakdan API** (`lib/ivr/nikud.ts`) for
free text and names. The Nakdan result is validated against a "consonant skeleton" of the
source (strip nikud + matres lectionis and compare) and only used if it truly matches and
contains vowels — otherwise the original text is returned unchanged, so vowelization can never
*corrupt* a name. Results are LRU-cached since names recur.

**Security.** The endpoint authenticates a shared secret Yemot sends on every request, compared
with `timingSafeEqual` (`lib/ivr/security.ts`). Card data never touches the app — PCI is handled
entirely by Yemot's native `credit_card` extension wired to Nedarim Plus; a per-caller extension
is provisioned on demand with the exact amount so concurrent callers don't collide.

---

## 4. Defense-in-depth authorization & fail-safe operations

**Two layers of authorization for the same invariants.**

- **App layer** (`lib/auth/require.ts`): `requireProfile()` / `requireManager()` /
  `requireDeveloper()` guard server components, redirecting to `/login`, `/blocked`, or
  `/dashboard`. Wrapped in React `cache()` so a layout and its page share one profile fetch
  per request.
- **Database layer** (RLS in `supabase/schema.sql`): two `SECURITY DEFINER` helpers
  `is_manager()` and `is_staff()` (both require `is_active`) back ~48 policies. Configuration
  tables are *read for staff, write for manager*; operational tables use one `is_staff()`
  policy; **`expenses` and business errors are manager-only**. A secretary who somehow reached
  a finance query still gets **zero rows** from the database.

The developer identity is a *separate axis* (`lib/auth/developer.ts`) — matched by email
against an `app_developers` table (with a `DEVELOPER_EMAILS` env fallback), deliberately kept
out of `profiles` so developers don't appear in the staff-management screen. The developer-only
logs page is guarded even against direct URL guessing.

**API routes are secured differently and correctly.** `/api/*` is excluded from the session
middleware (a phone switch and a cron runner don't have user cookies) and instead authenticates
its own shared secret — `IVR_SECRET`, `CRON_SECRET` — always via `timingSafeEqual`, always in a
header (never a URL param, which would leak into proxy/CDN logs).

**Two-level observability.** Level 1 = *business errors* for the manager (`business_errors`,
manager-only RLS) — a friendly Hebrew message + suggested fix. Level 2 = *technical logs* for
the developer (`app_logs` via `lib/dev-log.ts`) — best-effort, never throws, hidden from all
staff including the manager, with deep-links out to Vercel/Supabase logs. Raw English server
errors are never shown to users: `lib/errors.ts` and `lib/email/explain.ts` translate
Postgres/SMTP failures to Hebrew.

**Fail-safe throughout.** The `noop` email provider returns `ok:false` so nothing is ever
falsely marked "sent ✓". Config loaders `try/catch` down to env/code defaults, so a missing
table or an un-run migration degrades gracefully. Background work (emails, waitlist
notifications, name delivery to the secretary) runs via Next's `after()` so a phone caller is
never blocked on it. The hourly scheduler is idempotent: a `last_run_on` guard dedups, a `>=`
hour comparison lets a missed tick catch up later the same day, and each dispatched job
re-verifies the cron secret independently.
