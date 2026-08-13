# Instruction — Node 10: Owner/Staff Role Overlap Cleanup (INVESTIGATION ONLY, NO FIX)

## Background
Owner dashboard currently duplicates operational actions that Manager/Cook/Waiter already perform on their own dedicated dashboards. This creates overlapping access and race-condition risk (e.g. Owner and Manager can both act on the same reservation). Decision made: Owner becomes primarily a **read-only monitoring role** for day-to-day operations, keeping only true emergency override capability. This node is a prerequisite for Node 9 (Notifications), since notification targeting depends on finalized role permissions.

## Task
Investigate (do NOT modify) the following four areas of Owner access, and report exactly where each is implemented in code, and whether restriction needs to happen at UI level only or also at DB/RLS level.

## Areas to investigate

### 1. Live Orders — status action buttons
- Locate the Owner's Live Orders component (`preparing`, `ready`, `served`, `Bill`, and plain `Cancel` buttons — the plain Cancel is separate from Bulk Emergency Stop).
- Note exact file/component path for each button and the function/RPC each one calls.
- Confirm Bulk Emergency Stop is a fully separate code path (should remain untouched).

### 2. Bill page/route
- Locate the route/page opened when Owner clicks "Bill" (shows itemized bill, service charge toggle, "Generate Bill & Mark as Billed").
- Note the route path, component file, and what RPC/mutation "Generate Bill & Mark as Billed" calls.
- Check if this route is shared with Manager's billing flow (Manager Dashboard "Mark Paid") or a separate implementation.

### 3. Reservation approve/reject
- Locate where Owner currently has approve/reject action on Reservation Requests panel.
- Confirm whether Manager's approve/reject uses the same component/RPC or a different one.
- Note current RLS policy (if any) on the reservations table for both Owner and Manager roles — does the DB currently allow both roles to write to reservation status, or is this only a frontend-exposed action?

### 4. Tables page
- Locate Owner's Tables page (Reserve/Clear buttons, "Click to toggle reserved").
- Note whether Owner's Tables component is the same one Manager uses, or separate.
- Note current RLS policy (if any) on the tables table for Owner vs Manager write access.

## Also report
- For each of the 4 areas above, explicitly state: **"UI-only restriction sufficient"** or **"DB/RLS-level restriction also required"** — based on whether Owner currently has direct write permission at the database level (not just a visible button).
- Any shared component between Owner and Manager dashboards that would need conditional rendering (same component, role-based prop) vs. fully separate components.
- Any other Owner-side action not listed above that duplicates a Manager/Cook/Waiter action (flag if found, don't act on it).

## Output
Write findings to `03_Investigation_and_Errors/Chat13_Node10_Investigation_OwnerPermissionCleanup.md` in the Drive bridge folder. Structured by the 4 areas above, each with: file paths, RPC/mutation names, current RLS status, and the UI-only vs DB-level restriction verdict.

## Constraints
- Read-only investigation. Do not modify any project source file, RLS policy, or run any migration.
- No terminal changes beyond grep/search/read operations.
