# Investigation Result: Reservation Status Feature (Customer-Side)

## 1. Reservation Schema & Statuses
The reservation logic is backed by the `reservation_requests` table.
- **Statuses/Enum:** The `status` field is text-based but strongly enforced in the frontend as `'pending' | 'approved' | 'rejected' | 'arrived' | 'completed' | 'cancelled'`.

## 2. 30-Minute No-Show Auto-Release
- **Does it exist in code?** **NO.** There is no automated cron job or database trigger that automatically releases a table or cancels a reservation if a customer is 30 minutes late. 
- **What *does* exist?** The "30 minute" logic you recall is purely a visual filter in `app/dashboard/tables/page.tsx` and `app/order/cart/page.tsx`. It prevents walk-ins from being seated at a table if that table has an upcoming reservation within the *next* 30 minutes (`Date.now() >= reservedTime - 30 mins`). But if a customer is late, the table stays in the "Reserved" state indefinitely until an owner manually clears it or the reservation is fulfilled.
- **Cleanup Logic:** The only automated cleanup occurs in `mark_order_paid` and `cancel_active_orders` RPCs, which mark approved requests as 'cancelled' when a table's occupancy drops to 0.

## 3. Owner Manual Clear Action
- **Does it exist?** **YES.** 
- **Location:** In `app/dashboard/tables/page.tsx`, owners can click a "Clear" button on any reserved table. This triggers the `clearReservation(table)` function, which sets `reserved_from = null` on the table and bulk-updates any linked `approved` requests in `reservation_requests` to `cancelled`.

## 4. Querying a Customer's Reservation
- **Current Linkage:** `reservation_requests` does **NOT** have a `customer_id`, `user_id`, or `email` field. It only stores a string `customer_name`.
- **How to fetch:** Currently, customers must go to `/reserve/status` and manually type in their name. The system performs an `ilike('customer_name', '%name%')` query to find their requests. To show a logged-in customer their reservations automatically on a dashboard, we would need to either add a `customer_id` column to `reservation_requests` or match against their profile email/name.

## 5. Available Fields for Display
The `reservation_requests` table provides:
- `customer_name`
- `party_size`
- `requested_time`
- `status`
- `unique_code` (Entry code shown upon approval)
- `table_id` (UUID; requires joining `restaurant_tables(table_number)` to show the actual table number).
