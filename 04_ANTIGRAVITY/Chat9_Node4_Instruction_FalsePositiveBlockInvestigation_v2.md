# Instruction — Investigate False-Positive Block on Available Tables (Node 4, Regression)

**Chat #9 — Node 4 — Investigation only. Do NOT write any fix.**

## Bug observed (post-fix regression)
After Fix 1 (approve-time guards) was deployed and tested locally by Ayush:
- Table 1, 2, 3 show as "Available" on the dashboard (0 seated, no `reserved_from`) — no active reservation on them.
- Ayush made NO new reservation request on these tables.
- Yet approving ANY reservation request (tested with "Yash" → Table 2, "At" → Table 1, "At" → Table 2, "At" → Table 3) throws: **"This table already has an active reservation assigned."**
- Only Table 4 and Table 5 are genuinely reserved/occupied — those correctly show as unavailable in the dropdown (expected, not the bug).
- Dropdown filtering itself (which tables show up as options based on available seats) is working correctly — confirmed by Ayush. The bug is isolated to the **overlap check inside `approveRequest`**, which blocks Table 1/2/3 even though they are free.

## New lead from Ayush (important — investigate this first)
Ayush recalls that earlier during testing, he used the **"Clear" button** on the Tables dashboard to forcefully reset a table's status from Reserved → Available (this Clear button is visible on Table 4/5 style cards). His suspicion: **the Clear action may only update `restaurant_tables` (UI-facing state) and NOT update the corresponding `reservation_requests.status`** — leaving a stale `status = 'approved'` row behind for that table, tied to a now-cleared/completed reservation.

Since the new overlap check in `approveRequest` is:
```js
reservationRequests.some(r => r.status === 'approved' && r.table_id === tableId && r.id !== req.id)
```
...it purely checks `reservation_requests.status`, not the live table state (`restaurant_tables.reserved_from` / `occupied_seats`). If Clear doesn't also update `reservation_requests.status`, this exact mismatch would explain the false-positive block on Table 1/2/3.

## Investigate

1. **Locate the "Clear" button handler** (likely also in `app/dashboard/tables/page.tsx`, near `approveRequest`). Report the exact function and what it updates — confirm whether it touches `restaurant_tables` only, or also updates `reservation_requests.status` for the associated request(s).
2. **Inspect `reservation_requests` rows for Table 1, 2, 3's UUIDs**: are there rows with `status = 'approved'` and `table_id` pointing to these tables, left over from before a Clear action? Report the exact rows/data found.
3. Confirm: does `reservation_requests` have any status value other than `pending`/`approved`/`rejected` (e.g. `completed`, `seated`, `cancelled`) that Clear *should* be setting but currently isn't?
4. Based on 1-3, confirm or reject Ayush's hypothesis: **Clear updates `restaurant_tables` but not `reservation_requests.status`, leaving stale `approved` rows that the new overlap check incorrectly treats as still active.**

## Output required
Findings only — confirm/reject the hypothesis with exact code + data evidence. No fix. Report back before any fix is proposed.
