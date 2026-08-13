# Investigation Results: Stale Reservation Requests

## 1. Query/Component populating the "Reservation Requests" panel

*   **File:** `app/dashboard/tables/page.tsx`
*   **Function:** `fetchData` (Lines 51-61)
*   **Query:** 
    ```typescript
    supabase.from('reservation_requests')
      .select('*')
      .in('status', ['pending', 'approved'])
      .order('requested_time')
    ```
*   **Filter Logic:** It fetches **all** rows where `status` is `'pending'` or `'approved'`. There is **no date/time filter** and **no completion filter**.

## 2. Row State in `reservation_requests`

I queried the DB directly for these two users. Here is the raw data:

```json
[
  {
    "id": "b7d766b9-6f9c-44d0-bffd-30b367536c97",
    "customer_name": "viral",
    "party_size": 1,
    "requested_time": "2026-07-30T14:30:00+00:00",
    "status": "approved",
    "unique_code": "746699",
    "table_id": "8937cad2-7270-4858-b90b-27586cb3da88"
  },
  {
    "id": "d96124b8-2791-49d6-ae4b-354a5a1703f0",
    "customer_name": "ayushtest",
    "party_size": 3,
    "requested_time": "2026-08-02T15:10:00+00:00",
    "status": "approved",
    "unique_code": "957247",
    "table_id": "80515082-0ed0-4b03-9029-1c7a633288d7"
  }
]
```
Both records are currently stuck in the `"approved"` status in the database.

## 3 & 4. Why are they stuck and how were orders created?

To understand how a customer ordered but the reservation stayed "Approved", we look at `app/order/cart/page.tsx`:

1.  **Cart Code Verification (Lines 74-101):** 
    For a customer to use their reservation code in the cart, the cart explicitly checks:
    ```typescript
    .eq('unique_code', reservationCode)
    .eq('status', 'arrived')
    ```
    **Requirement:** The owner *must* click "Confirm Arrival" on the dashboard (which sets status to `"arrived"`) *before* the customer can enter their code to order.
2.  **Order Completion (Line 181):**
    If the customer successfully uses the code, placing the order triggers:
    `await supabase.from('reservation_requests').update({ status: 'completed' }).eq('id', verifiedReservation.id)`

**The Disconnect / Bypass Scenario:**
Because the status for these two is still `"approved"`, it proves **"Confirm Arrival" was never triggered**. 
If a customer tries to enter their code *before* the owner confirms arrival, the cart throws an error ("Invalid or unused code..."). 
To proceed, the customer likely just placed a normal walk-in order (without entering a code). The system then treated them as a new walk-in, created a brand new order, and **completely orphaned the original reservation request**.

Since the reservation request was orphaned in the `"approved"` state, and `app/dashboard/tables/page.tsx` has no time-based expiration (e.g. expiring old approved requests after 2 hours), they will stay on the dashboard forever.

## 5. Summary of Relevant Code Paths

*   **Dashboard Query (No Date Filter):** `app/dashboard/tables/page.tsx`, Line 55
*   **Arrival Requirement for Cart:** `app/order/cart/page.tsx`, Line 82
*   **Completion Trigger (Only fires if code used):** `app/order/cart/page.tsx`, Line 181
