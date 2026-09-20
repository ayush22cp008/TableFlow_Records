# Goal: Implement Node 13 (Cook/Waiter Order Claiming)

**Status:** REVIEWED — REQUIRED CHANGES APPLIED — APPROVED FOR CODING
**Supersedes:** Chat18_Node13_Implementation_revised_plan.md
**Reviewed against:** live TableFlow source (https://github.com/ayush22cp008/TableFlow), 2026-09-20

This implementation plan strictly follows the Node 13 specification to introduce exclusive order claiming for Cooks and Waiters. It enforces one-order-at-a-time rules, atomic concurrency, and prevents any un-claimed direct status updates.

**Deferred (optional, not included in this plan):** restricting `cook_cancel` / `waiter_cancel` RLS policies to claimant-only. Any Cook can still cancel any active order regardless of who claimed it, and same for Waiter — this is unchanged from current behavior. This does not break the claim system's correctness (one-order-at-a-time and race protection still hold); it's a scope decision, not a defect. Revisit only if it becomes an actual operational problem.

## Proposed Changes

### Database Migration

#### [NEW] [20260920000001_node13_order_claiming.sql](file:///C:/Users/ayush/Desktop/vibethon_project/supabase/migrations/20260920000001_node13_order_claiming.sql)

- **Schema changes:**
  - Add `claimed_by_cook_id` and `claimed_by_waiter_id` to `orders`, referencing `profiles(id)` with `ON DELETE SET NULL`.
  - Add `staff_name` to `profiles`.
- **Indexes (One-order-at-a-time):** Create partial unique indexes on `claimed_by_cook_id` and `claimed_by_waiter_id` (where NOT NULL) to enforce one active claim per staff member.
- **Claim-Invariant Cleanup Trigger:** Add a `BEFORE UPDATE` trigger on `orders`. This trigger will strictly enforce cleanup rules:
  - If status transitions to `ready` or `cancelled`, automatically set `claimed_by_cook_id = NULL`.
  - If status transitions to `served` or `cancelled`, automatically set `claimed_by_waiter_id = NULL`.
- **Atomic Claim/Completion RPCs:** Create 4 atomic RPCs (`claim_order_as_cook`, `complete_order_as_cook`, `claim_order_as_waiter`, `complete_order_as_waiter`) with strict `FOR UPDATE` locking on both the `profiles` row and the `orders` row to serialize concurrent attempts.
- **SECURITY DEFINER Hardening:**
  - All 4 RPCs will strictly use `SECURITY DEFINER`.
  - Enforce `SET search_path = public, pg_temp`.
  - Explicitly `REVOKE ALL ON FUNCTION ... FROM PUBLIC` and `GRANT EXECUTE ON FUNCTION ... TO authenticated`.
  - Internally verify `auth.uid()`, exact `role`, and `is_active = true` against the `profiles` table.
- **Manager Resolution:** In the claim RPCs, retrieving the active Manager for the `order_claimed` notification will strictly count active Managers. If there are `0` or multiple (`> 1`) active Managers, the RPC will explicitly throw an operational configuration error rather than using an arbitrary `LIMIT 1`.
- **RLS Adjustments (Close Bypass):** Drop the existing `cook_prep_to_ready` and `waiter_ready_to_served` policies to ensure status updates only happen through the completion RPCs. Cancellation policies (`cook_cancel`, `waiter_cancel`, `manager_cancel`) remain untouched — see deferred item above.
- **[REQUIRED — NEW] Manager Profiles Visibility Policy:**
  - `profiles` currently has no SELECT policy granting Manager visibility into staff profiles — verified live: only `profiles_own_read` (`id = auth.uid()`) and `profiles_owner_read` (`is_owner()`) exist. Without this, the Manager dashboard's join to `profiles` for claimant name/email will silently return `null` under RLS — no error, just empty data.
  - Add: `CREATE POLICY "profiles_manager_read" ON profiles FOR SELECT USING (has_role(ARRAY['manager']) AND role IN ('cook','waiter'));`
  - This must ship in the same migration as the claim columns, since the Manager UI work depends on it to be testable at all.

---

### Pre-Migration Checklist (manual, before running the migration)

- **[REQUIRED] Confirm exactly one active Manager exists**: run `SELECT count(*) FROM profiles WHERE role = 'manager' AND is_active = true;` in Supabase SQL Editor. Must return exactly `1`. The claim RPCs' Manager-notification resolution throws an exception on 0 or >1 — if the count is wrong, fix the data before running the migration, not after.

---

### Backend API

#### [MODIFY] [route.ts](file:///C:/Users/ayush/Desktop/vibethon_project/app/api/auth/staff-signup/route.ts)
- Persist `staff_name: codeData.staff_name` during both the re-activation (existing profile) branch and new user signup branch.

---

### Types

#### [MODIFY] [index.ts](file:///C:/Users/ayush/Desktop/vibethon_project/types/index.ts)
- Add `order_claimed` to `NotificationType`.
- Update `Order` type to include `claimed_by_cook_id`, `claimed_by_waiter_id`.
- Add `staff_name` to `UserProfile`. *(corrected from "Profile" — no `Profile` type exists in the current source; the actual type is `UserProfile`.)*

---

### Frontend UI

#### [MODIFY] [page.tsx (Cook)](file:///C:/Users/ayush/Desktop/vibethon_project/app/dashboard/cook/page.tsx)
- Update query to select `claimed_by_cook_id`.
- Show a "Claim" button for unclaimed orders.
- Show "Mark Ready" only if `claimed_by_cook_id == userId`.
- Grey out orders claimed by other Cooks.
- Use RPCs `claim_order_as_cook` and `complete_order_as_cook`.

#### [MODIFY] [page.tsx (Waiter)](file:///C:/Users/ayush/Desktop/vibethon_project/app/dashboard/waiter/page.tsx)
- Update query to select `claimed_by_waiter_id`.
- Use RPCs `claim_order_as_waiter` and `complete_order_as_waiter`.

#### [MODIFY] [page.tsx (Manager)](file:///C:/Users/ayush/Desktop/vibethon_project/app/dashboard/manager/page.tsx)
- Update the Supabase select query to fetch assigned staff names using PostgREST relation syntax disambiguation for the multiple foreign keys to `profiles`. (e.g., `cook:profiles!orders_claimed_by_cook_id_fkey(staff_name, email), waiter:profiles!orders_claimed_by_waiter_id_fkey(staff_name, email)`).
- Display the assigned Cook/Waiter name and email on order cards.
- Depends on the new `profiles_manager_read` RLS policy above — verify this policy is live before testing this UI, or the join will return empty.

## Verification Plan

### Automated Checks
- Run `npm run build` and `tsc --noEmit` to verify type safety and component builds.

### Manual Verification Coverage
- **Concurrency Race:** 2 instances trying to claim the same order simultaneously — verify exactly one wins (via `FOR UPDATE` target row lock).
- **One-Order-At-A-Time:** Cook tries claiming two orders — verify rejection (via partial unique index and `FOR UPDATE` profile lock).
- **Direct-Update Bypass:** Attempt direct `supabase.from('orders').update({status: 'ready'})` as Cook — verify RLS rejection.
- **Cancellation Race:** Manager cancels an order during an active claim — verify the BEFORE UPDATE claim-invariant trigger cleanly sets claim columns to NULL.
- **Profile Deletion/Deactivation:** Delete or deactivate a claimed Cook — verify FK `ON DELETE SET NULL` fires or `is_active` blocks further actions, leaving the order claimable.
- **Manager Notification:** Verify `order_claimed` notification is inserted accurately without arbitrary `LIMIT 1` assignment.
- **Manager Visibility:** Verify Manager dashboard actually displays claimant name/email (not blank/null) — this is the direct test of the new `profiles_manager_read` policy.
- **Realtime:** Ensure UI updates synchronously across KDS and Waiter Dashboards.
- **Node 9 Regression:** Ensure existing Node 9 `order_ready`, `order_served`, and `order_cancelled` notifications still fire correctly from the status-change triggers.
