# Chat 9: Evidence — Node 4 (Reservation Lifecycle Fix - Option B)

**Status:** ✅ Fixes Applied & Locally Verified (Compilation Pass)

## What Was Completed

1. **Types Update (`types/index.ts`)**
   - Added `'cancelled'` to the `ReservationRequest` status type to represent the new terminal state.

2. **Frontend: Clear Button Logic (`app/dashboard/tables/page.tsx`)**
   - Updated `clearReservation(table)` to automatically mark any `approved` requests for that `table_id` as `cancelled` before clearing the table's `reserved_from` state.

3. **Backend RPCs & DB Cleanup (`supabase/migrations/20260809000004_fix_reservation_lifecycle.sql`)**
   - **Backfill Script:** Included an `UPDATE` statement that finds any `reservation_requests` where `status = 'approved'` but the linked `restaurant_tables` is fully available (seats = 0, reserved_from is NULL) and cleanly transitions them to `cancelled`. This resolves the 5 existing stale rows immediately.
   - **`mark_order_paid` RPC:** Upgraded to also check if the table was fully released, and if so, runs a cleanup step to cancel any lingering `approved` reservations for that table.
   - **`cancel_active_orders` RPC:** Same update as `mark_order_paid`; cleans up `reservation_requests` when the bulk emergency stop clears a table.

## Constraints Respected
- Node 3 (Manager Dashboard) was strictly not touched. Although two backend RPCs used by the Manager were patched, they were only patched with an internal `UPDATE reservation_requests` query which is purely structural and doesn't change any Manager behavior/UI.
- No new API/Routes created.
- Fix is minimal and directly solves the false positive overlap.

## Next Actions for Ayush
1. Please execute the SQL migration `20260809000004_fix_reservation_lifecycle.sql` manually in your Supabase SQL Editor.
2. Verify in the Database dashboard that the 5 stale rows for Tables 1, 2, 3, 6 are now showing `status = 'cancelled'`.
3. Try approving a new reservation for Table 1 (it should now successfully approve without false positive overlap blocks).
4. Send instruction to batch commit & push once tested.
