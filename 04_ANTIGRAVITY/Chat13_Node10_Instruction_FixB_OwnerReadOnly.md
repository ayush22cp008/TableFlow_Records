# Instruction — Fix B: Owner Read-Only (Orders, Billing, Tables, Waitlist)

**Scope:** ONLY these 4 areas below. Do NOT touch `reservation_requests` (Fix A — already done/locked) or Menu (Fix C — comes later, and is a REVERSE case: Owner keeps Menu access, Manager loses it — do not apply this same pattern there).

## Goal
Owner becomes read-only for day-to-day operations across Live Orders, Billing, Tables, and Waitlist. Only "Bulk Emergency Stop" stays with Owner (already a separate code path — do not touch it).

## 1. Live Orders — `app/dashboard/orders/page.tsx`
- Hide/disable for Owner: `advanceStatus` buttons (`→ preparing`, `→ ready`, `→ served`), plain `submitCancel` (Cancel button).
- Keep untouched: Bulk Emergency Stop (`submitBulkCancel` / `cancel_active_orders` RPC) — this stays available to Owner.
- RLS: modify/drop Owner's access in the `"owner_all_updates"` policy on `orders` (found in `20260804000002_node2b_cancellation.sql`) so Owner can no longer directly update `orders` via this path. Confirm the Bulk Emergency Stop RPC path is NOT affected (it should use its own permission path, e.g. `SECURITY DEFINER` on the RPC — verify before changing the table-level policy).

## 2. Bill page — `app/dashboard/billing/[orderId]/page.tsx`
- Block/hide this route for Owner entirely (Manager's flow is separate: `app/dashboard/manager/page.tsx` using `mark_order_paid` RPC — untouched).
- RLS: remove Owner's direct update access on `orders` and `restaurant_tables` for this billing path.

## 3. Tables page — `app/dashboard/tables/page.tsx`
- Hide/disable for Owner: `cycleTableStatus`, `setReservation`, `clearReservation` (Reserve/Clear/toggle buttons).
- RLS: remove Owner from `"tables_write"` policy on `restaurant_tables`.

## 4. Waitlist — same file `app/dashboard/tables/page.tsx`
- Hide/disable for Owner: `seatWaitlistEntry`, `cancelWaitlistEntry`.
- RLS: remove Owner from `"waitlist_update"` policy on `waitlist`.

## Pattern to follow (same as Fix A, already verified working)
Use the same `role === 'manager'` conditional-render gate proven in Fix A on `tables/page.tsx`. Reuse `useAuth`/`role` pattern already in that file.

## Steps
1. Write the RLS SQL changes for all 3 policies (`owner_all_updates` on orders, `tables_write`, `waitlist_update`) — model the manager-only scoping after the working Fix A policy (`has_role(ARRAY['manager'])`).
2. Apply frontend conditional rendering in the 3 files listed above.
3. Build and confirm no compile errors. Do NOT run local dev server test — Ayush tests via Vercel deploy after push.
4. Write the rollback SQL for all 3 policy changes (forward + revert, same format as Fix A).

## Rollback template (fill in actual policy names/definitions used)
```sql
-- Orders
DROP POLICY IF EXISTS "<new_orders_policy_name>" ON orders;
CREATE POLICY "owner_all_updates" ON orders ...; -- restore original definition

-- Tables
DROP POLICY IF EXISTS "<new_tables_policy_name>" ON restaurant_tables;
CREATE POLICY "tables_write" ON restaurant_tables ...; -- restore original definition

-- Waitlist
DROP POLICY IF EXISTS "<new_waitlist_policy_name>" ON waitlist;
CREATE POLICY "waitlist_update" ON waitlist ...; -- restore original definition
```
Antigravity: capture the exact original policy definitions before dropping them (e.g. via `SELECT * FROM pg_policies WHERE ...` or from the known migration file) so the rollback is accurate, not a guess.

## After Antigravity finishes
Report back in `03_Investigation_and_Errors/Chat13_Node10_FixB_Result.md`:
- Exact SQL used per policy (forward + rollback, with real names filled in)
- Confirmation Bulk Emergency Stop path was verified unaffected
- Frontend changes confirmed scoped correctly (3 files)
- Build result

Do NOT push to GitHub yet — wait for Ayush's explicit go-ahead after he reviews.
