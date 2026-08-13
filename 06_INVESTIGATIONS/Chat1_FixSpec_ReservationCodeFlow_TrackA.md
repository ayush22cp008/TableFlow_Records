# Fix Spec — Reservation Code Flow (Confirm Arrival Removal)

Chat #1 | Node: Bug Fix — Stale Reservation Requests (resolves earlier investigation)

## Root Cause (confirmed via investigation)

- `/dashboard/tables` fetches all `reservation_requests` with status `pending`/`approved`, no time or completion filter.
- Cart requires status = `'arrived'` to accept a reservation code — only set via manual "Confirm Arrival" click by owner.
- If customer orders as walk-in instead of using the code (e.g. because arrival was never confirmed), the reservation is orphaned at `approved` forever — no expiry, no cleanup.

## Decided Fix — Track A (this fix only)

1. **Remove "Confirm Arrival" entirely** — no button, no `arrived` status step.
2. Cart code validation changes from `status = 'arrived'` to:
   - `status = 'approved'`
   - AND current time is before `requested_time + 30 minutes` (grace window)
3. If code entered after `requested_time + 30 minutes` → treat as invalid/expired (same error UX as invalid code today).
4. On successful order placement with valid code → status still moves to `'completed'` (unchanged, already works this way).
5. Approval flow (owner approving a reservation request) — unchanged.
6. Table assignment / billing — unchanged.

## Explicitly Out of Scope (this fix)

- Order priority/reservation-tagging — separate track (Track B), do not touch order queue logic in this fix.
- Sequential daily order numbering — separate track (Track B).
- Any UI for manually dismissing/expiring old requests — not needed, expiry is now automatic via the time check.

## Files Known to Be Involved (from investigation)

- `app/dashboard/tables/page.tsx` — remove Confirm Arrival button/handler, remove `arrived`-status dependency in UI.
- `app/order/cart/page.tsx` — update code validation query (line ~82) to new status+time logic; completion trigger (line ~181) stays as-is.

## Engineering Discipline Reminders

- Build on isolated git branch, don't touch main stable flow beyond what's listed above.
- No dual source of truth: reservation validity is now purely `status='approved'` + time check — do not introduce a second flag for "arrived".
- Evidence required before calling this fixed: test with a code used before the grace window (should work), and after grace window (should show invalid/expired).
