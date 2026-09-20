# TableFlow — Chat 18 — Node 13 Final Design v1.0

**Node:** 13 — Cook/Waiter Order Claiming System  
**Chat:** 18  
**Status:** DESIGN v1.0 — READY FOR SPEC/IMPLEMENTATION REVIEW  
**Implementation:** NOT STARTED  
**Source repository:** https://github.com/ayush22cp008/TableFlow  
**Reviewed against:** Chat18 Node 13 investigation + Claude design review + current source verification on 2026-09-20

## 1. Design Decision

**Result: FITS WITH REQUIRED CHANGES.**

The two-column claim model is retained. The final design adds explicit database security, atomic concurrency control, ownership enforcement, claim-release safeguards, and durable staff identity.

## 2. Current-System Facts That Drive the Design

- Cook currently receives/handles `preparing` orders in `app/dashboard/cook/page.tsx`.
- Waiter currently receives/handles `ready` orders in `app/dashboard/waiter/page.tsx`.
- Current Cook and Waiter completion actions are direct client `.update()` calls and are not ownership-aware.
- Current Node 2b RLS policies allow Cook/Waiter direct status transitions; these are the primary bypass paths that Node 13 must close.
- `orders` currently has no Cook/Waiter claim fields or assignment table.
- Node 9 is already implemented in the current source, including `notifications.recipient_id`, notification RLS, triggers, and Realtime wiring. Older records describing Node 9 as unimplemented are stale.
- Existing Node 9 status triggers should remain responsible for the normal `preparing → ready` and `ready → served` notifications.

## 3. Data Model

Add to `orders`:

```sql
claimed_by_cook_id uuid NULL REFERENCES profiles(id) ON DELETE SET NULL,
claimed_by_waiter_id uuid NULL REFERENCES profiles(id) ON DELETE SET NULL
```

**Reason for `ON DELETE SET NULL`:** the current staff deactivation route hard-deletes the staff `profiles` row. A claimed order must therefore automatically become unclaimed when the claimant profile is deleted.

No separate assignment table is required for Node 13 v1.

## 4. One-Order-at-a-Time Enforcement

Create partial unique indexes:

```sql
UNIQUE (claimed_by_cook_id) WHERE claimed_by_cook_id IS NOT NULL
UNIQUE (claimed_by_waiter_id) WHERE claimed_by_waiter_id IS NOT NULL
```

These are the database backstop for the rule:
- one active Cook claim per Cook
- one active Waiter claim per Waiter.

The RPC must also check for an existing active claim so the user receives a deterministic application error rather than relying only on a constraint violation.

## 5. Claim RPCs

Create:
- `claim_order_as_cook(p_order_id uuid)`
- `claim_order_as_waiter(p_order_id uuid)`

Each RPC must be `SECURITY DEFINER` with safe execution settings:

- `SET search_path = public, pg_temp`.
- Schema-qualify project tables/functions.
- Explicitly verify `auth.uid()` is not null.
- Explicitly verify caller role.
- Explicitly verify caller `is_active = true`.
- Explicitly verify target status.
- Explicitly verify the corresponding claim field is NULL.
- Explicitly verify the caller has no existing active claim.
- Revoke execute from `PUBLIC` and grant execute only to `authenticated`.

### Concurrency pattern

The RPC must serialize both sides of the race:

1. Lock the caller's `profiles` row with `FOR UPDATE` so two simultaneous claims by the same worker cannot both pass the one-order check.
2. Lock the target `orders` row with `FOR UPDATE`.
3. Re-check status and claim ownership after the locks are acquired.
4. Perform the claim update.
5. Insert the successful Manager notification in the same transaction.

Two different workers racing for the same order serialize on the order row; the first successful transaction establishes the claim and the second sees the order as already claimed.

## 6. Completion RPCs

Create:
- `complete_order_as_cook(p_order_id uuid)`
- `complete_order_as_waiter(p_order_id uuid)`

Cook completion requires:
- caller role = cook
- caller active
- order status = `preparing`
- `claimed_by_cook_id = auth.uid()`.

Successful result:
`preparing → ready` and Cook claim cleared.

