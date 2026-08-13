# Result — Auto-Hide Reservation Requests

Chat #1 | Node: Bug Fix

## 1. What Was Changed
*   **Target File:** `app/dashboard/tables/page.tsx`
*   **Visibility Logic Computed at Render Time:** 
    *   Instead of rendering all fetched requests, I introduced a `visibleReservations` array that evaluates each request.
    *   **Condition 1 (Order Completed):** It fetches the `orders` table dynamically in realtime. If it finds an order linked to the reservation's assigned table that was created within the grace window, it checks the order's status. If the status is `billed`, the reservation request is hidden exactly 5 minutes after the `updated_at` (bill generation) timestamp.
    *   **Condition 2 (No-Show/Expiration):** If no linked order exists, the request stays visible until the grace window ends (`requested_time + 30 mins`), at which point it automatically disappears.
    *   **Immediate Hiding Prevention:** Even if the reservation status instantly changes to `completed` when the code is used in the Cart, the dashboard still queries `completed` statuses so they stay visible during the dining session and only hide *after* the bill is generated.

## 2. Technical Adjustments
*   **TypeScript Types (`types/index.ts`):** I formally added the `'completed'` state to the `ReservationRequest` status type since it was missing but utilized by the backend, ensuring ESLint compliance.
*   **Database Queries:** Updated `fetchData()` and realtime listeners to include the `orders` table and `completed` reservation states.

## 3. Build & Verify
*   Verified that `npm run build` compiles with 0 errors.
*   The exact conditions requested in Track A follow-up have been strictly satisfied without creating new redundant database flags.

You can now push these changes and test the flow manually as requested!
