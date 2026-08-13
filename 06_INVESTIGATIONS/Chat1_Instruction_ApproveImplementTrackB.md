# Instruction — Approve & Implement Track B

Chat #1 | Node: Track B Implementation

## Approval

Proposed plan approved as submitted:
- `daily_order_counters` table (`counter_date`, `priority_count`, `walkin_count`) with atomic `INSERT ... ON CONFLICT DO UPDATE` for sequence generation.
- `place_order_and_occupy_table` RPC updated to accept `p_is_priority boolean` and assign `daily_number` atomically.
- `is_priority` + `daily_number` added to `orders` table and `Order` type.
- Cart passes `p_is_priority: !!verifiedReservation`.
- Orders dashboard sorts `is_priority DESC, created_at ASC`; displays R#/W# and priority badge in all 4 columns.

## Daily Reset Clarification (confirmed)

`counter_date` naturally creates a new row per calendar day — old dates' rows are NOT deleted, they remain as historical record. No cleanup/archival job needed.

## Task

Implement exactly as proposed. Work on an isolated git branch.

## Verification Plan (as proposed, please execute)

1. Run `npm run build` — confirm 0 TypeScript errors.
2. Provide the manual test script/steps to inject one reservation order + one walk-in order, confirm both get assigned `1` (i.e. R1 and W1).
3. Report expected result for viewing `/dashboard/orders` — confirm R1 should sort above W1 in each column regardless of placement order.

## Do Not
- Do not merge/push to main yet — wait for Ayush's go-ahead after build verification, per existing push protocol.
- Do not touch Track A / Auto-Hide logic — already locked and verified.
