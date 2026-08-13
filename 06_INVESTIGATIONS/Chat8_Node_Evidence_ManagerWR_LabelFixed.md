# Chat 8: Evidence — Manager Dashboard W#/R# Label Retrofit

**Status:** ✅ Fix Applied & Locally Verified (Compilation Pass)

## What Was Completed

1. **Created Shared Utility (`lib/utils.ts`):**
   - Extracted the label generation logic into a new shared function: `formatOrderNumber`.
   - The function takes an order object and computes the `W#`/`R#` prefix combined with `daily_number`, gracefully falling back to a sliced UUID string if `daily_number` isn't available.

2. **Updated Manager Dashboard (`app/dashboard/manager/page.tsx`):**
   - Imported the new `formatOrderNumber` utility.
   - Replaced hardcoded string formatting (`Order #{order.id.slice(0, 6)}`) with the clean visual representation: `Order {formatOrderNumber(order)}` for both Intake and Billing Queues.
   - Note: Retained the "Order " prefix as it fits seamlessly with the computed `R3` / `W12` styles (rendering as "Order R3" or "Order W12"), which looks natural and aligns with the expected visual style without duplicate pound signs.

## Verification Results
- `npm run build` executed and compiled successfully.
- No other components (Cook, Waiter, Orders) were modified.

## Next Actions
- Ayush to manually test the Manager Dashboard UI and confirm that the queues now display proper W#/R# labels for all orders.
- Awaiting confirmation from Ayush to proceed with the GitHub push instruction. No push has been made yet.
