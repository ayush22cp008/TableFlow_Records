# Investigation Result: Table Number + Order Number on My Orders (Customer-Side)

## 1 & 2. Is a table number already captured on orders?
- **YES.** The `orders` schema contains a `table_id` field (UUID linking to `restaurant_tables`).
- **Reliability:** It is extremely reliable. In `app/order/cart/page.tsx`, the `placeOrder` and `placeOrderFromWaitlist` functions both ensure a `table_id` is passed to the `place_order_and_occupy_table` RPC. Even if a user doesn't have a reservation, the system auto-finds the "best-fit available table" and uses its `table_id`. An order is *never* placed without a `table_id` (if no table is available, the user is put on the waitlist instead of placing an order).

## 3. How table is determined today (If missing)
- N/A, since it is present and automatically assigned via capacity-matching logic (or via reservation code) during checkout.

## 4. `formatOrderNumber()` reusability
- **Location & Logic:** Located in `lib/utils.ts`. It takes any object with `{ id, daily_number?, is_priority? }` and returns the `R#` or `W#` label (or fallback `#hash`).
- **Reusability:** **100% reusable.** The customer's order rows from the DB already contain `id`, `daily_number`, and `is_priority`. We can simply import and call `formatOrderNumber(order)` on the customer side without changing its implementation.

## 5. The `my-orders` query
- **Current Query:** In `app/order/my-orders/page.tsx`, the query is `.select('*')`.
- **What's missing:** This fetches all base `orders` columns (including `table_id`, `daily_number`, `is_priority`), which is enough for `formatOrderNumber()`. However, because `table_id` is a UUID, it does **not** include the human-readable `table_number`.
- **Required Change:** The query just needs to be updated to `.select('*, restaurant_tables(table_number)')` to fetch the joined table number for display.