Waiter completion requires:
- caller role = waiter
- caller active
- order status = `ready`
- `claimed_by_waiter_id = auth.uid()`.

Successful result:
`ready → served` and Waiter claim cleared.

These RPCs replace the current direct Cook `markReady()` and Waiter `markServed()` mutations.

## 7. Claim-Cancellation / Status Safety

Add a database-level order claim invariant trigger that clears invalid claims during status changes.

Required behavior:
- entering `ready` clears `claimed_by_cook_id`.
- entering `served` clears `claimed_by_waiter_id`.
- entering `cancelled` clears both claim fields.
- entering `placed` or `preparing` must not retain a waiter claim.

This is a safety net for all cancellation/status paths, including Manager/Owner direct updates and `cancel_active_orders`.

### Cancellation race

The claim RPC locks the target order row before deciding whether the order is claimable.

Therefore:
- if cancellation wins first, the claim RPC sees `cancelled` and fails;
- if claim wins first, the later cancellation obtains the row lock, changes status to `cancelled`, and the invariant trigger clears the claim.

The final database state is therefore deterministic and cannot leave a cancelled order holding a worker claim.

Existing Cook/Waiter cancellation permissions from Node 2b are otherwise unchanged in Node 13; cancellation remains a separate operational action and is not treated as completion ownership.

## 8. Staff Deactivation / Reassignment

The verified deactivation route `app/api/staff/deactivate/route.ts` hard-deletes the staff `profiles` row.

Therefore Node 13 uses the claim foreign keys with `ON DELETE SET NULL` instead of adding a separate deactivation trigger.

Result:
`claimed order → staff profile deleted → claim automatically becomes NULL → order is claimable again`.

No automatic assignment to another named worker is required for v1.

## 9. Durable Staff Identity

The current system stores staff name in `invite_codes.staff_name`, while `invite_codes` has an auto-delete trigger that removes the oldest 10 used codes once 10 used codes exist.

Therefore `invite_codes.staff_name` is not a durable identity source for Manager assignment history.

Add to `profiles`:

```sql
staff_name text
```

Populate `profiles.staff_name` during staff signup/reactivation from the validated invite code.

Relevant source: `app/api/auth/staff-signup/route.ts` already has access to `codeData.staff_name`, so this is a localized onboarding/data-model change.

## 10. Manager Visibility

Manager needs a live assignment view for active operational orders.

The current Manager dashboard mainly shows `placed` intake and `served` billing, so Node 13 should add an assignment-monitoring section rather than distort the existing queues.

For active `preparing` / `ready` orders, display:
- order number
- status
- Cook claimant name/email or `Unclaimed`
- Waiter claimant name/email or `Unclaimed`.

Add a Manager-only RLS SELECT policy for staff profiles, restricted to staff roles, so claimant name/email can be resolved safely.

The Manager UI must not depend on `invite_codes` for claimant identity.

## 11. Manager Notification

Reuse the existing Node 9 `notifications` table.

Extend the existing `valid_type` constraint with:
`order_claimed`.

When a claim succeeds, insert:
- `recipient_id = <single active Manager profile id>`
- `order_id = claimed order id`
- `type = 'order_claimed'`
- message such as `Order #X accepted by [name]`.

Because the project defines Manager as a single-person role, the implementation should resolve the active Manager explicitly. It should not silently choose one if multiple active Managers exist.

The notification insert belongs in the successful claim RPC so the notice corresponds to the actual winning claimant.

## 12. Node 9 Compatibility

No second notification subsystem is needed.

The existing Node 9 `notify_order_status_change` trigger should remain active:
- Cook completion (`preparing → ready`) continues to create `order_ready` for Waiters.
- Waiter completion (`ready → served`) continues to create `order_served` for Manager.

The new `order_claimed` notification occurs on the claim action itself, which does not change order status. Therefore it does not duplicate those Node 9 status notifications.

## 13. RLS / Bypass Closure

The final migration must explicitly remove or replace the current direct Cook/Waiter completion policies that permit unowned direct updates.

Required outcome:
- Cook cannot directly set `preparing → ready` from the client.
- Waiter cannot directly set `ready → served` from the client.
- Completion is possible only through the ownership-aware RPC.

