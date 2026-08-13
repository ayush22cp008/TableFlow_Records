# Chat 9: Node 4 — Verification of Stale Reservation Rows

## 1 & 2. DB Verification Results
Ayush's hypothesis is **100% correct**. 
I ran a custom Node script directly querying the live DB via the Supabase JS client. 
There are exactly **5 stale rows** in `reservation_requests` that have `status = 'approved'`, but point to a table that is currently completely free (`occupied_seats = 0`, `status = 'available'`, `reserved_from = null`).

Yes, Table 1, 2, and 3 are all affected.
The exact stale rows found:
- **Req ID:** `ebbc6666-54be-46bf-98f0-f7646421ee90` (Table 3)
- **Req ID:** `b7d766b9-6f9c-44d0-bffd-30b367536c97` (Table 1)
- **Req ID:** `3d67ce72-5d93-4c5d-8124-cce7fd380658` (Table 1)
- **Req ID:** `6fde22d9-5d58-4ef7-96c2-8556b4675963` (Table 2)
- **Req ID:** `d96124b8-2791-49d6-ae4b-354a5a1703f0` (Table 6)

## 3. Root Cause Confirmation
Because these rows are permanently stuck in `'approved'` status, the newly added overlap check (`r.status === 'approved' && r.table_id === tableId`) correctly triggers a false positive, thinking the table is still reserved, even though it was manually cleared on the `restaurant_tables` side.

## 4. Schema Lifecycle Analysis
The `reservation_requests` schema **does** have a lifecycle status beyond `approved`: it uses `'completed'` (set when the customer scans their code and places an order via the cart). 

However, it is **missing** transitions for exceptions/overrides. If a customer never shows up (no-show), or if the Manager manually clicks "Clear" on a table, or if the `mark_order_paid` / `cancel_active_orders` RPCs free up a table, the original `reservation_requests` row is completely ignored and left in `'approved'` status forever.

## 5. Options for determining "Active" (For Ayush to Decide)
To fix this false-positive overlap check, here are the options:

**Option A: Defensive UI Check (Rely on Table State)**
Instead of just checking `reservation_requests.status === 'approved'`, the overlap check can cross-reference the `tables` state. If the target table's `reserved_from` is `null`, it implies any associated "approved" requests are stale/void.
- *Pros:* Quick to implement in React, fixes the immediate UI block.
- *Cons:* Leaves junk/stale state in the database, breaking analytics and future features.

**Option B: DB-level Consistency (Proper Lifecycle Management)**
Add a `cancelled` or `expired` status to `reservation_requests`. Update all backend flows (Clear button, Mark Paid RPC, Cancel RPC) to actively look up any `'approved'` requests tied to that `table_id` and transition them to `cancelled` when releasing the table.
- *Pros:* Database remains consistent and clean.
- *Cons:* Requires updating multiple RPCs and UI functions (Clear button).

**Option C: Both (Recommended)**
Implement a quick UI patch (Option A) to unblock the dashboard immediately, and batch the DB cleanup (Option B) for a future structural refactor.

**Status:** Verification complete. Awaiting your decision on the fix approach. No code has been modified.
