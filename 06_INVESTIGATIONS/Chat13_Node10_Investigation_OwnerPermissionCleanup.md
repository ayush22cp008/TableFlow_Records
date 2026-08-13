# Investigation — Node 10: Owner Permission Cleanup

This document outlines the overlapping Owner/Staff operational permissions and specifies the required changes to restrict the Owner role to a primarily read-only/monitoring capacity for day-to-day operations.

## 1. Live Orders — status action buttons
- **Component / Path:** `app/dashboard/orders/page.tsx`
- **Actions:** 
  - `advanceStatus` function handles `preparing` -> `ready` -> `served` via `supabase.from('orders').update({ status: next })`.
  - `submitCancel` function handles the plain Cancel button, updating `orders` and `restaurant_tables` directly.
- **Bulk Emergency Stop:** Verified to be a fully separate code path in the same file. It uses `submitBulkCancel` which calls the `cancel_active_orders` RPC.
- **RLS Status:** Owner currently has direct table update access via the `"owner_all_updates"` policy on the `orders` table (found in `20260804000002_node2b_cancellation.sql`).
- **Verdict:** **DB/RLS-level restriction also required** (Owner's update policy on `orders` must be modified/dropped to prevent manual API bypassing).

## 2. Bill page/route
- **Component / Path:** `app/dashboard/billing/[orderId]/page.tsx`
- **Actions:** `generateBill` function directly calls `.update()` on `restaurant_tables` (to free seats) and `orders` (to set `status = 'billed'`).
- **Manager Flow:** Manager does NOT use this route. Manager billing is located in `app/dashboard/manager/page.tsx` and uses a completely separate DB implementation: `supabase.rpc('mark_order_paid')`.
- **RLS Status:** Owner relies on their direct `update` access to `orders` and `restaurant_tables`.
- **Verdict:** **DB/RLS-level restriction also required**.

## 3. Reservation approve/reject
- **Component / Path:** `app/dashboard/tables/page.tsx`
- **Actions:** `approveRequest` and `rejectRequest` functions directly call `.update()` on `reservation_requests`.
- **Manager Flow:** Manager uses the exact same component (`app/dashboard/tables/page.tsx`), as both roles are routed here from the Navbar.
- **RLS Status:** The `reservation_requests` table currently has an open public update policy: `"Allow public update" ON reservation_requests FOR UPDATE TO public USING (true)`.
- **Verdict:** **DB/RLS-level restriction also required** (RLS must be properly scoped to Manager-only to secure the table at the DB level).

## 4. Tables page
- **Component / Path:** `app/dashboard/tables/page.tsx`
- **Actions:** 
  - `cycleTableStatus`, `setReservation`, `clearReservation` (updates `restaurant_tables`).
  - `seatWaitlistEntry`, `cancelWaitlistEntry` (updates `waitlist`).
- **Manager Flow:** Manager uses this exact same component.
- **RLS Status:** Owner has explicit write access via `"tables_write"` on `restaurant_tables` and `"waitlist_update"` on `waitlist`.
- **Verdict:** **DB/RLS-level restriction also required** (Owner must be removed from the allowed roles in the RLS policies).

## Additional Findings
- **Shared Components:** `app/dashboard/tables/page.tsx` is fully shared between Owner and Manager. To hide the action buttons from the Owner, conditional rendering (e.g., checking if `userRole === 'manager'`) will be necessary inside this file.
- **Other Overlapping Actions:** The Menu Page (`app/dashboard/menu/page.tsx`) allows both Owner and Manager to toggle menu item availability (`is_available`). This is another operational action that duplicates Manager functionality. (RLS policy `"menu_write"` currently allows both manager and owner).
