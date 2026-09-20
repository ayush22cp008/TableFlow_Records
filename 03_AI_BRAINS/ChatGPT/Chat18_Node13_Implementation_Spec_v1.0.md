# TableFlow — Chat 18 — Node 13 Implementation Specification v1.0

**Node:** 13 — Cook/Waiter Order Claiming System
**Status:** APPROVED DESIGN → IMPLEMENTATION SPECIFICATION
**Implementation:** NOT STARTED
**Source Repository:** https://github.com/ayush22cp008/TableFlow
**Records:** https://github.com/ayush22cp008/TableFlow_Records

## 1. Implementation Goal

Implement exclusive Cook/Waiter order claiming in the existing TableFlow workflow without breaking the existing order lifecycle, Node 9 notifications, cancellation, staff management, or Realtime behavior.

The database is the enforcement authority. UI checks are convenience only.

## 2. Existing Lifecycle — Must Remain

```text
placed → preparing → ready → served → billed
```

Cook claim entry state: `preparing`.
Waiter claim entry state: `ready`.

Cook:
`preparing → claimed → ready`

Waiter:
`ready → claimed → served`

Do not introduce a new order status for claiming.

## 3. Database Migration

Create one Node 13 migration in `supabase/migrations/` after the current latest migration.

Recommended filename:

```text
20260920000001_node13_order_claiming.sql
```

The migration must include:

### 3.1 Order claim columns

```sql
ALTER TABLE public.orders
  ADD COLUMN IF NOT EXISTS claimed_by_cook_id uuid NULL REFERENCES public.profiles(id) ON DELETE SET NULL,
  ADD COLUMN IF NOT EXISTS claimed_by_waiter_id uuid NULL REFERENCES public.profiles(id) ON DELETE SET NULL;
```

### 3.2 One-active-order indexes

```sql
CREATE UNIQUE INDEX IF NOT EXISTS idx_orders_one_active_cook_claim
ON public.orders (claimed_by_cook_id)
WHERE claimed_by_cook_id IS NOT NULL;

CREATE UNIQUE INDEX IF NOT EXISTS idx_orders_one_active_waiter_claim
ON public.orders (claimed_by_waiter_id)
WHERE claimed_by_waiter_id IS NOT NULL;
```

These enforce one active Cook claim per Cook and one active Waiter claim per Waiter.

## 4. Durable Staff Name

Add:

```sql
ALTER TABLE public.profiles
  ADD COLUMN IF NOT EXISTS staff_name text;
```

Reason: `invite_codes.staff_name` is not durable because the project has an auto-delete trigger for used invite codes.

Update `app/api/auth/staff-signup/route.ts` so `staff_name: codeData.staff_name` is written in BOTH profile update paths:

- existing-profile / reactivation branch
- new-user branch after profile creation.

Do not remove the existing email/role behavior.

## 5. Claim RPCs

Create four RPCs:

```text
claim_order_as_cook(p_order_id uuid)
claim_order_as_waiter(p_order_id uuid)
complete_order_as_cook(p_order_id uuid)
complete_order_as_waiter(p_order_id uuid)
```

All four must use:

- `LANGUAGE plpgsql`
- `SECURITY DEFINER`
- `SET search_path = public, pg_temp`
- explicit authorization checks inside the function
- schema-qualified project tables/functions

Also explicitly control execution privileges:

```sql
REVOKE ALL ON FUNCTION ... FROM PUBLIC;
GRANT EXECUTE ON FUNCTION ... TO authenticated;
```

Do not copy the security characteristics of `mark_order_paid` blindly.

## 6. Claim RPC Authorization

### Cook claim

Require:
- authenticated caller
- `auth.uid()` exists
- caller profile role = `cook`
- caller `is_active = true`
- target order exists
- target order status = `preparing`
- `claimed_by_cook_id IS NULL`
- caller has no other active Cook claim.

### Waiter claim

Same rules, except:
- role = `waiter`
- target status = `ready`
- `claimed_by_waiter_id IS NULL`
- caller has no other active Waiter claim.

## 7. Atomic Concurrency Control

Claim RPCs must serialize concurrent attempts.

Required sequence:

