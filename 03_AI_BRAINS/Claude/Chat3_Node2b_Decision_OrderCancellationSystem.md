# Chat #3 | Node 2b — Decision: Order Cancellation System (LOCKED)

## Context
Original Permission Matrix (Chat1_Node1) did not scope cancellation at all. This was surfaced during Manager Orders RLS policy review and expanded into a full cancellation design covering all roles.

## Feature 1: Single-Order Cancellation (scope addition to Node 2b)

- **Who:** Manager, Cook, Waiter, Owner — all four roles can cancel an order.
- **When:** Order status must be `placed`, `preparing`, or `ready`. NOT allowed once `served` (food has reached the customer — cancellation no longer makes sense; a different void/refund flow would apply, out of scope here).
- **Requirement:** A reason (free text) is mandatory to submit a cancellation — cancellation request is rejected if no reason provided.
- **Result:** Order status -> `cancelled`. Flow stops there permanently, no forward transitions after cancellation.
- **Normal flow unchanged:** `placed -> preparing -> ready -> served -> billed` remains exactly as locked in the original Permission Matrix. Cancellation is a separate, parallel exit path available from any of the first three states — not a replacement for the normal flow.

## Feature 2: Owner Bulk Emergency Stop (NEW feature, standalone)

- **Who:** Owner ONLY. Manager/Cook/Waiter do NOT get bulk access — single-order cancellation only (Feature 1).
- **What:** Owner can cancel multiple orders/tables at once for restaurant-wide emergencies (fire, food safety issue, natural disaster, etc.)
- **Selection modes (both required):**
  1. **Select specific** — Owner checkbox-selects specific tables/orders to bulk-cancel.
  2. **Cancel all** — single action cancels ALL currently-active orders (placed/preparing/ready) at once.
- **Reason capture:** Predefined dropdown (Fire / Food Safety / Natural Disaster / Other) — mandatory selection. Optional free-text field for additional detail, not required.
- **Result:** All selected/all-active orders -> `cancelled`, same terminal behavior as single-order cancellation.

## RLS / Schema Implications
- `orders` table needs a `cancellation_reason` (text) column, and likely a `cancellation_category` (text/enum: fire/food_safety/natural_disaster/other/manual) to distinguish bulk emergency cancels from routine single cancels.
- UPDATE policies for Manager/Cook/Waiter/Owner need a `-> cancelled` transition added, scoped to `USING (status IN ('placed','preparing','ready'))`, with `WITH CHECK (status = 'cancelled' AND cancellation_reason IS NOT NULL)`.
- Bulk stop is likely a dedicated RPC (transaction wrapping multiple row updates) rather than plain client-side UPDATE, so "cancel all active orders" is atomic. Antigravity to determine best implementation (RPC vs. batched client updates) during implementation, reporting back the approach before building.

## Next
Antigravity instruction to follow: extend orders schema (cancellation_reason, cancellation_category), add scoped cancel transitions to each role's UPDATE RLS policy, build single-cancel UI (Manager/Cook/Waiter/Owner dashboards) and Owner-only bulk emergency stop UI (select-specific + cancel-all modes).
