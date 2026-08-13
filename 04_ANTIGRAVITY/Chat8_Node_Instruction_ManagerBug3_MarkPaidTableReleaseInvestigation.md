# TableFlow — Chat #8 — Instruction (Investigation Only)

**To:** Antigravity
**From:** Claude
**Type:** Investigation — NO code changes, NO fixes in this prompt

---

## Task

Bug 3: After a Manager marks an order Paid on the Billing Queue, the corresponding table's occupancy status does not update. Confirmed via manual testing: Table 5 had two orders (Order W11, Order W12) both marked Paid successfully, but the Tables page still showed Table 5 as "Partially Occupied" instead of releasing back to Available.

Investigate the root cause. Do not fix yet — report findings only.

## What to Investigate

1. **Find the Mark Paid handler.** In `app/dashboard/manager/page.tsx`, locate the function triggered by the "Mark Paid" button. Report exact file + line reference.

2. **Confirm what it actually does.** Does it only update `orders.status` (e.g. to `'billed'` or `'paid'`), or does it also call any table-release logic? Report the exact Supabase update/RPC call(s) made.

3. **Compare against the working cancellation flow.** The single-order cancellation and Owner bulk emergency stop flows are confirmed working and do release tables properly (per earlier locked work — `cancel_active_orders` RPC and related). Check how those flows release table occupancy (e.g. do they update `restaurant_tables.seated`/`occupied` count directly, or call a dedicated RPC?). Report the exact mechanism they use.

4. **Check table occupancy source of truth.** Look at the `restaurant_tables` table schema and the `place_order_and_occupy_table` RPC (used when an order is placed) — confirm exactly which column(s) represent seat occupancy (e.g. `seated_count`, `is_occupied`) and how they get incremented on order placement, so we know what needs to be decremented/reset on payment.

5. **Determine likely root cause.** Is Mark Paid simply missing a call to release/decrement table occupancy that the cancellation flow already has? Or is there a different mechanism expected (e.g. table should only release when ALL orders for that table are paid, not per-order)? Table 5 in the reported bug had 2 separate orders — confirm whether release logic (if it existed) would need to account for multiple orders per table before considering it "free."

## What to Report Back

- Exact Mark Paid handler code (file + lines)
- Exact confirmation of what DB calls it makes today (status update only, or also table release)
- Exact working mechanism used by cancellation flow to release tables, for comparison
- Table occupancy schema — which column(s) track it, how they're incremented on order placement
- Root cause conclusion: what's missing, and whether the fix needs to account for multiple orders per table

Do **not** apply any fix in this pass — investigation and recommendation only. The fix instruction will be a separate prompt once this is reviewed.

## Output

Save findings to: `03_Investigation_and_Errors/Chat8_Node_Investigation_ManagerBug3_MarkPaidTableRelease.md`

Report back to Claude with the file path once done.