1. Resolve and lock the caller's profile row with `FOR UPDATE`.
2. Resolve and lock the target order row with `FOR UPDATE`.
3. Re-check all eligibility conditions after locking.
4. Verify the caller still has no active claim.
5. Update the correct claim column.
6. Insert the Manager `order_claimed` notification in the same transaction.
7. Commit.

The target order row lock determines the winner when multiple workers claim the same order.

The profile-row lock prevents the same Cook/Waiter from winning two simultaneous claim attempts.

The partial unique index remains the final database backstop.

## 8. Manager Resolution

The claim RPC must identify the active Manager explicitly.

Because project architecture defines Manager as a single-person role:

- exactly one active Manager is expected;
- zero active Managers is an operational configuration error;
- multiple active Managers is an operational configuration error.

Do not silently select an arbitrary Manager with `LIMIT 1` if multiple active Managers exist.

The claim transaction should fail clearly when the Manager configuration is invalid rather than creating an orphan `order_claimed` event.

## 9. Claim Notification

Extend the existing Node 9 notification type constraint with:

```text
order_claimed
```

The notification must use the existing `notifications` table:

```text
recipient_id = manager profile id
recipient_role = NULL
order_id = claimed order id
type = order_claimed
message = Order #X accepted by [name]
```

Do not create another notification table or delivery mechanism.

The notification is inserted only after the claim has actually succeeded.

## 10. Completion RPC — Cook

`complete_order_as_cook(p_order_id)` must:

1. Verify authenticated active Cook.
2. Lock the target order row.
3. Require `status = preparing`.
4. Require `claimed_by_cook_id = auth.uid()`.
5. Set `status = ready`.
6. Set `claimed_by_cook_id = NULL`.
7. Update `updated_at`.

No other Cook may complete that order.

The existing Node 9 status trigger will then generate the normal Waiter `order_ready` notification.

## 11. Completion RPC — Waiter

`complete_order_as_waiter(p_order_id)` must:

1. Verify authenticated active Waiter.
2. Lock the target order row.
3. Require `status = ready`.
4. Require `claimed_by_waiter_id = auth.uid()`.
5. Set `status = served`.
6. Set `claimed_by_waiter_id = NULL`.
7. Update `updated_at`.

The existing Node 9 status trigger will then generate the normal Manager `order_served` notification.

## 12. Claim Invariant Trigger

Create a `BEFORE UPDATE` trigger on `public.orders` that clears invalid claim ownership whenever order state moves beyond the claim lifecycle.

Required invariants:

- `NEW.status = 'ready'` → `NEW.claimed_by_cook_id = NULL`.
- `NEW.status IN ('placed', 'preparing')` → `NEW.claimed_by_waiter_id = NULL`.
- `NEW.status IN ('served', 'billed', 'cancelled')` → both claim fields NULL.

This is a safety net for every mutation path, including existing Manager/Owner/cancellation updates.

The trigger must not overwrite a valid active Cook claim on a `preparing` order or a valid active Waiter claim on a `ready` order.

## 13. Cancellation Race

The claim RPC and cancellation must converge to a deterministic final state.

Required behavior:

- Claim locks order first. If cancellation has already committed, claim sees `cancelled` and fails.
- If claim commits first, later cancellation changes status to `cancelled`; the invariant trigger clears both claim fields.
- Final state must never be `cancelled` with a worker claim remaining.

Do not modify the existing cancellation business rules beyond what is required to keep claims consistent.

## 14. Existing Cook/Waiter Direct-Update Bypass

The current Node 2b policies permit direct Cook/Waiter completion updates.

Node 13 must remove the direct completion bypass.

Specifically, the migration must remove the existing Cook/Waiter completion policies that permit unowned direct status transitions:

```text
cook_prep_to_ready
waiter_ready_to_served
```

Do NOT remove the separate Cook/Waiter cancellation policies unless a concrete conflict is found.

Completion becomes RPC-only.

A direct client attempt such as:

```ts
supabase.from('orders').update({ status: 'ready' })
```

must fail for a Cook because no Cook UPDATE policy permits that transition.

## 15. Profiles RLS for Manager Assignment Visibility

The current `profiles` RLS allows own-user reads and Owner reads.

Add a Manager-only SELECT policy allowing the active Manager to read staff identity fields needed for assignment display:

- `id`
- `staff_name`
- `email`
- `role`
- `is_active`

Restrict the policy to staff roles (`cook`, `waiter`, `manager`).

