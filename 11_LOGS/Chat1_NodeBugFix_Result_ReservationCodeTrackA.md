# Result — Reservation Code Flow (Track A)

Chat #1 | Node: Bug Fix

## 1. Files Changed
*   `app/dashboard/tables/page.tsx`:
    *   Removed `confirmArrival` function.
    *   Removed the "Confirm Arrival" button and code input field from the UI rendering for the "Approved" list.
*   `app/order/cart/page.tsx`:
    *   Updated `verifyReservationCode` query to `.eq('status', 'approved')` instead of `'arrived'`.
    *   Added fetching of `requested_time`.
    *   Added conditional logic: `if (Date.now() > requestedTimeMs + 30 * 60 * 1000)` to trigger the exact same error as an invalid/missing code.

## 2. Build / Compile Result
*   **Result:** `✓ Compiled successfully`
*   No TypeScript errors or build issues. Static pages generated successfully.

## 3. Manual Testing Steps (Ayush to Verify)

**Test Case 1: Code works within grace window**
1.  As Owner: Go to `/dashboard/tables`, approve a pending reservation request (ensure the requested time is within the last 30 minutes, or in the future).
2.  Note the 6-digit code shown in the Supabase DB or UI (if visible in your DB editor).
3.  As Customer: Go to `/order/cart`.
4.  Enter the code in the "Have a reservation code?" field.
5.  **Expected:** Code is accepted, assigns table, and placing the order successfully sets the DB status to `completed`.

**Test Case 2: Code expired after grace window**
1.  In your Supabase SQL editor/dashboard, manually edit an approved reservation's `requested_time` to be **40 minutes ago** (e.g. `now() - interval '40 minutes'`).
2.  As Customer: Go to `/order/cart`.
3.  Enter the exact valid code for that reservation.
4.  **Expected:** The cart should show the red error text: `"Invalid or unused code, or missing table assignment."` (It behaves exactly like an invalid code because it exceeded the 30-minute grace period).
