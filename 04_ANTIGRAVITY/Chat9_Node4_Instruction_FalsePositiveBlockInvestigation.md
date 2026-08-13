# Instruction — Investigate False-Positive Block on Available Tables (Node 4, Regression)

**Chat #9 — Node 4 — Investigation only. Do NOT write any fix.**

## Bug observed (post-fix regression)
After Fix 1 (approve-time guards) was deployed and tested locally by Ayush:
- Table 1, 2, 3 show as "Available" on the dashboard (0 seated, no `reserved_from`) — no active reservation on them.
- Ayush made NO new reservation request on these tables.
- Yet approving ANY reservation request (tested with "Yash" → Table 2, "At" → Table 1, "At" → Table 2, "At" → Table 3) throws: **"This table already has an active reservation assigned."**
- Only Table 4 and Table 5 are genuinely reserved/occupied — those correctly show as unavailable in the dropdown (expected, not the bug).
- The bug is that Table 1, 2, 3 — which ARE free — are being wrongly blocked at approval time.

## Suspected root cause (hypothesis, not yet confirmed)
The overlap check added in `approveRequest`:
```js
reservationRequests.some(r => r.status === 'approved' && r.table_id === tableId && r.id !== req.id)
```
This scans the **entire `reservationRequests` list** (currently 30-31 requests) for any row with `status === 'approved'` and matching `table_id` — regardless of whether that reservation is still active/current or already completed/stale (e.g. guest already seated, ordered, paid, and table released back to available).

If any historical approved request has `table_id` pointing to Table 1/2/3 from an earlier (now-completed) cycle, this check will treat it as still "active" even though the table's actual live state (`occupied_seats`, `reserved_from` on `restaurant_tables`) shows it as free.

## Investigate

1. Confirm: does the overlap check filter on reservation *recency/currency* at all, or purely on `status === 'approved'`? Report exact current code.
2. Query/inspect `reservation_requests` table: are there approved rows with `table_id` = Table 1, 2, or 3's UUID from earlier in testing (now stale, table already freed)?
3. Confirm whether there's any field that distinguishes "approved and currently occupying/reserved" vs "approved and already completed/released" (e.g. does `reservation_requests` have a status like `completed`/`seated`/`cancelled`, or is `approved` the terminal status forever?).
4. Report whether `restaurant_tables.reserved_from` being null/empty for Table 1/2/3 (i.e. genuinely free) is being checked anywhere, or if the block is purely driven by stale `reservation_requests` rows.

## Output required
Findings only — confirm or reject the hypothesis, report exact data state for Table 1/2/3's related `reservation_requests` rows, and clarify what "active" should mean for this check. No fix. Report back before any fix is proposed.
