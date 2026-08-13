# Instruction — Fix: Reservation Requests Panel Clutter (Option C)

**Chat #9 — Fix only. Investigation already complete (see `Chat9_Investigation_ReservationRequestsPanel.md`). Do not re-investigate.**

## Confirmed findings (context)
- Query fetches all `pending`/`approved`/`completed` rows, no date/limit filter — currently returns up to 50 rows.
- Client-side `visibleReservations` filter keeps `completed` ("Seated") entries visible for the entire duration of their linked order, only hiding them 5 min after the order is `billed`. This has no functional purpose — staff use the Tables grid and Orders dashboard for active meal management, not this panel.

## Fix — both parts

### Part 1: Query-level date window
Add a date/time filter to the `reservation_requests` fetch in `app/dashboard/tables/page.tsx` so it only pulls **today's** requests (based on `requested_time`), instead of fetching unbounded historical rows. Keep the existing `.in('status', [...])` and `.order('requested_time')` as-is — just add the date bound.

### Part 2: Client-side filter fix
Update the `visibleReservations` logic so `completed` ("Seated") entries are hidden from the panel shortly after becoming `completed` — do NOT wait for the linked order to reach `billed`. Suggested: hide `completed` entries ~5-10 minutes after their `status` transitions to `completed`, independent of order/billing state. Keep `pending` and `approved` visibility logic unchanged (not in scope here).

## Constraints
- Do not touch Node 3 (Manager Dashboard) or Node 4 logic (overlap check, lifecycle status transitions) — both LOCKED, not in scope.
- Do not delete or modify any `reservation_requests` rows in the DB — this is a display/query filter only, not a data cleanup.
- Keep the fix minimal — no unrelated refactors.
- No manual UI testing on your side — Ayush will test manually and confirm before push.
- Report back: files touched, exact diff/snippet for both parts, and build/compile result.
