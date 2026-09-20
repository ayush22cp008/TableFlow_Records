# TableFlow — Chat 18 — Node 13 Investigation Result Report

**Node:** 13 — Cook/Waiter Order Claiming System
**Chat:** 18
**Type:** Investigation Result Report
**Status:** INVESTIGATION COMPLETE — Ready for Design/Spec Stage
**Date:** 2026-09-20
**Repository Investigated:** https://github.com/ayush22cp008/TableFlow (main branch)

---

## 1. Executive Summary

No order claiming system exists in the current TableFlow source. Cook and Waiter workflows are fully unguarded: any Cook with `role = cook` and `is_active = true` can mark any `preparing` order as `ready`; any Waiter can mark any `ready` order as `served`. There is no `assigned_cook_id`, `assigned_waiter_id`, atomic claim RPC, or race-condition protection of any kind. The database schema contains no claim or assignment column on the `orders` table. The entire claiming mechanism — DB columns, RPC, RLS, UI — must be built from scratch as part of Node 13.

---

## 2. Cook Current State

**File:** `app/dashboard/cook/page.tsx` — `CookDashboardPage()`

- **Visible statuses:** Only `status = 'preparing'`. Orders in `placed`, `ready`, `served`, `billed`, `cancelled` are not visible to Cook.
- **Query:** `supabase.from('orders').select('*, order_items(quantity, menu_items(name))').eq('status', 'preparing').order('is_priority', { ascending: false }).order('created_at', { ascending: true })`
- **Status transition:** Cook presses "Mark Ready" → `status: 'ready'` via direct `.update()` on the `orders` table (no RPC). Realtime subscription then fires to refresh the list.
- **Realtime:** `supabase.channel('cook_orders_realtime').on('postgres_changes', { event: '*', schema: 'public', table: 'orders' }, fetchPreparingOrders)` — all events on `orders` trigger a full refetch.
- **Multi-Cook race:** UNGUARDED. If two Cooks both see order X at `preparing`, both can press "Mark Ready" simultaneously. The second `.update()` call will still succeed (RLS `WITH CHECK` only verifies `status = 'ready'` on the outgoing row, not atomicity). No claim or lock exists.
- **Auth/role check:** Enforced by Supabase RLS (`has_role(ARRAY['cook']) AND status = 'preparing'` on SELECT and `WITH CHECK` on UPDATE). `has_role()` verifies `is_active = true` (see `20260809000007_enforce_active_role.sql`).
- **Existing assignment field:** NONE. No `claimed_by`, `assigned_cook_id`, or equivalent column on `orders`.

---

## 3. Waiter Current State

**File:** `app/dashboard/waiter/page.tsx` — `WaiterDashboardPage()`

- **Visible statuses:** Only `status = 'ready'`.
- **Query:** `supabase.from('orders').select('*, order_items(quantity, menu_items(name)), restaurant_tables(table_number)').eq('status', 'ready').order('is_priority', { ascending: false }).order('created_at', { ascending: true })`
- **Status transition:** Waiter presses "Mark Served" → `status: 'served'` via direct `.update()`. Realtime fires to refresh.
- **Realtime:** `supabase.channel('waiter_orders_realtime').on('postgres_changes', { event: '*', schema: 'public', table: 'orders' }, fetchReadyOrders)` — all events trigger full refetch.
- **Multi-Waiter race:** UNGUARDED. Same as Cook: two Waiters can simultaneously mark the same `ready` order as `served`. No claim or lock.
- **Auth/role check:** RLS `has_role(ARRAY['waiter']) AND status = 'ready'` on USING and `WITH CHECK (status = 'served')`.
- **Existing assignment field:** NONE.

---

## 4. Manager Current State

**File:** `app/dashboard/manager/page.tsx` — (full component, truncated in output but verified)

- **Visible statuses:** `placed` (Intake Queue) and `served` (Billing Queue). Does not show `preparing`, `ready`, `billed`, `cancelled` in the main UI.
- **Status transitions:**
  - `placed → preparing` via `markPreparing(order.id)` — direct `.update({ status: 'preparing' })`.
  - `served → billed` via `markPaid(order.id)` — calls RPC `mark_order_paid(p_order_id, p_payment_method)`.
- **Staff/profile info near orders:** No staff name or ID displayed on order cards. No `assigned_cook_id` or `assigned_waiter_id` in the query.
- **Realtime:** VERIFIED present (from prior Node 9 work and realtime channel investigations).
- **Manager RLS:** `has_role(ARRAY['manager']) AND status IN ('placed', 'served')` on USING; `WITH CHECK (status IN ('preparing', 'billed', 'cancelled'))`.

