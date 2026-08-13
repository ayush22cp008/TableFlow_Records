# Manager Dashboard — 3 Bugs Found (Chat 6, deferred to post-Waiter node)

**Scope:** Fix AFTER Waiter Dashboard is complete — separate future chat/node. Not urgent, just noted here so evidence isn't lost.

## Bug 1 — Intake Queue: priority orders not sorted first

Manager Dashboard's "Intake Queue (Placed)" shows orders in wrong order. A `W` (walk-in) order appeared above an `R` (reservation/priority) order.

**Expected (per existing locked rule):** R-priority orders always sort above W orders in every column — same rule already working correctly in Owner/Cook views (Live Orders, KDS). Manager Dashboard's Intake Queue does not follow this rule.

**Evidence:** Screenshot shows `Order #62e951` (Table 1, walk-in) listed above `Order #46f247` (Table 4, which was later confirmed to be the `R1 PRIORITY` order — same order visible with priority badge in Owner/Cook views at the same timestamp).

## Bug 2 — Billing Queue: sorted by amount, not priority

Manager Dashboard's "Billing Queue (Served)" is sorting by ascending bill amount ($160 before $215) instead of priority-first.

**Expected:** Same priority-first rule as Bug 1 — R orders above W orders, regardless of bill amount.

**Evidence:** Screenshot shows Order #46f24705 (Table 4, $160) listed before Order #9b6812d6 (Table 5, $215) in Billing Queue — this happens to be walk-in-then-reservation or similar amount-based ordering, inconsistent with Owner's "Served" column at the same time, which correctly shows R1 (priority) above W7.

## Bug 3 — Mark Paid does not release table occupancy

After clicking "Mark Paid" on the Manager Dashboard billing card for an order tied to Table 1, the Tables page (`/dashboard/tables`) still shows Table 1 as "Full" / "2/2 seated" instead of releasing back to "Available".

**Evidence:** 
- Screenshot of Manager Dashboard shows Order #62e951df (Table 1) billed and "Mark Paid" clicked/available.
- Screenshot of `/dashboard/tables` taken after this shows Table 1 still "Full", "2/2 seated" — not reset.

**Root cause not yet investigated** — likely the Mark Paid handler on Manager Dashboard doesn't call the same table-release logic that other billing-closure paths use (if any exists). Needs investigation before fix (per engineering discipline rule: investigation and fix are separate prompts).

## Next Action (when this node starts)

1. Investigate Bug 1 & 2 together (likely same root cause — Intake/Billing queue fetch logic missing the priority-sort clause that Live Orders/KDS have).
2. Investigate Bug 3 separately (likely a missing table-release call in the Mark Paid handler).
3. Do NOT mix investigation and fix in the same prompt per standing rule.
