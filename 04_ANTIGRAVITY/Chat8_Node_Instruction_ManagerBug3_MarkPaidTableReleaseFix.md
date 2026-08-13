# TableFlow — Chat #8 — Instruction (Fix Only)

**To:** Antigravity
**From:** Claude
**Type:** Fix — Bug 3 (Mark Paid table release)

---

## Context

Per investigation (`Chat8_Node_Investigation_ManagerBug3_MarkPaidTableRelease.md`): `markPaid()` in `app/dashboard/manager/page.tsx` only updates `orders.status`, never touches `restaurant_tables`. Root cause confirmed. Fix must replicate the cancellation flow's proven pattern: decrement `occupied_seats` by the paid order's `party_size`, table flips to `'available'` only when `occupied_seats <= 0` (correctly handles multiple orders per table).

## Fix

**1. Create a new RPC `mark_order_paid`** (new migration file, e.g. `supabase/migrations/20260809000001_mark_order_paid_rpc.sql`), modeled directly on the existing `cancel_active_orders` RPC's table-release logic:

- Input: order id (and payment method)
- Atomically, in one transaction:
  - Update `orders`: `status = 'billed'`, `payment_method = <input>`, `updated_at = now()`
  - Update `restaurant_tables` for that order's `table_id`:
    ```sql
    UPDATE restaurant_tables
    SET occupied_seats = GREATEST(0, occupied_seats - v_order_record.party_size),
        status = CASE WHEN (occupied_seats - v_order_record.party_size) <= 0 THEN 'available' ELSE 'occupied' END
    WHERE id = v_order_record.table_id;
    ```
  - Do NOT touch `reserved_from` here (that's cancellation-specific — confirm whether Mark Paid should also clear it; if unsure, leave it untouched and flag this for review rather than guessing)
- Use `SECURITY DEFINER` if that's the pattern `cancel_active_orders` uses (check and match).

**2. Update `markPaid()` in `app/dashboard/manager/page.tsx` (lines ~69-78)** to call the new RPC instead of the direct `orders` update:
```typescript
async function markPaid(orderId: string) {
  const method = paymentMethods[orderId] || 'cash'
  await supabase.rpc('mark_order_paid', { p_order_id: orderId, p_payment_method: method })
  fetchOrders()
}
```
(Adjust RPC param names to match actual signature you write.)

**3. Apply the migration to the live Supabase DB** (manual SQL Editor, since CLI `db push` doesn't work on this machine per prior notes).

## Scope

Only touch:
- New migration file (new RPC)
- `markPaid()` function in `app/dashboard/manager/page.tsx`

Do not touch the cancellation RPC itself, or any other order status handlers.

## Evidence required

- Migration applied successfully to live DB (confirm via query that `mark_order_paid` RPC exists)
- Build must pass
- Screenshot or Ayush's manual confirmation: mark an order Paid on a table with multiple pending orders, confirm table stays "Occupied"/"Partially Occupied" until the LAST order on that table is paid, then confirm it flips to "Available"
- Report to: `03_Investigation_and_Errors/Chat8_Node_Evidence_ManagerBug3_Fixed.md`

## Next

Do NOT commit/push yet — wait for Ayush's explicit go-ahead after manual verification. This will be batched together with the W#/R# label fix push per Ayush's instruction.