---

## 5. Database / Schema Facts

**Source:** `supabase/migrations/20260804000001_node2b_schema_rls.sql`, `20260809000001_mark_order_paid_rpc.sql`, `20260809000003_place_order_capacity_guard.sql`

- **`assigned_cook_id`:** DOES NOT EXIST in any migration file. Confirmed by `Select-String` across all migrations — zero matches.
- **`assigned_waiter_id`:** DOES NOT EXIST. Same confirmation.
- **Separate claim/assignment table:** DOES NOT EXIST. No migration creates such a table.
- **`orders` confirmed columns (from RPC and SELECT queries):** `id`, `customer_id`, `table_id`, `subtotal`, `total`, `status`, `party_size`, `is_priority`, `daily_number`, `created_at`, `updated_at`, `payment_method`.
- **Existing order-mutation RPCs:**
  - `place_order_and_occupy_table(p_customer_id, p_table_id, p_subtotal, p_total, p_party_size, p_is_priority)` — INSERT + table occupancy, atomic via RPC. Source: `20260809000003_place_order_capacity_guard.sql`.
  - `mark_order_paid(p_order_id, p_payment_method)` — UPDATE to `billed` + table seat release. Source: `20260809000001_mark_order_paid_rpc.sql`.
- **No atomic claim RPC exists.** Cook and Waiter status transitions are bare `.update()` calls from the client.
- **Foreign keys to profiles:** `orders.customer_id → profiles.id` (confirmed). No FK to profiles for staff identity.

---

## 6. RLS / Permissions Facts

**Source:** `supabase/migrations/20260804000001_node2b_schema_rls.sql`, `20260809000007_enforce_active_role.sql`

| Role | SELECT | UPDATE (USING) | UPDATE (WITH CHECK) |
|---|---|---|---|
| Cook | All orders | `has_role(['cook']) AND status = 'preparing'` | `has_role(['cook']) AND status = 'ready'` |
| Waiter | All orders | `has_role(['waiter']) AND status = 'ready'` | `has_role(['waiter']) AND status = 'served'` |
| Manager | All orders | `has_role(['manager']) AND status IN ('placed','served')` | `has_role(['manager']) AND status IN ('preparing','billed','cancelled')` |
| Owner | All orders | `has_role(['owner'])` | `has_role(['owner'])` |
| Customer | Own orders only | — | — |

- **`has_role()` function** (source: `20260809000007_enforce_active_role.sql`): checks `role = ANY(allowed_roles) AND is_active = true`. Deactivated staff are immediately blocked from RLS access — no claim reassignment mechanism exists for in-flight orders.
- **No claim-specific RLS policy** exists anywhere in the migration history.

---

## 7. Race-Condition Facts

- **Cook race:** Two Cooks, both seeing order X at `preparing`, press "Mark Ready" simultaneously. Both `.update({ status: 'ready' }).eq('id', order.id)` calls pass RLS USING check (both have `role = cook, is_active = true, order.status = 'preparing'`). The second UPDATE is idempotent on status (already `ready`) but will still execute without error. No duplicate action harm at status level, but both Cooks believe they handled the order — there is no "winner" established.
- **Waiter race:** Identical pattern for `ready → served`. Two Waiters can both mark the same order served. Again idempotent on status field, but no ownership is recorded.
- **No DB-level protection exists:** No `FOR UPDATE` locking, no `unique` constraint on a claim column, no RPC enforcing atomic check-then-set.
- **PostgreSQL capability available:** `UPDATE ... WHERE status = 'preparing' AND claimed_by IS NULL RETURNING id` (conditional UPDATE + affected-row check) is a standard atomic pattern available without additional extensions. `FOR UPDATE SKIP LOCKED` is also available. Neither is currently used.

---

## 8. One-Order-at-a-Time Facts

- **No enforcement exists.** A Cook can press "Mark Ready" on any number of `preparing` orders in quick succession — there is no active-claim counter or lock.
- **Data required to enforce it:** A `claimed_by_cook_id` column on `orders` (or equivalent) that is SET when Cook claims and CLEARED when Cook marks ready (or order is cancelled). A check `WHERE claimed_by_cook_id IS NULL` on the claim RPC would enforce one-at-a-time if combined with a `UNIQUE (claimed_by_cook_id)` partial index (excluding NULL).
- **Cook release states (current):** Order transitions to `ready` or `cancelled` — both would release the Cook. These states exist in the current status constraint.
- **Waiter lifecycle equivalent:** Same pattern. `claimed_by_waiter_id` set on claim, cleared on `served` or `cancelled`.

