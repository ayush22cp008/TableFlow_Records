# Chat 8: Evidence — Manager Dashboard Bug 3 (Mark Paid Table Release) Fixed

**Status:** ✅ Fix Applied & Locally Verified (Compilation Pass)

## What Was Completed

1. **New RPC Migration Created:**
   - File: `supabase/migrations/20260809000001_mark_order_paid_rpc.sql`
   - Created `mark_order_paid` RPC with `SECURITY DEFINER` that acts atomically.
   - It updates the order status to `'billed'` and sets the `payment_method`.
   - Crucially, it replicates the cancellation flow's table release logic: decrements `occupied_seats` by `party_size` and sets the table to `'available'` only when `occupied_seats <= 0`. This seamlessly handles tables with multiple orders.

2. **Manager Dashboard Updated (`app/dashboard/manager/page.tsx`):**
   - Refactored the `markPaid()` function.
   - It now calls the newly created `mark_order_paid` RPC via Supabase instead of doing a direct `.update()` on the `orders` table.

## Verification Results
- `npm run build` executed and compiled successfully.
- Code conforms strictly to the instructions without modifying any cancellation logic.

## Next Actions
- **Action for Ayush:** Since the Supabase CLI is unavailable for `db push`, please run the SQL from `20260809000001_mark_order_paid_rpc.sql` manually in the Supabase SQL Editor.
- **Testing:** Verify the fix by marking an order Paid on a table with multiple pending orders. Confirm the table stays "Occupied" / "Partially Occupied" until the LAST order is paid, and then finally flips to "Available".
- Awaiting confirmation from Ayush to proceed with the batch GitHub push instruction. No push has been made yet.
