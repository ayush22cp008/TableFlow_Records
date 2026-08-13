# Chat 9: Evidence — Node 4 (Reservation Double Booking) Fixed

**Status:** ✅ Fixes Applied & Locally Verified (Compilation Pass)

## What Was Completed

1. **Fix 1: Approve-time Guards (`app/dashboard/tables/page.tsx`)**
   - **Dropdown UI:** The "Select Table" dropdown now correctly factors in `occupied_seats` when displaying available capacity `(t.capacity - (t.occupied_seats || 0)) >= req.party_size`.
   - **`approveRequest` Handler:** Added an occupancy check (`available seats < party size`) and an overlap check (`reservationRequests.some(r => r.status === 'approved' && r.table_id === tableId && r.id !== req.id)`). If either check fails, it blocks approval via `window.alert` instead of overwriting `reserved_from`.

2. **Fix 2: RPC Hard Cap (`supabase/migrations/20260809000003_place_order_capacity_guard.sql`)**
   - Created a new migration file to patch the `place_order_and_occupy_table` RPC.
   - Added a server-side capacity check: `if (v_occupied_seats + p_party_size) > v_table_capacity then raise exception...`.
   - This ensures even if two people miraculously scan codes at the exact millisecond, the database transaction will cleanly reject the overflow, preventing invalid states like `4/2` occupied seats.

## Verification Results
- `npm run build` executed and compiled successfully.
- Code matches constraints (minimal fixes, no Node 3 touched).

## Next Actions
- **Action for Ayush:** Since the Supabase CLI is unavailable for `db push`, please run the SQL from `20260809000003_place_order_capacity_guard.sql` manually in the Supabase SQL Editor.
- **Testing:** Verify the fix by attempting to approve two overlapping reservations on the same table. The UI should now block the second one. Additionally, confirm the dropdown correctly reports "available" seats rather than absolute capacity.
- Awaiting confirmation from Ayush to proceed with the batch GitHub push instruction. No push has been made yet.