---

## 9. Reassignment / Deactivation Facts

**Source:** `20260809000005_staff_management_updates.sql`, `20260809000007_enforce_active_role.sql`

- **Deactivation mechanism:** `profiles.is_active` set to `false`. `has_role()` immediately returns `false` → all RLS policies block access. Deactivated Cook/Waiter cannot SELECT or UPDATE orders.
- **In-flight order behavior on deactivation:** UNKNOWN. If Cook X claimed order Y and is deactivated mid-cook, order Y stays at `preparing` with no reassignment. No recovery, reassignment, or abandoned-work mechanism exists in any migration.
- **Force logout:** `force_logout_all_staff()` RPC deletes sessions for all staff (`waiter`, `cook`, `manager`). Does not address in-flight claimed orders.
- **Existing reassignment mechanism:** NONE.

---

## 10. Candidate Claim Insertion Points

| Event | Current file | Current action | Claim insertion point |
|---|---|---|---|
| Cook claims order | None — no claim concept | N/A | New: RPC `claim_order_as_cook(p_order_id)` |
| Cook marks ready | `app/dashboard/cook/page.tsx`, `markReady()` | `.update({ status: 'ready' })` | Modify to also clear `claimed_by_cook_id` |
| Waiter claims order | None | N/A | New: RPC `claim_order_as_waiter(p_order_id)` |
| Waiter marks served | `app/dashboard/waiter/page.tsx`, `markServed()` | `.update({ status: 'served' })` | Modify to also clear `claimed_by_waiter_id` |
| Manager notification | `app/dashboard/manager/page.tsx` | Does not show claimant | Future: notification trigger fires `order_claimed` type (not in scope yet) |

---

## 11. Confirmed Facts vs UNKNOWN

### CONFIRMED
- No `assigned_cook_id`, `assigned_waiter_id`, or claim table exists anywhere in the codebase.
- Cook sees only `preparing` orders; Waiter sees only `ready` orders.
- Both transitions (`preparing→ready`, `ready→served`) are unguarded bare `.update()` calls.
- `has_role()` enforces `is_active = true` — deactivated staff lose RLS access immediately.
- Two existing atomic RPCs exist (`place_order_and_occupy_table`, `mark_order_paid`) proving the team's RPC pattern is established and usable as a template.
- PostgreSQL supports atomic conditional UPDATE (`WHERE claimed_by IS NULL RETURNING id`) without extensions.

### UNKNOWN
- Whether Supabase's hosted Postgres version supports `FOR UPDATE SKIP LOCKED` (likely yes, but not verified against actual Supabase project version).
- Exact behavior if a Cook is deactivated mid-claim — no recovery path exists and none is documented.
- Whether Manager should see claimant name alongside order during Node 13 (not in current UI; would require schema + UI change).

---

## 12. Node 13 Design Inputs for Next Stage

The following facts are confirmed inputs for the Node 13 spec:

1. **Two new columns needed:** `claimed_by_cook_id uuid REFERENCES profiles(id)` and `claimed_by_waiter_id uuid REFERENCES profiles(id)` on `orders` table.
2. **Two new atomic RPCs needed:** `claim_order_as_cook(p_order_id)` and `claim_order_as_waiter(p_order_id)` — must use conditional UPDATE pattern to prevent race conditions.
3. **RLS must be extended** to expose `claimed_by_*` columns to Cook/Waiter/Manager SELECT without granting direct UPDATE on those columns (UPDATE goes through RPCs only).
4. **One-order-at-a-time:** Enforced via a partial unique index `(claimed_by_cook_id) WHERE claimed_by_cook_id IS NOT NULL` — prevents a Cook claiming two orders simultaneously at the DB level.
5. **UI changes:** Cook dashboard needs a "Claim" button (before "Mark Ready"); Waiter dashboard needs a "Claim" button (before "Mark Served"). Claimed orders should be visually distinct (e.g., dimmed or hidden for other staff).
6. **Abandoned claim handling:** No existing mechanism — Node 13 spec must define timeout/reassignment behavior or explicitly defer it.
7. **Manager notification insertion point:** The RPC `claim_order_as_cook` / `claim_order_as_waiter` is the natural trigger point for a future `order_claimed` notification type (Node 12 / future work, not Node 13 scope).
8. **RPC template:** Follow the same `SECURITY DEFINER` + `LANGUAGE plpgsql` pattern established by `mark_order_paid` and `place_order_and_occupy_table`.