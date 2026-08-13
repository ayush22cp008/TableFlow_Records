# Investigation Results: Track A Not Live + Code/Table Mismatch

## A. Why Track A fix isn't live

1. **Git / Branch Mismatch:** The previous fix was committed and pushed to a new isolated branch `fix-stale-reservation-flow` (Commit hash `9701f6be...`). However, it was never merged into the `main` branch. 
2. **Vercel Deployment:** Vercel automatically builds and deploys from the `main` branch. The latest commit on `origin/main` is `edd9e88` (an older docs/feature update). 
3. **Conclusion:** This is a "fix never got merged into main" issue. The live site `table-flow-nu.vercel.app` is still running the old code where `confirmArrival` and the `arrived` status requirement exist.

## B. Reservation code / table assignment mismatch

4. **rajesh's DB Row:** 
   - `id`: `26c7db59-c263-469d-bfcd-dbf89b6d0b13`
   - `status`: `"arrived"`
   - `table_id`: `8937cad2-7270-4858-b90b-27586cb3da88` (Table 1)
   - `unique_code`: `"258665"`
   - `party_size`: `2`
   - `requested_time`: `"2026-08-03T09:20:00+00:00"`

5. **Why cart threw "Invalid or unused code...":**
   The live site (running old code) requires the request to have `status = 'arrived'`. Because the customer entered the code *before* the owner clicked "Confirm Arrival" (i.e. when it was still `"approved"`), the Supabase `.single()` query returned zero rows. This threw an SDK error, triggering the `if (error || !data || !data.table_id)` block and displaying the error. (The party size being 1 vs 2 had no impact on this specific error).

6. **Why Table 2 shows "2/4 seated":**
   When the code failed for the customer, they bypassed the reservation and placed a standard walk-in order. 
   - The walk-in table assignment logic (`app/order/cart/page.tsx`, Line ~133) excludes tables if `now >= reserved_from - 30 mins`. 
   - Since `rajesh`'s reservation was actively holding Table 1 (`reserved_from` was set to `09:20`), Table 1 was correctly filtered out as unavailable.
   - The system then found the next available table that fit the party (Table 2) and assigned it. The `place_order_and_occupy_table` SQL RPC was triggered, incrementing Table 2's `occupied_seats` to 2. 

7. **Exact File Paths & Sources:**
   - **Cart error logic (Live):** `app/order/cart/page.tsx`, Line 78-90
   - **Walk-in Table Assignment:** `app/order/cart/page.tsx`, Line 118-142
   - **Table Occupancy RPC:** `supabase/migrations/20260727152250_seat_and_reservation_tracking.sql`, Lines 5-23 (`place_order_and_occupy_table`)