Do not expose unrelated profile data.

## 16. Manager Dashboard

Update `app/dashboard/manager/page.tsx`.

Current Manager dashboard contains:
- Intake Queue (`placed`)
- Billing Queue (`served`)
- `markPreparing()`
- `markPaid()`
- `manager_orders_realtime`.

Keep those workflows intact.

Add a separate assignment-monitoring section for operational orders:

- `preparing` orders → Cook claimant
- `ready` orders → Waiter claimant.

Display:
- order number
- table when available
- status
- Cook name/email or `Unclaimed`
- Waiter name/email or `Unclaimed`.

When embedding both claimant profiles through Supabase, use the explicit FK relation names if PostgREST requires disambiguation because both foreign keys point to `profiles`.

## 17. Cook Dashboard

Update `app/dashboard/cook/page.tsx`.

Current behavior:
- queries `status = preparing`
- direct `markReady()` update.

New behavior:

### Unclaimed order
`Claim Order` button.

### Claimed by current Cook
Show `Claimed by You` and `Mark Ready`.

### Claimed by another Cook
Show claimed/assigned state and no claim action.

### Current Cook already has another active claim
Disable claim actions for additional unclaimed orders.

The UI must call the RPCs and handle database rejection cleanly.

Do not rely on the UI to enforce ownership.

## 18. Waiter Dashboard

Update `app/dashboard/waiter/page.tsx`.

Current behavior:
- queries `status = ready`
- direct `markServed()` update.

New behavior mirrors Cook:

- unclaimed → `Claim Order`
- claimed by current Waiter → `Claimed by You` + `Mark Served`
- claimed by another Waiter → no claim action
- current Waiter already has another active claim → disable additional claims.

Use the completion and claim RPCs.

## 19. Staff Signup

Update `app/api/auth/staff-signup/route.ts`.

Current route already has `codeData.staff_name` available.

Add `staff_name` to BOTH profile updates:

### Reactivation branch
Existing update currently sets role/active/login state. Add `staff_name`.

### New-user branch
Existing profile update currently sets role. Add `staff_name`.

Do not change invite validation behavior.

## 20. TypeScript Types

Update `types/index.ts`:

### `Order`
Add:
- `claimed_by_cook_id: string | null`
- `claimed_by_waiter_id: string | null`

### `UserProfile`
Add:
- `staff_name?: string | null`
- `is_active?: boolean` if needed by Manager assignment types.

### `NotificationType`
Add:
- `order_claimed`.

### `AppNotification`
No structural change required because `recipient_id`, `order_id`, type, and message already exist.

## 21. NotificationBell

`components/NotificationBell.tsx` already supports direct `recipient_id` notifications and renders the stored message generically.

No separate Node 13 NotificationBell architecture is required.

Only make a NotificationBell change if implementation discovers an actual TypeScript/runtime incompatibility caused by the new type.

## 22. Realtime

Reuse existing channels:

- Cook: `cook_orders_realtime`
- Waiter: `waiter_orders_realtime`
- Manager: `manager_orders_realtime`

Claim field changes are ordinary `orders` row changes.

Before manual verification, confirm in Supabase that the existing `orders` table is still configured for Realtime publication.

No new Realtime table/channel is required for Node 13.

## 23. Node 9 Notification Compatibility

Keep the existing Node 9 triggers:

- `preparing → ready` → `order_ready` to Waiter role.
- `ready → served` → `order_served` to Manager role.

Claim itself does not change status, so it should create only `order_claimed`.

Do not duplicate `order_ready`/`order_served` notifications from the Node 13 claim step.

## 24. Security Requirements

Every new RPC must:

- reject anonymous callers;
- verify exact application role;
- verify active profile;
- verify target order ownership/state;
- use safe `search_path`;
- be executable only by intended authenticated callers;
- avoid trusting client-supplied claimant IDs.

The claimant ID must always come from `auth.uid()`.

No RPC may accept a Cook/Waiter profile ID as a caller identity parameter.

## 25. Error Handling

Recommended business-error messages:

- `Order is not available for claiming.`
- `You already have an active order.`
- `This order is already claimed.`
- `You do not own this order.`
- `No active Manager is configured.`
- `Multiple active Managers are configured.`

Frontend should surface a clear error and refetch the queue after a failed claim caused by a concurrent worker.