The existing Node 2b staff cancellation policies remain separate and must not be accidentally widened.

Owner/Manager operational permissions remain as currently defined unless a concrete Node 13 requirement requires a change.

## 14. Realtime

Reuse the existing `orders` Realtime channels:
- Cook: `cook_orders_realtime`.
- Waiter: `waiter_orders_realtime`.
- Manager: existing Manager order realtime path.

Claim-field changes are changes to the existing `orders` row, so no separate Realtime channel is required.

After one worker claims an order, other worker dashboards must refresh and stop presenting that order as claimable.

The existing Realtime publication must remain verified in Supabase before testing; do not assume it from source alone.

## 15. Cook UI

`app/dashboard/cook/page.tsx` should change from:
`Mark Ready` on every preparing order

to a claim-aware flow:

1. Unclaimed order → `Claim Order`.
2. Claimed by current Cook → show `Claimed by You` + `Mark Ready`.
3. Claimed by another Cook → show assignment state and no claim action.
4. If current Cook already owns an active order → prevent another claim.

The UI restriction is only a UX layer; the database remains the enforcement authority.

## 16. Waiter UI

`app/dashboard/waiter/page.tsx` should change from:
`Mark Served` on every ready order

to:

1. Unclaimed ready order → `Claim Order`.
2. Claimed by current Waiter → `Claimed by You` + `Mark Served`.
3. Claimed by another Waiter → no claim action.
4. If current Waiter already owns an active order → prevent another claim.

## 17. Existing SECURITY DEFINER Warning

Do not copy `mark_order_paid` security behavior blindly.

The current `mark_order_paid` RPC lacks the explicit caller-role checks and `search_path` protection required for the new Node 13 RPCs.

Use the explicit authorization approach already demonstrated by `force_logout_all_staff`, while adding the missing safe `search_path` configuration and controlled function execution privileges.

## 18. Implementation Touchpoints

Expected files/areas:

- `supabase/migrations/` — claim columns, indexes, RPCs, RLS, invariant trigger, notification type update, profile staff-name field/policy.
- `app/dashboard/cook/page.tsx` — claim/completion UI and RPC calls.
- `app/dashboard/waiter/page.tsx` — claim/completion UI and RPC calls.
- `app/dashboard/manager/page.tsx` — assignment monitor.
- `app/api/auth/staff-signup/route.ts` — persist `staff_name` into `profiles`.
- `types/index.ts` — claim fields, `staff_name`, and `order_claimed` notification type.

No implementation is authorized by this design document alone.

## 19. Implementation Constraints

- Investigation and implementation remain separate checkpoints.
- All DB migrations are run manually through Supabase SQL Editor.
- Antigravity performs source/build verification, not browser testing.
- Browser/Vercel behavior is verified manually.
- Realtime publication/configuration is explicitly verified before claiming Realtime success.
- No customer notification Bell is introduced.
- Node 12 push notifications remain outside Node 13.

## 20. Final Acceptance Conditions for Node 13

Node 13 cannot be marked complete until all of the following are verified:

- Two Cooks cannot claim the same order.
- Two Waiters cannot claim the same order.
- One Cook cannot hold two active Cook claims.
- One Waiter cannot hold two active Waiter claims.
- A non-claimant cannot complete another worker's order.
- Direct client UPDATE cannot bypass claim ownership.
- Claim vs cancellation produces a deterministic final state.
- Staff profile deletion releases the claim.
- Manager sees current claimant identity.
- Successful claims create the Manager `order_claimed` notification.
- Normal Node 9 completion notifications still work.
- Realtime updates all affected dashboards.
- Existing cancellation/Manager/Owner workflows still work.

## 21. Final Checkpoint

`Node 9 = IMPLEMENTED / LOCKED`
`Node 13 Investigation = COMPLETE`
`Node 13 Design v0.1 = REVIEWED`
`Node 13 Design v1.0 = READY`
`Node 13 Implementation = NOT STARTED`
`Node 13 Testing = NOT STARTED`
`Node 13 Lock = NOT STARTED`
`Node 12 = WAITING`