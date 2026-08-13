# Investigation Results — Node 9: Notification System Discovery (Round 2)

## 1. `unique_code` on Reservations
- **Exact Schema:** The `unique_code` column is of type `text` in the `reservation_requests` table.
- **Generation:** It is generated dynamically upon approval in `app/dashboard/tables/page.tsx` using `Math.floor(100000 + Math.random() * 900000).toString()`, resulting in a 6-digit string.
- **Uniqueness & Persistence:** It is unique per reservation request (generated when the request is approved). It persists indefinitely in the database table alongside the request.
- **Customer UI Flow:** 
  - **Display:** If a customer is logged in, their active reservation is fetched automatically via `customer_id = user.id` in `app/order/reservation/page.tsx`. If it is approved, the UI displays their `unique_code` as "Your Entry Code".
  - **Re-entry / Use:** In `app/order/cart/page.tsx`, there is an input field labeled "Have a reservation code?". The customer can manually enter their `unique_code` here and click "Verify".
- **Lookup Mechanism:** Yes, there is an existing lookup mechanism today. The `verifyReservationCode` function in `app/order/cart/page.tsx` performs a query: `supabase.from('reservation_requests').select(...).eq('unique_code', reservationCode).eq('status', 'approved')`. It verifies the code exists, is approved, and is within 30 minutes of the requested time, returning the reservation details and assigned table.

## 2. Does "Cancel ALL Active Orders" Include `Served` Status Orders?
- **Result:** **No.** An order with the `Served` status can never be included in a bulk cancel.
- **Exact Status List/Filter:** The cancellation is processed via the `cancel_active_orders` RPC in the database. The exact query uses an explicit filter: `WHERE status IN ('placed', 'preparing', 'ready')`.
- **Evidence:** 
  ```sql
  -- From: supabase/migrations/20260809000004_fix_reservation_lifecycle.sql
  SELECT id, table_id, party_size 
  FROM orders 
  WHERE status IN ('placed', 'preparing', 'ready')
    AND (p_order_ids IS NULL OR id = ANY(p_order_ids))
  ```
  Since `served` (and `cancelled`) are omitted from the `IN` clause, they are completely excluded from the cancellation operation.

## 3. Bulk Emergency Stop — Confirm Owner-only
- **Result:** Confirmed. The Bulk Emergency Stop functionality is restricted exclusively to the Owner.
- **Location:** The button exists only in `app/dashboard/orders/page.tsx`. It is wrapped in an explicit role check:
  ```tsx
  {userRole === 'owner' && (
    <button onClick={() => { setBulkModalOpen(true); /* ... */ }}>
      ⚠️ Bulk Emergency Stop
    </button>
  )}
  ```
- **Manager Dashboard:** A search through `app/dashboard/manager/page.tsx` confirms that no equivalent button or modal exists for the Manager route. The capability is not accessible there.
