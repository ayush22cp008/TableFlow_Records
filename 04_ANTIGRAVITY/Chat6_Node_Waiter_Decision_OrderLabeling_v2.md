# Waiter Dashboard — Order Labeling + Priority Sort Decision (Chat 6, v2)

**Supersedes:** `Chat6_Node_Waiter_Decision_OrderLabeling.md` (v1) — v1 only covered labeling, this adds priority sorting. Ignore v1, use this file. (v1 left in place per no-delete rule — do not action it.)

**Scope:** Waiter Dashboard only (not Manager, not Customer — those are separate future decisions, deferred)

## Decision 1 — Order Labeling

Waiter Dashboard order cards must show **both**:
1. The existing daily-reset sequence label (`W1, W2, W3...` for walk-in, `R1, R2, R3...` for reservation orders) — same `daily_order_counters` system already implemented and used in Owner/Cook views. No new numbering system to be built.
2. The real table number (e.g. `Table 2`)

Displayed together, e.g.: `W3 · Table 2`

**Reason:** Waiter needs to know which table an order belongs to. Customer name/email cannot be used (privacy; duplicate names possible). Reuse the existing per-day-reset `W#`/`R#` system rather than inventing a new identifier.

## Decision 2 — Priority Sorting (NEW in v2)

Waiter Dashboard's order queue (the "Ready" column Waiter acts on, i.e. Ready→Served) must sort **R (reservation/priority) orders above W (walk-in) orders**, regardless of which arrived first or what time they arrived.

- Same rule already correctly implemented in Owner's Live Orders and Cook's KDS — reuse that exact sorting logic, do not write a new sort.
- This is a hard rule, not time-dependent: even if a W order arrived earlier and an R order arrived later, R still sorts above W.
- **Known bug to avoid:** Manager Dashboard currently does NOT apply this correctly (Intake Queue and Billing Queue both sort wrong — see `Chat6_Node_Manager_Bugs_PriorityDeferred.md`, fix deferred to a separate node). Waiter Dashboard must not repeat this mistake — build it correctly from the start using the same sort logic as Owner/Cook, not by copying Manager's queue-fetch code.

## Explicitly deferred (not in this scope)

- Manager Dashboard W#/R# labeling AND priority-sort bugs — separate future task/node (see `Chat6_Node_Manager_Bugs_PriorityDeferred.md`).
- Customer-side order views (labeling) — deferred.

## Note for whoever builds Waiter Dashboard

When Waiter Dashboard is built (next task after `AuthForm.tsx` waiter routing branch), include BOTH the `W#/R#` + Table labeling AND the priority-first sort from the start — don't build it plain and retrofit later. Reference Owner/Cook's existing implementation for both the counter system and the sort logic rather than reinventing either.
