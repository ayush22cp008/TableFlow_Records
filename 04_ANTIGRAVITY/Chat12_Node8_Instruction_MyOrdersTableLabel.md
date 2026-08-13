# Instruction: My Orders — Table Number + Order Label

**Node:** 8 — Customer Dashboard Revamp
**Type:** FIX — Frontend only. Independent of the other two instructions, can be done in any order.

## Changes required

In `app/order/my-orders/page.tsx`:

1. Update the query from `.select('*')` to `.select('*, restaurant_tables(table_number)')` to fetch the joined table number.
2. Import and reuse `formatOrderNumber()` from `lib/utils.ts` (no changes to that function needed — it already accepts `{ id, daily_number?, is_priority? }`, which customer order rows already have).
3. On each order card, display: table number (from the new join) + the formatted order label (O#/W#-style, matching staff dashboard convention).

## Output
Confirm build passes, screenshot of an order card showing the new table number + label. No push — wait for explicit go-ahead per project rule.
