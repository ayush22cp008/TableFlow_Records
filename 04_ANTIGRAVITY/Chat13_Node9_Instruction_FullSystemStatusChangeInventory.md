# Instruction — Node 9: Full-System Status/State Change Inventory (INVESTIGATION ONLY, NO FIX)

## Task
Scan the entire TableFlow codebase and produce a complete inventory of every place a **status/state change** happens. This is scoping for the Notifications System (Node 9). Do NOT write any notification code, migrations, or fixes — investigation/report only.

## What to find
For every status/state transition in the system, note:
- **Entity** (order, reservation, staff/invite, table occupancy, etc.)
- **File + function/component** where the transition happens
- **Trigger role** — who causes it (owner / manager / waiter / cook / customer / system)
- **Old status → new status** values involved
- **Who would plausibly need to know** — customer, specific staff role, or owner (just note it, don't decide final behavior)

## Areas to cover (don't limit to only these — find anything missed too)
1. **Orders** — Order Placed → Preparing → Ready → Served → Billed, and Cancelled (all cancel entry points, including bulk "Emergency Stop")
2. **Reservations** — Pending → Approved / Rejected, and any other reservation status transitions
3. **Staff / Invite codes** — activation, deactivation, invite code use/expiry
4. **Table occupancy** — occupied/released transitions

## Also report
- Any existing notification-like mechanism already in the code (email sending, toasts, alerts, real-time subscriptions) — list what exists and where, even if partial/unused.
- Any status transition that happens in more than one code path (possible dual-source-of-truth risk).

## Output
Write findings to `03_Investigation_and_Errors/Chat13_Node9_Investigation_StatusChangeInventory.md` in the Drive bridge folder. Structured list/table format, grouped by entity. No fix suggestions — pure inventory.

## Constraints
- Read-only investigation. Do not modify any project source file.
- No terminal changes beyond grep/search/read operations.
