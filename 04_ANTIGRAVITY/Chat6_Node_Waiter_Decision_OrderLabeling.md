# Waiter Dashboard — Order Labeling Decision (Chat 6, pre-Chat 7)

**Scope:** Waiter Dashboard only (not Manager, not Customer — those are separate future decisions, deferred)

## Decision

Waiter Dashboard order cards must show **both**:
1. The existing daily-reset sequence label (`W1, W2, W3...` for walk-in, `R1, R2, R3...` for reservation orders) — same `daily_order_counters` system already implemented and used in Owner/Cook views. No new numbering system to be built.
2. The real table number (e.g. `Table 2`)

Displayed together, e.g.: `W3 · Table 2`

**Reason:** Waiter needs to know which table an order belongs to. Customer name/email cannot be used (privacy; duplicate names possible). Reuse the existing per-day-reset `W#`/`R#` system rather than inventing a new identifier.

## Explicitly deferred (not in this scope)

- Manager Dashboard currently shows only real table number (no `W#`/`R#`) — retrofitting this is a separate future task, not part of Waiter Dashboard build.
- Customer-side order views currently show only real table number — same, deferred.

## Note for whoever builds Waiter Dashboard

When Waiter Dashboard is built (next task after `AuthForm.tsx` waiter routing branch), include this labeling from the start — don't build it without `W#`/`R#` and retrofit later.
