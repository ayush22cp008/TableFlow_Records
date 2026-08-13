# Investigation Results — Node 9: Notification System Discovery

## 1. Database Schema
- **Result:** No `notifications` table or similar exists in the database.
- Confirmed by scanning all Supabase migration files. There is no existing schema, columns, or RLS policies for notifications.

## 2. Existing Realtime Channels
- **`orders_board`**: Located in `app/dashboard/orders/page.tsx`. It uses Supabase Realtime (`postgres_changes`) and subscribes to all events (`*`) on the `orders` table.
- **`my_orders_realtime`**: Located in `app/order/my-orders/page.tsx`. It uses `postgres_changes` and subscribes only to `UPDATE` events on the `orders` table.
- **`menu_realtime`**: Located in `app/order/page.tsx`. It uses `postgres_changes` and subscribes to all events (`*`) on the `menu_items` table.
- **`waitlist_realtime`**: A channel with this exact name does not exist. However, `app/dashboard/tables/page.tsx` creates a channel named `tables_realtime` that uses `postgres_changes` to subscribe to events on four tables: `waitlist`, `restaurant_tables`, `reservation_requests`, and `orders`.

## 3. Order Status Change Flow
- **Location:** The flow is primarily handled by the `advanceStatus` function in `app/dashboard/orders/page.tsx`, as well as role-specific functions like `markPreparing` in `app/dashboard/manager/page.tsx`.
- **Trigger Exact Point:** A manual button click on the dashboard triggers a direct Supabase update query (e.g., `supabase.from('orders').update({ status: next })`). It does not use an RPC for the status transition.

## 4. New Order Placed → Manager Flow
- **Location:** The "Accept (Send to Kitchen)" flow is located in `app/dashboard/manager/page.tsx` (`markPreparing` function).
- **Trigger Mechanism:** This is a **separate manual action**. It is not automatically triggered by a new order insert. When an order is inserted, it defaults to the `placed` status. The Manager clicks the button to manually transition the order from `placed` to `preparing`.

## 5. Bulk Emergency Stop Cancellation Flow
- **Location:** The Bulk Cancel modal and submission logic (`submitBulkCancel`) live in `app/dashboard/orders/page.tsx`.
- **Cancel Reason Storage:** The cancellation category and details are captured in the modal UI and passed to the `cancel_active_orders` RPC. They are stored directly in the `orders` table in the `cancellation_category` and `cancellation_reason` columns.
- **Old Status Accessibility:** The order status at the time of cancellation is **not readily accessible** after the fact. The RPC executes a bulk update in the database that directly overwrites the `status` column with `'cancelled'`. The previous status is lost, meaning status-based staff targeting would require the frontend to map the current state *before* triggering the RPC, or the RPC would need to be modified to log the old status.

## 6. Reservation Approve/Reject Flow
- **Location:** The `approveRequest` and `rejectRequest` functions are located in `app/dashboard/tables/page.tsx`.
- **Customer User ID Accessibility:** The Customer's user ID is **NOT accessible** here. The `reservation_requests` table only contains a `customer_name` string and `unique_code`, but no `user_id` or `customer_id`. Customers are unauthenticated (anonymous) during this flow, so targeting a notification to a specific user session is not currently possible with the existing schema.

## 7. Bell Icon / Notification UI
- **Result:** Confirmed. There is currently **NO** existing bell icon or notification list component anywhere in the codebase (neither in `components/` nor `app/`).

## 8. Roles and User ID Access
- **Mechanism:** The application identifies the logged-in user via `supabase.auth.getUser()` or the `useAuth()` context (which uses `supabase.auth.getSession()`).
- **Role Check:** The user's role is determined by joining/querying the `profiles` table using the authenticated user's ID: `supabase.from('profiles').select('role').eq('id', user.id).single()`.
- **Note on Dashboards:** Some role-specific dashboards (like Manager or Cook) do not query the user ID directly in their local `page.tsx` components. They rely on middleware and Supabase RLS to restrict access. If we need to target notifications, we will have to look up the `user_id` for specific roles or use role-based channels/RLS policies.
