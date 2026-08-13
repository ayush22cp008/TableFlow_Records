# Instruction for Antigravity — Chat 16 / Node 9 — Investigation Only

**Type: INVESTIGATION ONLY. Do NOT write/modify any code, do NOT create the `notifications` table, do NOT design schema. Read-only code investigation.**

## Goal
Find the exact file + function + code location for each of the 5 locked Node 9 notification event types, plus supporting context needed before schema/UI design starts.

## Investigate — 5 Event Types

For each, report: file path, function/handler name, the relevant code block (as-is, no changes), and how the actor's `user_id`/role is available at that point.

1. **Order Status change** — Preparing → Ready → Served. Find where each transition is triggered (likely Cook/Waiter dashboards). Note if all three transitions go through one function or separate ones.

2. **Order Cancelled (Bulk Emergency Stop)** — Find the `cancel_active_orders` RPC call site in Owner's frontend. Report:
   - Exact query/filter used to fetch affected orders BEFORE the RPC call (needed for status-snapshot targeting per locked design)
   - Where "Cancel ALL" vs "Select Specific Orders" modes diverge in code
   - Where Category/Additional Details (cancel reason) are captured

3. **Reservation Approved/Rejected** — Find Manager's approve/reject action (file + function). Confirm it only applies to logged-in customers per locked scope (i.e., how logged-in vs anonymous reservations are distinguished in this code path).

4. **New Reservation Request** — Find the customer-side submit function. Confirm current recipient logic (should reach Manager only, not Owner).

5. **New Order Placed** — Find where a new order row is created (customer checkout/cart flow). Also locate `markPreparing` in `app/dashboard/manager/page.tsx` (the "Accept → Send to Kitchen" action) and confirm it is a separate, manually-triggered step from order creation — do not conflate the two.

## Also confirm (supporting context)

- Current auth/role access pattern used across the files above (`supabase.auth.getUser()` vs `useAuth()` context) — note any inconsistency between files.
- Any existing bell icon, notification component, or `notifications`-related leftover code/table anywhere in the codebase (expected: none, per Chat 15 ground truth — just confirm still true).
- Current shared layout/header file(s) per role (Owner/Manager/Waiter/Cook/Customer) — where a bell icon would need to be inserted to be visible app-wide for that role.
- List of existing Supabase Realtime channels currently in use (confirm against known list: `orders_board`, `my_orders_realtime`, `menu_realtime`, `tables_realtime`) — flag if any additional channel exists that wasn't previously documented.

## Output format
Structured report, one section per numbered item above. Tag each finding as VERIFIED (seen directly in code) — this is a code-read task so everything should be VERIFIED, not INFERRED. If something can't be found, say so explicitly rather than guessing.

## Rules
- No fixes, no refactors, no new files, no schema/table creation.
- No side quests — if unrelated bugs/issues are spotted, note them separately at the end under "Unrelated observations" and do not act on them.
