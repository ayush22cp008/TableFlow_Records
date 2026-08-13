# Instruction — Confirmed Root Cause: Stale reservation_requests Rows (Node 4, Regression)

**Chat #9 — Node 4 — Investigation + scoping only. Do NOT write any fix yet.**

## Confirmed root cause (from Ayush, not a hypothesis anymore)
While testing the Mark Paid flow earlier, Ayush ran this raw SQL directly in Supabase SQL Editor (no WHERE clause — applies to ALL rows):
```sql
UPDATE restaurant_tables
SET occupied_seats = 0,
    status = 'available',
    reserved_from = NULL;
```
This reset **every row in `restaurant_tables`** back to available/empty. It touched **only `restaurant_tables`** — it did NOT touch `reservation_requests` at all.

Result: `restaurant_tables` now correctly shows Table 1/2/3 as available (0 seated, no `reserved_from`). But `reservation_requests` still has old rows with `status = 'approved'` and `table_id` pointing to Table 1/2/3 from before this reset — those rows were never updated because the SQL only hit `restaurant_tables`.

The new overlap check in `approveRequest`:
```js
reservationRequests.some(r => r.status === 'approved' && r.table_id === tableId && r.id !== req.id)
```
only reads `reservation_requests.status`, so it sees these stale `approved` rows and wrongly blocks approval on tables that are actually free per `restaurant_tables`.

**This confirms the dual-source-of-truth problem**: `restaurant_tables` (live occupancy) and `reservation_requests.status` (`approved`) can now disagree, and the overlap check only trusts the second one.

## What to investigate / scope (no fix yet)

1. Query `reservation_requests` for all rows with `status = 'approved'` where the associated table is NOT currently occupied/reserved per `restaurant_tables` (i.e. `occupied_seats = 0` and `reserved_from IS NULL` for that `table_id`). Report count and which tables/rows are affected.
2. Confirm: is `approved` currently a terminal status in `reservation_requests`, or does the schema have (or need) a further status like `completed`/`seated`/`released` to mark a reservation as no longer active once the party is done?
3. Report how the overlap check *should* determine "active" — options to consider (don't implement, just lay out): (a) check live `restaurant_tables` state instead of/in addition to `reservation_requests.status`, (b) add a proper lifecycle status to `reservation_requests` and update it wherever a table gets released (Clear button, Mark Paid RPC, this ad-hoc reset), (c) both.

## Constraint
This is a scoping/investigation prompt only. Do not modify `approveRequest`, `reservation_requests`, or `restaurant_tables` in this step. Report back findings and the options above — fix will be written as a separate instruction after Ayush decides the approach.
