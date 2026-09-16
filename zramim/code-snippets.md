# Zramim — Curated Code Snippets

Real excerpts from the codebase, chosen to show engineering judgment rather than volume. Each
has a one-line "why it matters." Safe to publish (no secrets, no customer data).

---

### 1. Atomic registration RPC (Postgres)
*Why it matters: multi-row integrity enforced by the database, not hopeful app code.*

```sql
-- supabase/migrations/0013_register_subscription.sql
-- Purchase = subscription + N bookings + a ledger charge, in ONE transaction.
insert into public.subscriptions (customer_id, subscription_type_id, group_id,
  lesson_count, price, started_on, status)
  values (p_customer_id, p_subscription_type_id, p_group_id,
    v_type.lesson_count, v_type.price, v_started_on, 'active')   -- price/count snapshotted
  returning id into v_sub_id;

foreach ts in array p_occurrences loop
  insert into public.class_instances (group_id, starts_at) values (p_group_id, ts)
    on conflict (group_id, starts_at) do update set group_id = excluded.group_id
    returning id into v_instance_id;                             -- idempotent get-or-create
  insert into public.bookings (subscription_id, customer_id, class_instance_id,
    status, attendance_kind, is_override)
    values (v_sub_id, p_customer_id, v_instance_id, 'scheduled', p_attendance_kind, p_override);
end loop;                                                        -- capacity enforced by trigger

insert into public.ledger_entries (customer_id, subscription_id, direction, amount, note)
  values (p_customer_id, v_sub_id, 'charge', v_type.price, 'חיוב מנוי: ' || v_type.name);
```

---

### 2. Pure balance function (TypeScript)
*Why it matters: overpayment becoming reusable credit is a defined behavior, not a bug.*

```ts
// lib/domain/ledger.ts
export function computeBalance(rows: LedgerRow[]): CustomerBalance {
  let charged = 0, paid = 0, refunded = 0;
  for (const r of rows) {
    const amount = Number(r.amount);
    if (r.direction === "charge") charged += amount;
    else if (r.direction === "payment" || r.direction === "credit") paid += amount;
    else if (r.direction === "refund") refunded += amount;
  }
  const net = paid - refunded;
  return {
    charged, paid, refunded,
    owed:   Math.max(0, charged - net),   // open debt
    credit: Math.max(0, net - charged),   // overpayment → reusable credit balance
  };
}
```

---

### 3. Deferred-revenue waterfall (TypeScript)
*Why it matters: every shekel of cash is explained; buckets sum to cash received.*

```ts
// lib/domain/deferred.ts
export function allocateDeferred(c: DeferredComponents): DeferredAllocation {
  const nz = (n: number) => (Number.isFinite(n) && n > 0 ? n : 0);
  let remaining = nz(c.netCash);
  const earned         = Math.min(remaining, nz(c.recognizedToDate)); remaining -= earned;
  const deferredActive = Math.min(remaining, nz(c.futureActive));     remaining -= deferredActive;
  const frozenHeld     = Math.min(remaining, nz(c.frozenValue));      remaining -= frozenHeld;
  const credit         = remaining;                       // surplus beyond all obligations
  const debt           = Math.max(0, c.charged - c.netCash);
  return { earned, deferredActive, frozenHeld, credit, debt,
           futureOwned: deferredActive + frozenHeld + credit };
}
```

---

### 4. IVR response builder + protocol hygiene (TypeScript)
*Why it matters: strips protocol-breaking chars but keeps Hebrew nikud for correct TTS.*

```ts
// lib/ivr/response.ts
const BREAKING = /[.\-"'&=|,]/g;                 // chars that break Yemot's command language
export function sanitize(text: string): string { // nikud is intentionally preserved
  return text.replace(BREAKING, " ").replace(/\s+/g, " ").trim();
}

export function say(...segments: Segment[]): string {
  const parts = segments.map((s) =>
    "t" in s ? `t-${sanitize(s.t)}`              // TTS text
    : "d" in s ? `d-${String(s.d).replace(/\D/g, "")}`  // digits, read one-by-one
    : `f-${s.f}`);                               // pre-uploaded audio file
  return `id_list_message=${parts.join(".")}`;
}
```

