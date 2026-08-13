# Investigation Report: Notification Insertion Points (Node 9)
**Type: Code-Read Only (No changes made)**

## 1. Order Status change (VERIFIED)
- **Preparing → Ready (Cook):** 
  - File: `app/dashboard/cook/page.tsx`
  - Function: `markReady(order: Order)`
  - Actor context: Cook role is implicitly assumed by the route. No explicit `user.id` is read during this function (it relies on RLS/session for the update).
- **Ready → Served (Waiter):**
  - File: `app/dashboard/waiter/page.tsx`
  - Function: `markServed(order: WaiterOrder)`
  - Actor context: Similar to Cook, relies on session/RLS.
- **Placed → Preparing (Manager):**
  - File: `app/dashboard/manager/page.tsx`
  - Function: `markPreparing(orderId: string)`
  - Note: These are **separate functions** in separate files. Each transition is explicitly handled by the respective role's dashboard.

## 2. Order Cancelled - Bulk Emergency Stop (VERIFIED)
- **File:** `app/dashboard/orders/page.tsx`
- **Function:** `submitBulkCancel` triggers the `supabase.rpc('cancel_active_orders', {...})` call.
- **Pre-RPC State Targeting:** 
  - **Cancel ALL vs Select Specific:** Diverges at line 117 `const payloadIds = bulkMode === 'all' ? null : selectedOrderIds`. `selectedOrderIds` is populated by clicking specific orders on the live board. 
  - **Category/Reason:** Captured via state variables `bulkCategory` (default: 'fire') and `bulkReason` from the modal form. These are passed as `p_category` and `p_reason` to the RPC.
  - Actor context: Owner role is required to open the bulk stop modal (`userRole === 'owner'`).

## 3. Reservation Approved/Rejected (VERIFIED)
- **File:** `app/dashboard/tables/page.tsx`
- **Functions:** `approveRequest(req: ReservationRequest, tableId: string)` and `rejectRequest(req: ReservationRequest)`
- **Customer Context:** Customer reservation requests are created with a mandatory `customer_id` mapping to the logged-in customer's `user.id` (inserted in `app/order/reservation/page.tsx`). The table dashboard updates the status of these requests directly.

## 4. New Reservation Request (VERIFIED)
- **File:** `app/order/reservation/page.tsx`
- **Function:** `submitRequest(e: React.FormEvent)`
- **Recipient Logic:** An insert is made into the `reservation_requests` table. No explicit targeted notification exists yet. Managers/owners see this via a generic realtime fetch on `app/dashboard/tables/page.tsx`.

## 5. New Order Placed (VERIFIED)
- **File:** `app/order/cart/page.tsx`
- **Location:** Inside the checkout flow (`.from('orders').insert({...})`).
- **Separation:** Confirmed that `markPreparing` in `app/dashboard/manager/page.tsx` (Accept → Send to Kitchen) is a completely separate manual step. Order creation only inserts a row with `status: 'placed'`.

## Supporting Context (VERIFIED)
- **Auth/Role Pattern Inconsistency:** 
  - `app/order/cart/page.tsx`, `app/order/reservation/page.tsx`, and `components/Navbar.tsx` use the custom `useAuth()` hook.
  - `app/dashboard/orders/page.tsx` manually calls `await supabase.auth.getUser()` inside a `useEffect` init block.
- **Leftover Bell/Notification Code:** None found. No `notifications` table or bell icon exists in the codebase.
- **Shared Header/Layout:** `components/Navbar.tsx` is the primary shared navigation bar across all roles. A bell icon would need to be inserted here to be visible app-wide.
- **Realtime Channels:** Documented channels found during investigation include:
  - `orders_board` (in `orders/page.tsx`)
  - `manager_orders_realtime` (in `manager/page.tsx`)
  - `cook_orders_realtime` (in `cook/page.tsx`)
  - `reservation_status_realtime` (in `reservation/page.tsx`)
  These names differ slightly from the expected/documented list (e.g. `manager_orders_realtime`, `cook_orders_realtime` instead of `my_orders_realtime`).
