# Instruction — Node 11 Batch 1: Realtime Auto-Update (Manager, Cook, Waiter Dashboards)

**Type:** Fix. Implement code changes as specified below.

## Context

Coverage audit confirmed these 3 operational dashboards currently require manual refresh to see new/updated data. Existing realtime pattern already works correctly elsewhere in the codebase (e.g. `orders_board` in `app/dashboard/orders/page.tsx`, `tables_realtime` in `app/dashboard/tables/page.tsx`) — follow the same pattern for consistency.

## Scope — 3 files only

1. `app/dashboard/manager/page.tsx`
2. `app/dashboard/cook/page.tsx`
3. `app/dashboard/waiter/page.tsx`

## What to do

For each of the 3 files:

- Add a Supabase Realtime subscription using `postgres_changes`, following the exact pattern used in `app/dashboard/orders/page.tsx` (channel setup, subscribe, cleanup on unmount).
- Subscribe to the `orders` table (all 3 dashboards care about order state changes — Manager for new/placed orders, Cook for preparing, Waiter for ready-to-serve).
- On any relevant event (INSERT/UPDATE — determine which events actually matter for each role's view based on what data they display), update local component state so the UI reflects the change without a manual refresh.
- Give each channel a distinct name to avoid collisions with existing channels, e.g. `manager_orders_realtime`, `cook_orders_realtime`, `waiter_orders_realtime`.
- Remove/replace the existing "manual refresh" button-triggered fetch logic only if it becomes fully redundant — otherwise leave the manual refresh button in place as a fallback (do not remove functionality, only add auto-update on top).
- Do not change any business logic, role permissions, or UI layout beyond what's needed to wire up the subscription.

## Constraints (per project rules)

- No DB/schema changes — this is frontend-only (subscription wiring), same as existing realtime pages.
- No RLS changes.
- Do not touch any other files outside the 3 listed above.
- Do not touch Node 10's locked role-permission logic — only add data-sync, don't alter what each role can see/do.
- After implementation: report files modified, confirm build passes (`npm run build` or equivalent), and confirm no unexpected files were touched (per preflight/postflight rule).

## Testing note

Ayush will manually verify in-browser: opening two sessions (e.g. Owner action + Manager/Cook/Waiter dashboard open) and confirming the second updates without refresh. Do not claim UI verification — that's Ayush's step.

## Output

Report back with: files modified, build result, and the exact channel names used for each of the 3 dashboards.
