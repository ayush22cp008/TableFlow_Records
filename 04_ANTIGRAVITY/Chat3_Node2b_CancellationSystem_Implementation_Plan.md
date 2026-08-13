# Node 2b: Order Cancellation System

This document outlines the architecture and execution steps for implementing the comprehensive Order Cancellation System, including single-order cancellation for staff and bulk emergency stops for owners, along with fixing the Manager RLS policy.

## Proposed Changes

### 1. Database Schema Extensions (`supabase/migrations/...`)
I will create a new SQL migration file (`20260804000002_node2b_cancellation.sql`) that performs the following updates:

#### A. Extend `orders` table
- Add `cancellation_reason` (text)
- Add `cancellation_category` (text) with `CHECK (cancellation_category IN ('fire', 'food_safety', 'natural_disaster', 'other', 'manual'))`

#### B. RLS Fixes & Paired Transition Policies
To perfectly prevent cross-product transitions (e.g. `served` -> `preparing`), we MUST use separate, explicitly paired policies for each valid state transition. `USING` checks the old state, and `WITH CHECK` verifies the new state.

I will drop the existing `orders_update_*` policies and create explicit pairs:
- **Cook:**
  - `cook_prep_to_ready`: `USING (status='preparing') WITH CHECK (status='ready')`
  - `cook_cancel`: `USING (status IN ('placed','preparing','ready')) WITH CHECK (status='cancelled' AND cancellation_reason IS NOT NULL)`
- **Waiter:**
  - `waiter_ready_to_served`: `USING (status='ready') WITH CHECK (status='served')`
  - `waiter_cancel`: `USING (status IN ('placed','preparing','ready')) WITH CHECK (status='cancelled' AND cancellation_reason IS NOT NULL)`
- **Manager:**
  - `manager_placed_to_prep`: `USING (status='placed') WITH CHECK (status='preparing')`
  - `manager_served_to_billed`: `USING (status='served') WITH CHECK (status='billed')`
  - `manager_cancel`: `USING (status IN ('placed','preparing','ready')) WITH CHECK (status='cancelled' AND cancellation_reason IS NOT NULL)`
- **Owner:**
  - `owner_all_updates`: `USING (has_role(ARRAY['owner'])) WITH CHECK (has_role(ARRAY['owner']))`

#### C. Bulk Emergency Stop (RPC Approach)
**Approach Decision:** I recommend building this as a **Postgres RPC (Stored Procedure)** rather than batched client updates. 
- *Why?* Atomicity and performance. If an owner is hitting the emergency stop for 50 active orders, doing it in a single DB transaction guarantees everything cancels instantly without race conditions or partial network failures.
- **RPC Signature:** `cancel_active_orders(p_reason text, p_category text, p_order_ids uuid[] DEFAULT NULL)`
- **Logic:**
  1. Validates `auth.uid()` has the `owner` role.
  2. Updates `orders` matching the IDs (or all active if NULL) to `cancelled`.
  3. Automatically zeroes out `occupied_seats` on the associated `restaurant_tables`.

### 2. Application/Frontend Changes

#### [MODIFY] `types/index.ts`
- Add `cancellation_reason?: string` and `cancellation_category?: string` to the `Order` type.

#### [MODIFY] `app/dashboard/orders/page.tsx`
- **Single Cancel (All Staff):** Update the `cancelOrder` function to open a prompt (or small modal) to require a mandatory free-text reason before allowing the cancellation to proceed. The update will set `cancellation_category: 'manual'`.
- **Bulk Cancel (Owner Only):** 
  - Add a "⚠️ Bulk Emergency Stop" button at the top of the board, visible only if `userRole === 'owner'`.
  - When clicked, opens a modal with:
    - Mode toggle: "Cancel All Active Orders" vs "Select Specific Orders"
    - If "Select Specific", reveals checkboxes next to active orders on the board.
    - Category dropdown: (Fire / Food Safety / Natural Disaster / Other) (Mandatory)
    - Additional Details text area (Optional)
  - Submits the payload to the `cancel_active_orders` RPC.

## Verification Plan
1. **RLS Verification:** Attempt to transition an order from `served` to `preparing` directly in the DB as a Manager (it must be rejected by RLS).
2. **Single Cancel:** Cancel an order as a Waiter without providing a reason (must fail). Provide a reason, and it should succeed.
3. **Bulk Cancel:** As an owner, trigger a Bulk Emergency Stop for "Fire". Verify all active orders transition to `cancelled` and the tables become `available`.

## User Review Required

> [!IMPORTANT]
> - Do you approve the **RPC Approach** for the Bulk Emergency Stop, or would you strongly prefer client-side batching?
> - Do you approve the **Paired RLS Policies** method (e.g. creating 7 distinct `CREATE POLICY` statements to strictly isolate every transition)? This is the only bulletproof way to prevent cross-product state bugs in Postgres RLS.