---

### 5. Constant-time secret check for machine endpoints (TypeScript)
*Why it matters: cron/IVR endpoints are secured against timing attacks, header-only.*

```ts
// lib/cron.ts — timing-safe compare; secret in a header, never a URL param
export function verifyCron(request: Request): boolean {
  const secret = process.env.CRON_SECRET;
  if (!secret) return false;
  const provided =
    request.headers.get("authorization")?.replace(/^Bearer\s+/i, "") ??
    request.headers.get("x-cron-secret") ?? "";
  const a = Buffer.from(provided), b = Buffer.from(secret);
  return a.length === b.length && timingSafeEqual(a, b);
}
```

---

### 6. Request-memoized, defense-in-depth auth guard (TypeScript)
*Why it matters: one profile fetch per request; blocked users get a friendly page, not a bounce.*

```ts
// lib/auth/require.ts (shape)
export const requireProfile = cache(async () => {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) redirect("/login");
  const { data: profile } = await supabase
    .from("profiles").select("*").eq("id", user.id).single();
  if (!profile || !profile.is_active) redirect("/blocked");
  return profile;
});
// RLS enforces the SAME invariants in the database: is_manager()/is_staff() both require is_active.
```

---

### 7. Server Action with Hebrew-first validation (TypeScript)
*Why it matters: hard duplicate-prevention with an override for real edge cases (namesakes).*

```ts
// app/dashboard/customers/actions.ts
if (!allowDupName) {
  const { data: sameName } = await supabase
    .from("customers").select("id").eq("full_name", fullName).limit(1).maybeSingle();
  if (sameName) return { ok: false,
    error: "קיימת כבר לקוחה בשם זה. אם זו לקוחה אחרת, סמני 'אישור שם כפול' והוסיפי שוב." };
}
// DB unique index on (phone, full_name) is the real backstop; 23505 → a friendly Hebrew message.
```

---

### 8. XSS-safe report tables for email (TypeScript)
*Why it matters: customer names are escaped before landing in an HTML email.*

```ts
// lib/domain/reports.ts
const esc = (v: string) => String(v ?? "")
  .replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");

export function htmlTable(headers: string[], rows: string[][]): string {
  const head = headers.map((h) => `<th style="text-align:right">${esc(h)}</th>`).join("");
  const body = rows.map((r) => `<tr>${r.map((c) => `<td>${esc(c)}</td>`).join("")}</tr>`).join("");
  return `<table dir="rtl">...${head}...${body}...</table>`;
}
```

---

### 9. Pluggable email adapter with a safe default (TypeScript)
*Why it matters: `noop` returns `ok:false` — the system never falsely reports "sent".*

```ts
// lib/email/providers/noop.ts (behavior)
export async function send(): Promise<EmailResult> {
  console.log("[email:noop] not configured — logging instead of sending");
  return { ok: false, error: "ספק מייל לא הוגדר" };   // never a false success
}
// lib/email/index.ts selects resend | smtp/gmail | noop at runtime via dynamic import().
```

---

### 10. Water-themed shared UI (TSX)
*Why it matters: a small, consistent design system — brand palette, RTL, focus rings.*

```tsx
// components/ui.tsx — "liquid" primary button, brand gradient, motion on hover
export function Button({ variant = "primary", className = "", ...props }) {
  const base = "rounded-full px-5 py-2 text-sm font-semibold transition " +
               "disabled:opacity-50 hover:-translate-y-0.5";
  const styles = variant === "primary"
    ? "btn-liquid bg-gradient-to-br from-[#ef5d80] to-[#d43e64] text-white"
    : "border border-gray-200 bg-white text-brand hover:border-brand-soft";
  return <button className={`${base} ${styles} ${className}`} {...props} />;
}
```
