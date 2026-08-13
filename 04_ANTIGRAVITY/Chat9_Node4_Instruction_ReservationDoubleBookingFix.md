# Instruction — Fix Reservation Double-Booking (Node 4)

**Chat #9 — Node 4 — Fix only. Investigation already complete (see `Chat9_Node4_Investigation_ReservationDoubleBooking.md`). Do not re-investigate.**

## Root cause (confirmed)
1. `approveRequest()` in `app/dashboard/tables/page.tsx` has no overlap check and no occupancy check — it blindly overwrites `restaurant_tables.reserved_from` on every approval, so two overlapping reservations can both get approved for the same table.
2. `place_order_and_occupy_table` RPC blindly does `occupied_seats = occupied_seats + p_party_size` with no hard cap against `capacity`, so seated count can exceed table capacity (e.g. 4/2).

Two independent gaps → two fixes required.

## Fix 1 — Approve-time guard (`approveRequest`, `app/dashboard/tables/page.tsx`)

Before approving, check the target table for:
- **Occupancy check:** `occupied_seats + req.party_size <= capacity`
- **Overlap check:** no other `reservation_requests` row with `status = 'approved'` already assigned to this `table_id` with an unconsumed/active reservation (i.e. not yet checked in / not yet released)

If either check fails, block the approval and show a clear error (e.g. "Table already reserved/occupied for this time" or "Not enough seats available") instead of overwriting `reserved_from`.

Also fix the table-selection dropdown filter to account for current `occupied_seats`, not just absolute `capacity` (currently only checks `t.capacity >= req.party_size`).

## Fix 2 — RPC hard cap (`place_order_and_occupy_table`)

Add a server-side guard inside the RPC: before incrementing `occupied_seats`, check `occupied_seats + p_party_size <= capacity`. If it would exceed, raise an error / abort the transaction instead of allowing an overshoot like 4/2.

## Constraints
- Keep both fixes minimal — no unrelated refactors.
- Do not touch Node 3 (Manager Dashboard) — it's LOCKED.
- No manual UI testing on your side — Ayush will test manually and confirm before this gets pushed.
- Report back: files touched, exact diff/snippet of each fix, and build/compile result.
