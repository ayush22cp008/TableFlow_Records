# Instruction — Node 9: Full-System Role-Wise Change Inventory (INVESTIGATION ONLY, NO FIX)

## Task
Scan the entire TableFlow codebase and produce a complete inventory of every **meaningful UI or database change** in the system — not just status field transitions. This is scoping for the Notifications System (Node 9). Do NOT write any notification code, migrations, or fixes — investigation/report only.

## Scope clarification (broader than just "status")
Include ANY event where something important changes and a role (owner, manager, waiter, cook, customer) plausibly should be informed — even if it's not a `status` column update. Examples of what counts:
- Status/state transitions (order, reservation, staff/invite, table occupancy)
- New record creation (new order placed, new reservation request, new staff invite used)
- Any DB write that changes what another role sees on their dashboard
- Any UI action one role takes that visibly affects another role's screen
- Anything currently silent (no toast/alert/email) where a role would only find out by manually refreshing/checking

## What to find
For every such event, note:
- **Entity** (order, reservation, staff/invite, table occupancy, or other)
- **What changes** — UI, DB, or both; old value → new value if applicable
- **File + function/component** where it happens
- **Trigger role** — who causes it (owner / manager / waiter / cook / customer / system)
- **Affected role(s)** — who would plausibly need to know (just note it, don't decide final notification behavior)
- **Currently silent or already surfaced?** — does the affected role find out only by manually checking/refreshing, or is there already some signal (toast, badge, email)?

## Areas to cover (don't limit to only these — find anything missed too)
1. **Orders** — Order Placed → Preparing → Ready → Served → Billed, Cancelled (all entry points incl. bulk "Emergency Stop")
2. **Reservations** — new request, Pending → Approved/Rejected, any other transitions
3. **Staff / Invite codes** — activation, deactivation, invite code use/expiry
4. **Table occupancy** — occupied/released transitions
5. **Anything else found** — any other role-to-role visible change not covered above (e.g. manager reassigning something, owner editing menu/pricing that staff should know about, etc.) — flag it even if unsure it needs a notification.

## Also report
- Any existing notification-like mechanism already in the code (email sending, toasts, alerts, badges, real-time subscriptions) — list what exists and where, even if partial/unused.
- Any change that happens in more than one code path (possible dual-source-of-truth risk).

## Output
Write findings to `03_Investigation_and_Errors/Chat13_Node9_Investigation_RoleWiseChangeInventory.md` in the Drive bridge folder. Structured list/table format, grouped by entity. No fix or notification-design suggestions — pure inventory.

## Constraints
- Read-only investigation. Do not modify any project source file.
- No terminal changes beyond grep/search/read operations.
