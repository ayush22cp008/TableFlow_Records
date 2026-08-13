# Chat 9: Evidence — Reservation Requests Panel Clutter Fix (Option C)

**Status:** ✅ Fix Applied & Locally Verified (Compilation Pass)

## What Was Completed

1. **Part 1: Query-level Date Window (`app/dashboard/tables/page.tsx`)**
   - Modified `fetchData` to only fetch `reservation_requests` where `requested_time >= startOfToday`. 
   - This immediately cuts out all historical rows from previous days (which comprised the bulk of the 50 rows).
   - Only today's `pending`, `approved`, and `completed` requests are loaded into state.

2. **Part 2: Client-side Filter Fix (`app/dashboard/tables/page.tsx`)**
   - Updated the `visibleReservations` logic.
   - For `completed` status ("Seated" entries), it now hides the entry exactly 5 minutes after the customer is seated (using either the `linkedOrder.created_at` timestamp or `requested_time`).
   - The logic no longer waits for the entire order lifecycle (billing) to finish, instantly resolving the visual clutter issue while keeping the panel focused on upcoming arrivals.

## Constraints Respected
- **No DB Modifications:** The database rows are completely untouched. This is strictly a fetch and display fix.
- **Node 3 & Node 4 untouched:** The `mark_order_paid`, `cancel_active_orders` RPCs, and the `approveRequest` overlap checks remain locked and unchanged.
- Fix is minimal and directly addresses the UI clutter.

## Next Actions for Ayush
1. Please open the Tables dashboard in your browser.
2. Check the Reservation Requests side panel — the old completed requests (e.g., mahesh, Za, ajh) should no longer be visible.
3. Test approving a new request, seating them, and observing it disappear from the side panel shortly after seating (vs hovering there forever).
4. Send instruction to batch commit & push once tested.
