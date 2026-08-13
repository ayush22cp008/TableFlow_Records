# TableFlow — Chat #8 — Instruction (Fix Only)

**To:** Antigravity
**From:** Claude
**Type:** Fix — Bug 3b (Reservation table release for Manager Mark Paid)

---

## Context

Per investigation (`Chat8_Node_Investigation_ManagerBug3b_ReservationTableRelease.md`): `mark_order_paid` RPC never clears `reserved_from`, so reservation orders stay visually "Reserved" (purple) after Manager marks them Paid, even though `occupied_seats` correctly drops to 0. Root cause and fix confirmed.

**Bonus finding (do NOT fix in this pass, just noted for record):** Owner's Generate Bill flow (`app/dashboard/billing/[orderId]/page.tsx`, lines 62-69) unconditionally sets `reserved_from = null`, which has a latent multi-order bug (clears reservation state even if the table has other unpaid orders). This is out of scope for this fix — will be tracked separately.

## Fix

**Update the `mark_order_paid` RPC** (in `supabase/migrations/20260809000001_mark_order_paid_rpc.sql`, or as a new migration `20260809000002_mark_order_paid_reserved_from_fix.sql` if the original is already applied and you prefer not to edit an applied migration — use whichever matches this project's existing migration conventions) to conditionally clear `reserved_from`:

```sql
UPDATE restaurant_tables
SET occupied_seats = GREATEST(0, occupied_seats - v_order_record.party_size),
    status = CASE WHEN (occupied_seats - v_order_record.party_size) <= 0 THEN 'available' ELSE 'occupied' END,
    reserved_from = CASE WHEN (occupied_seats - v_order_record.party_size) <= 0 THEN NULL ELSE reserved_from END
WHERE id = v_order_record.table_id;
```

This only clears `reserved_from` when the table is fully releasing (last order paid), correctly handling multi-order tables.

## Scope

Only touch the `mark_order_paid` RPC (new migration or update to existing, per your judgment on migration conventions). Do NOT touch:
- Owner's Generate Bill flow (`app/dashboard/billing/[orderId]/page.tsx`) — its bug is noted but out of scope
- `app/dashboard/manager/page.tsx` — no changes needed here, it already calls the RPC correctly

## Evidence required

- Migration applied successfully to live DB (manual SQL Editor, per standing note that CLI `db push` doesn't work on this machine)
- Build must pass
- Screenshot or Ayush's manual confirmation: mark a Reservation (R#) order Paid via Manager Dashboard, confirm table flips from purple/Reserved to green/Available correctly
- Also re-confirm multi-order case still works: a table with 2 orders (at least one reservation) — paying the first should NOT release the table; paying the last should
- Report to: `03_Investigation_and_Errors/Chat8_Node_Evidence_ManagerBug3b_Fixed.md`

## Next

Do NOT commit/push yet — this will be batched together with the W#/R# label fix and the original Bug 3 fix, per Ayush's standing instruction to batch pushes.
