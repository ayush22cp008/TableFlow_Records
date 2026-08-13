# Investigation — Stale Reservation Requests Not Clearing

Chat #1 | Node: Bug Investigation (Reservation Label bug — follow-up)

## Task

Investigate only. No fixes, no code changes.

## Observed Evidence (screenshots, dashboard)

1. `/dashboard/tables`: All 6 tables show "Available", 0 seated — no table currently occupied.
2. `/dashboard/tables`, "Reservation Requests (2)" panel: two old reservation requests ("viral" — Party of 1, "ayushtest" — Party of 3) still show status "Approved" with an active "Confirm Arrival" button and "Enter code" field.
3. User confirms (manual knowledge, not from UI): both of these reservations are old — their full lifecycle already completed at the time: order was placed, order was completed, and a bill was already generated.
4. `/dashboard/orders`: Live Orders board (Order Placed / Preparing / Ready / Served) is completely empty — 0 in every column.

## What This Means (to verify, not assumed)

Reservation request records whose full lifecycle (arrival → order → completed → billed) already finished in the past are still appearing in the active "Reservation Requests" panel as "Approved" with a pending "Confirm Arrival" action — as if arrival was never confirmed.

## Investigate

1. Find the query/component that populates the "Reservation Requests" panel on `/dashboard/tables`. What filter (if any) determines which reservation_requests rows appear here — status-based, date-based, or unfiltered?
2. For the two example reservations ("viral" party of 1, "ayushtest" party of 3): query their actual current row state in `reservation_requests` (and any linked `orders` / `bills` table) — status field, timestamps, and whether arrival/order/bill fields are populated.
3. Check whether "Confirm Arrival" was ever actually triggered for these two, or whether the order+bill was created through a separate path that bypassed the arrival-confirmation flow entirely.
4. Determine why a reservation with a completed order + generated bill still shows as "Approved" / pending arrival — is there no status transition (e.g. to "completed"/"closed") after billing, or is the panel querying without a completion filter?
5. Report exact file paths, relevant line numbers, and current field values found for these two records.

## Do Not

- Do not modify any files.
- Do not write a fix. This is investigation only.
- Do not assume the cause — report what the code and data actually show.
