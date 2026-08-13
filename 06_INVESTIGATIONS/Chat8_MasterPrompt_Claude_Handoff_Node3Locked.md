# TableFlow Staff Role System — Chat #8 — Master Prompt (Claude Side)

**Status:** Node 3 (Manager Dashboard) — ✅ LOCKED

---

## Node Map

| Node | Description | Status |
|---|---|---|
| 1 | Permission Matrix | ✅ LOCKED |
| 2b | Schema, RLS, invite codes, cancellation, email delivery | ✅ LOCKED |
| Routing | Role-based auth redirects | ✅ LOCKED |
| Cook Dashboard | KDS | ✅ LOCKED |
| Manager Dashboard | Intake/Billing queues, priority sort, W#/R# labels, Mark Paid table release | ✅ LOCKED (this session) |
| Waiter Dashboard | — | ✅ LOCKED |
| 4 | Reservation double-booking prevention | ⬜ NOT STARTED (new, discovered this session) |
| Notifications | — | ⬜ NOT STARTED |

## Node 3 (Manager Dashboard) — Locked Summary

Three bugs fixed and verified this session, in order:

1. **Priority sort (Intake + Billing Queue)** — both queues were missing `is_priority` sort clause, only sorting by `created_at`. Fixed by adding `.order('is_priority', {ascending:false}).order('created_at',{ascending:true})` matching Owner/Cook's existing pattern. Verified: R-orders now appear above W-orders in both queues.

2. **W#/R# label retrofit** — Manager Dashboard was showing raw UUID fragments (`Order #c7720b`) instead of clean sequential labels. Added shared `formatOrderNumber()` util, wired into Manager's Intake/Billing queue rendering. Verified: Manager now shows `Order W13`, `Order R4` etc., matching Owner/Cook/Waiter convention. (Underlying `daily_number` confirmed DB-stored via `daily_order_counters`, resets daily by design — not a bug.)

3. **Mark Paid → table release (two-part fix)**
   - **3a:** `markPaid()` only updated `orders.status`, never touched `restaurant_tables`. Fixed via new `mark_order_paid` RPC (mirrors the proven `cancel_active_orders` release pattern) — decrements `occupied_seats` by `party_size`, flips table to `available` only when fully released. Handles multi-order tables correctly.
   - **3b:** After 3a, Walk-in orders released correctly but Reservation orders stayed visually "Reserved" (purple) — root cause: `reserved_from` was never cleared by the new RPC. Fixed by conditionally clearing `reserved_from` only when the table is fully releasing (`occupied_seats <= 0` after decrement), preserving correct behavior for multi-order reservation tables.
   - **Noted, not fixed (out of scope):** Owner's Generate Bill flow unconditionally clears `reserved_from` regardless of remaining orders on the table — same latent multi-order bug 3a/3b just fixed for Manager. Tracked for future cleanup, not blocking.

All three fixes manually verified live by Ayush and pushed to GitHub.

## New Issue Discovered This Session (Out of Node 3 Scope)

**Reservation double-booking:** Two separate Reservation Requests approved for the same table with overlapping time windows (e.g. Table 1, Seats 2 — "Ak" party approved for 3:55 PM and "Ar" party approved for 3:45 PM, both on the same table). Approval flow does not check for existing active reservations/occupancy on the target table before allowing approval. Observed side effect: table's `seated` count went to `4/2` (exceeding capacity).

This is scoped as **Node 4** — see `02_Instructions/` for investigation prompt once started.

## Next Action

Start Node 4 investigation: reservation approval flow — why no overlap/capacity check exists before approving a reservation request onto an already-reserved/occupied table.