## 26. Implementation Order

Implement in this order:

1. Database migration.
2. Profile/staff-name data path.
3. Claim/completion RPCs.
4. RLS bypass closure.
5. Claim invariant trigger.
6. Notification type extension and claim notification.
7. TypeScript types.
8. Cook dashboard.
9. Waiter dashboard.
10. Manager assignment monitor.
11. Build/type-check verification.

Do not mix implementation with manual browser verification.

## 27. Migration Execution Rule

Create the migration in the source repository, but run the SQL manually through the Supabase SQL Editor according to the project's established process.

Before execution:
- inspect the final migration;
- confirm it does not overwrite unrelated Node 9/Node 11/Node 2b behavior;
- confirm existing orders are compatible with NULL claim fields.

## 28. Build Verification

Antigravity/source verification must confirm:

- TypeScript passes.
- Next.js production build passes.
- New RPC references compile.
- Supabase query relation syntax is valid.
- No stale direct Cook/Waiter completion mutation remains.

Browser verification is separate and must be done manually.

## 29. Manual Acceptance Matrix

### Cook

| Test | Expected |
|---|---|
| Unclaimed preparing order | Cook can claim |
| Second Cook claims same order | Rejected |
| Current Cook claims second order | Rejected |
| Current Cook completes own order | `ready`, Cook claim cleared |
| Cook completes another Cook's order | Rejected |
| Direct Cook status update | Rejected by RLS |

### Waiter

| Test | Expected |
|---|---|
| Unclaimed ready order | Waiter can claim |
| Second Waiter claims same order | Rejected |
| Current Waiter claims second order | Rejected |
| Current Waiter completes own order | `served`, Waiter claim cleared |
| Waiter completes another Waiter's order | Rejected |
| Direct Waiter status update | Rejected by RLS |

### Concurrency / Cancellation

| Test | Expected |
|---|---|
| Two Cooks claim same order | Exactly one succeeds |
| Two Waiters claim same order | Exactly one succeeds |
| Claim + Emergency Stop race | Final state deterministic; no retained claim on cancelled order |
| Cancel claimed Cook order | Cook claim cleared |
| Cancel claimed Waiter order | Waiter claim cleared |

### Staff lifecycle

| Test | Expected |
|---|---|
| Deactivate/delete claimed Cook | FK sets claim NULL; order becomes claimable |
| Deactivate/delete claimed Waiter | FK sets claim NULL; order becomes claimable |
| New staff signup | `profiles.staff_name` populated |
| Staff reactivation path | `profiles.staff_name` updated |

### Manager

| Test | Expected |
|---|---|
| Manager sees claimed Cook | Name + email visible |
| Manager sees claimed Waiter | Name + email visible |
| Successful Cook claim | Manager receives `order_claimed` notification |
| Successful Waiter claim | Manager receives `order_claimed` notification |
| Realtime claim update | Dashboard refreshes correctly |

### Regression

- Node 9 notification Bell still works.
- Normal `order_ready` notification still works.
- Normal `order_served` notification still works.
- Manager `Accept (Send to Kitchen)` still works.
- Manager `Mark Paid` still works.
- Existing cancellation workflows still work.
- Existing customer order flow still works.

## 30. Completion Boundary

Node 13 must NOT be marked locked after source changes alone.

Required sequence:

```text
Implementation
→ source/build verification
→ Supabase migration execution
→ DB verification
→ manual Vercel/browser verification
→ regression verification
→ evidence/report
→ approval
→ Node 13 LOCKED
```

## 31. Out of Scope

- Customer notification Bell.
- OS/browser push.
- PWA push infrastructure.
- Email notifications for claims.
- Node 12 push implementation.
- New order status values.
- Separate assignment table unless implementation discovers a concrete blocker that cannot be solved with the approved two-column design.

## 32. Final Checkpoint

```text
Node 9                  ✅ LOCKED
Node 13 Investigation   ✅ COMPLETE
Node 13 Design v1.0     ✅ CLAUDE APPROVED
Node 13 Spec v1.0       ✅ READY
Node 13 Implementation  ⬜ NOT STARTED
Node 13 Testing         ⬜ NOT STARTED
Node 13 Lock            ⬜ NOT STARTED
Node 12                 ⬜ WAITING
```