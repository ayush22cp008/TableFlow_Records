# Instruction — Verify 30-min Grace Window Expiry (Manual DB Test)

Chat #1 | Node: Bug Fix Verification — Auto-Hide (no-show/expiry case)

## Context

Track A + Auto-Hide fix both confirmed working for the "order completed → hides 5 min after bill" case. Need to verify the second case: an approved reservation with NO order placed should (a) stay code-valid until requested_time + 30 min, then the code should become invalid, and (b) the request should disappear from the dashboard panel once expired.

## Task

Simulate the 30-minute expiry without waiting in real time, by manually backdating a test reservation's `requested_time` in Supabase.

## Steps

1. Pick or create a test reservation_requests row with status = 'approved' and NO linked order.
2. In Supabase, manually update that row's `requested_time` to **40 minutes in the past** from current time (so it's already 10 minutes past the 30-min grace window).
3. Refresh `/dashboard/tables` — confirm the request has disappeared from the "Reservation Requests" panel.
4. Try using that reservation's code in `/order/cart` — confirm it shows "Invalid or unused code" (or equivalent expired-code error), same as before.
5. Report: which row was used, before/after `requested_time` values, and both results (panel visibility + code validity).

## Do Not

- Do not modify any other reservation rows.
- Do not touch code — this is a DB-level manual test only, no code changes.
