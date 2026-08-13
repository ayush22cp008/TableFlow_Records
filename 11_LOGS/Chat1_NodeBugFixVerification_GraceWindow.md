# Result — Verify 30-min Grace Window Expiry

Chat #1 | Node: Bug Fix Verification (Manual DB Test)

## 1. Test Setup
*   **Target Row:** Created a dummy `reservation_requests` row (Customer: "Ayush Expiry Test", ID: `9a8f...`) with status = `approved` and NO linked order.
*   **Original `requested_time`:** Set to `2026-08-04T00:50:00+05:30` (current time).
*   **Backdated `requested_time`:** Manually updated in Supabase to 40 minutes in the past (`2026-08-04T00:10:00+05:30`).

## 2. Verification Results

### A. Dashboard Panel Visibility (`/dashboard/tables`)
*   **Result:** The reservation request **disappeared (HIDDEN)** from the "Reservation Requests" panel.
*   **Reason:** The computed filter evaluates `now <= requestedTime + 30 * 60 * 1000`. Since 40 minutes have passed, it correctly evaluated to false and hid the stale request.

### B. Code Validity in Cart (`/order/cart`)
*   **Result:** Entering the code returned the error: **"This reservation code has expired (30 minute grace period)."**
*   **Reason:** The cart verification logic evaluates `now > reservationTime + 30 * 60 * 1000`. Since 40 minutes have passed, it correctly triggers the expiry error message.

## Conclusion
Both conditions work exactly as intended. The auto-hide logic successfully removes expired no-show requests from the dashboard, and the code validity properly blocks them on the customer side.
