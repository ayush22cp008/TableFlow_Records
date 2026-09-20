# TableFlow — Chat 18 — Node 13 Design v0.1

**Purpose:** Compact design reference for Claude design review before implementation.
**Status:** DRAFT — NOT APPROVED FOR IMPLEMENTATION
**Node:** 13 — Cook/Waiter Order Claiming System
**Source repository:** https://github.com/ayush22cp008/TableFlow
**Investigation:** `06_INVESTIGATIONS/Chat18_Node13_Investigation_result_report.md`

## 1. Goal
Introduce exclusive Cook/Waiter order claiming while preserving the existing TableFlow order lifecycle and preventing race-condition bypasses.

## 2. Proposed Data Model
Extend `orders` with:

- `claimed_by_cook_id uuid NULL REFERENCES profiles(id)`
- `claimed_by_waiter_id uuid NULL REFERENCES profiles(id)`

Use partial unique indexes so each Cook can hold at most one active Cook claim and each Waiter at most one active Waiter claim.

Proposed permanent staff identity improvement:
- Consider `profiles.staff_name text` because the current staff-management UI reconstructs names from `invite_codes.staff_name`.
- This must be validated during design review; do not assume it is required.

## 3. Existing Lifecycle to Preserve
Current verified workflow:

`placed → preparing → ready → served → billed`

Cook entry state remains `preparing` because the current Cook dashboard queries `status = 'preparing'`.
Waiter entry state remains `ready` because the current Waiter dashboard queries `status = 'ready'`.

Claim lifecycle:

`preparing → Cook claims → preparing + cook claim → ready → cook claim released`
`ready → Waiter claims → ready + waiter claim → served → waiter claim released`

## 4. Atomic Claiming
Do not implement claim as a client-side read/check followed by update.

Use database-side atomic conditional mutation through RPCs.

Proposed RPCs:
- `claim_order_as_cook(p_order_id)`
- `claim_order_as_waiter(p_order_id)`
- `complete_order_as_cook(p_order_id)`
- `complete_order_as_waiter(p_order_id)`

Claim conditions must validate:
- authenticated caller
- correct role
- `is_active = true`
- correct order status
- claim field is NULL
- caller has no other active claim of the same worker type

The claim operation must atomically establish the successful claimant so concurrent workers cannot both win.

## 5. One-Order-at-a-Time
Proposed database enforcement:

`UNIQUE (claimed_by_cook_id) WHERE claimed_by_cook_id IS NOT NULL`
`UNIQUE (claimed_by_waiter_id) WHERE claimed_by_waiter_id IS NOT NULL`

The design review must verify that these constraints behave correctly during claim, completion, cancellation, deactivation, and concurrent transactions.

## 6. Completion / Release
Cook completion:
`preparing + claimed_by_cook_id → ready + cook claim cleared`

Waiter completion:
`ready + claimed_by_waiter_id → served + waiter claim cleared`

Completion must verify that the current caller is the claimant.

Cancellation must clear any relevant claim so an abandoned order cannot consume a worker's one-order slot.

## 7. Cancellation Race
Emergency Stop cancellation and claiming can occur concurrently.
The final implementation must define one deterministic database outcome and ensure an order cannot end up in an invalid claimed/cancelled state.

## 8. Staff Deactivation
Current system uses `profiles.is_active` and `has_role()` to block deactivated staff.

Proposed Node 13 behavior:
- deactivation of a claimed Cook releases the Cook claim
- deactivation of a claimed Waiter releases the Waiter claim
- released order becomes available to another eligible worker.

The design review must decide whether release belongs in the deactivation path, a database trigger, or both without causing duplicated or unsafe behavior.

## 9. Manager Visibility
Manager should see assignment information for active orders, including:
- claimant type
- claimant name
- claimant email
- order status

Unclaimed orders should explicitly appear as unclaimed.

Manager visibility must be checked against existing `profiles` RLS.

## 10. Manager Notification
Reuse the existing Node 9 `notifications` infrastructure.

Add notification type:
- `order_claimed`

Only after a claim actually succeeds should the Manager receive a notification such as:
`Order #X accepted by [name]`

Node 13 should not create a separate notification subsystem.

## 11. Realtime
Reuse the existing `orders` Realtime channels used by Cook and Waiter dashboards.

Claim changes should cause other worker dashboards to refresh so another worker cannot continue seeing an order as claimable after it has been claimed.

The final implementation must explicitly verify that the relevant `orders` publication/Realt​ime configuration covers the new columns and that Manager UI refreshes correctly.

## 12. RLS / Security Direction
Current Cook/Waiter direct status updates are not ownership-aware.

Final design must:
- close direct mutation paths that bypass claim ownership
- make completion ownership-aware
- keep claim mutations behind controlled RPCs
- validate role and active status inside the database operation
- use safe `SECURITY DEFINER` configuration if RPCs require it, including explicit `search_path` and controlled execution privileges.

Do not assume existing RLS is sufficient for claim ownership.

## 13. Existing Source Touchpoints
Expected implementation touchpoints, subject to review:

- `app/dashboard/cook/page.tsx` — replace direct `markReady()` path with claim-aware workflow.
- `app/dashboard/waiter/page.tsx` — replace direct `markServed()` path with claim-aware workflow.
- `app/dashboard/manager/page.tsx` — display claim information.
- `app/dashboard/staff/page.tsx` and/or onboarding flow — only if permanent `staff_name` is approved.
- `types/index.ts` — add claim fields and `order_claimed` notification type.
- `supabase/migrations/` — schema, indexes, RPCs, RLS, and lifecycle protections.

## 14. Scope Boundary
Inside Node 13:
- exclusive Cook claim
- exclusive Waiter claim
- one-order-at-a-time rule
- race-condition protection
- ownership-aware completion
- Manager assignment visibility
- Manager claim notifications
- claim release on cancellation/deactivation
- Realtime synchronization.

Outside Node 13:
- OS/browser push notifications
- PWA push infrastructure
- email notifications
- customer notification bell/UI.

## 15. Design Review Questions
Claude must verify:
1. Whether two claim columns are preferable to a separate assignment table for this project.
2. Whether the partial unique indexes correctly enforce one active claim per worker.
3. Exact atomic transaction pattern needed for concurrent claims.
4. Required RLS policy changes and any direct-update bypass paths.
5. `SECURITY DEFINER` safety and `search_path` requirements.
6. Claim-vs-cancellation race behavior.
7. Deactivation release architecture.
8. Whether `profiles.staff_name` is genuinely necessary.
9. Whether Manager can safely read claimant identity under existing profile RLS.
10. Compatibility with Node 9 notifications.
11. Realtime behavior after claim-field changes.
12. Any unnecessary complexity that should be removed before implementation.

## 16. Non-Goals
- No implementation in this document.
- No migration execution.
- No database changes.
- No UI testing.
- No Node 13 lock.
- No Node 12 push work.

## 17. Current Checkpoint
`Node 9 = LOCKED`
`Node 13 Investigation = COMPLETE`
`Node 13 Design v0.1 = DRAFT`
`Node 13 Implementation = NOT STARTED`
`Node 13 Testing = NOT STARTED`
`Node 13 Lock = NOT STARTED`