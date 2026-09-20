# TableFlow — Chat 18 — Node 13 Investigation

**Node:** 13 — Cook/Waiter Order Claiming System  
**Chat:** 18  
**Type:** Investigation only  
**Status:** READY FOR SOURCE INVESTIGATION  
**Repository:** https://github.com/ayush22cp008/TableFlow

## Objective
Investigate the current TableFlow source before any Node 13 implementation.
Identify exactly how Cook and Waiter order handling works today, where order claiming must be introduced, what DB/schema/RLS constraints exist, and what must be investigated for safe exclusive claiming.

## Scope

### 1. Current Cook workflow
- Current order query/filter.
- Statuses currently visible to Cook.
- Current order status transitions.
- Exact file, function, and component responsible.
- Realtime behavior.
- Whether multiple Cooks can currently act on the same order.
- Current authentication and role checks.
- Cook RLS policies relevant to orders.
- Existing assignment/claim field or mechanism, if any.

### 2. Current Waiter workflow
- Current order query/filter.
- Statuses currently visible to Waiter.
- Current order status transitions.
- Exact file, function, and component responsible.
- Realtime behavior.
- Whether multiple Waiters can currently act on the same Ready order.
- Current authentication and role checks.
- Waiter RLS policies relevant to orders.
- Existing assignment/claim field or mechanism, if any.

### 3. Current Manager workflow
- Current order visibility.
- Current order transitions.
- Relevant order fields.
- Existing staff/profile information available near orders.
- Files, functions, and components relevant to future assignment visibility.
- Realtime behavior.
- Manager RLS policies relevant to orders.

### 4. Database / schema investigation
- Current `orders` table schema.
- Whether `assigned_cook_id` exists.
- Whether `assigned_waiter_id` exists.
- Whether a separate assignment/claim table exists.
- Existing foreign keys to profiles/staff identities.
- Relevant constraints and indexes.
- Current order-update RLS for Cook, Waiter, Manager, and Owner roles.
- Existing order mutation RPCs/functions.
- Whether any existing RPC can support atomic claiming safely.

### 5. Race-condition investigation
- Determine what happens today if two Cooks attempt to claim the same order simultaneously, based only on existing source/database behavior.
- Determine the equivalent current behavior for two Waiters attempting to act on the same Ready order.
- Investigate existing DB capabilities relevant to atomic conditional updates, affected-row checks, unique constraints, transactional RPCs, or row locking.
- Do not design or implement the Node 13 claiming solution yet.

### 6. One-order-at-a-time feasibility
- Identify what current data would be required to enforce Cook’s one-active-claimed-order rule.
- Verify what current order states could represent release of the Cook for another claim.
- Investigate the equivalent Waiter lifecycle.
- Do not implement the one-order-at-a-time rule.

### 7. Reassignment / deactivation
- Investigate current staff deactivation behavior.
- Determine whether deactivated Cooks/Waiters can still access or mutate orders.
- Investigate any existing abandoned-work, reassignment, or recovery mechanism.
- Mark UNKNOWN where the repository does not establish the behavior.

### 8. Manager notification insertion point
- Identify the exact likely source/database insertion point for a future notification such as `Order #X accepted by [name]`.
- Do not implement the notification.

## Required output
The completed investigation must contain these sections:

1. Executive Summary
2. Cook Current State
3. Waiter Current State
4. Manager Current State
5. Database / Schema Facts
6. RLS / Permissions Facts
7. Race-Condition Facts
8. One-Order-at-a-Time Facts
9. Reassignment / Deactivation Facts
10. Candidate Claim Insertion Points
11. Confirmed Facts vs UNKNOWN
12. Node 13 Design Inputs for Next Stage

Every important statement should include the exact repository file path plus the relevant function/component name whenever the source provides one.

## Strict investigation rules
- READ ONLY.
- No source modifications.
- No SQL migration modifications.
- No database changes.
- No Claim/Accept buttons.
- No fixes.
- No schema decisions.
- No invented behavior.
- Explicitly distinguish VERIFIED facts from UNKNOWN items.
- Mention Node 12 only where its dependency matters.
- Customer-side notification UI is out of scope.
- Investigation first; implementation comes later.
- Do not declare Node 13 implemented or locked.

## Expected conclusion
The investigation should identify the concrete current-state facts required to write the Node 13 design/implementation specification in the next stage. It must not declare Node 13 implemented or locked.