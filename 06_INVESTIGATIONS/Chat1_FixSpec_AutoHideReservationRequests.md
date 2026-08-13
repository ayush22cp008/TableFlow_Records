# Fix Spec — Auto-Hide Reservation Requests After Completion

Chat #1 | Node: Bug Fix — follow-up to Track A (reservation code flow)

## Context

Track A (Confirm Arrival removal) is confirmed working on live site. New small gap found: approved reservation requests stay visible in the "Reservation Requests" panel on `/dashboard/tables` indefinitely, even after the customer has already placed their order and the bill was generated.

## Decided Fix

Reservation requests should disappear from the panel under two conditions:

1. **Order completed (bill generated):** Once the reservation's linked order is completed and a bill is generated, hide the request from the panel **5 minutes after** that completion/bill-generation timestamp.
2. **No-show (customer never ordered):** If no order was ever placed, the request stays visible until the reservation code itself expires (per Track A's existing logic — `requested_time` + 30 min grace window). Once expired, it should no longer show either.

## Scope

- This is a **panel visibility change only** — filter which `reservation_requests` rows are queried/rendered on `/dashboard/tables`. Underlying DB status/fields do not need a new state; use existing timestamps (order completion time, `requested_time` + grace window) to compute visibility at query/render time.
- Do NOT change anything else: approval flow, code validation logic, table assignment, billing — all unchanged (already fixed/working from Track A).
- Do NOT start Track B (priority/numbering) — separate task.

## Files Likely Involved

- `app/dashboard/tables/page.tsx` — the reservation requests panel query/render logic (same file Track A touched).
- Need to check: is there a linked `orders` or `bills` table with a completion timestamp that can be joined/queried against `reservation_requests`? Investigate this if not already known before implementing.

## Engineering Discipline Reminders

- No dual source of truth — compute visibility from existing timestamps, don't add a redundant "hidden" flag if a timestamp comparison already achieves it.
- Evidence required before calling this fixed: test with (a) a reservation whose order completes — confirm it panel-hides ~5 min after, and (b) an approved reservation with no order — confirm it stays until code expiry, then hides.
