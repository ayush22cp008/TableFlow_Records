# Instruction — Verify Root Cause: Stale reservation_requests Rows (Node 4, Regression)

**Chat #9 — Node 4 — Investigation + scoping only. Do NOT write any fix yet.**

## Strong hypothesis from Ayush (needs DB verification, not yet confirmed)
While testing the Mark Paid flow earlier, Ayush ran this raw SQL directly in Supabase SQL Editor (no WHERE clause — applies to ALL rows):
```sql
UPDATE restaurant_tables
SET occupied_seats = 0,
    status = 'available',
    reserved_from = NULL;
```
This touched **only `restaurant_tables`** — it did NOT touch `reservation_requests`.

Hypothesis: `restaurant_tables` now correctly shows Table 1/2/3 as available, but `reservation_requests` still has old rows with `status = 'approved'` and `table_id` pointing to Table 1/2/3 from before this reset. The new overlap check in `approveRequest`:
```js
reservationRequests.some(r => r.status === 'approved' && r.table_id === tableId && r.id !== req.id)
```
only reads `reservation_requests.status`, so if stale `approved` rows exist, it would wrongly block approval on tables that are actually free per `restaurant_tables`.

**This is not yet confirmed — verify against actual DB data first, before assuming this is the cause.**

## What to investigate / verify (no fix yet)

1. Run a query against `reservation_requests` for all rows with `status = 'approved'`. For each, cross-check the associated `table_id` against `restaurant_tables` — is `occupied_seats = 0` and `reserved_from IS NULL` for that table right now? Report exact rows where this mismatch exists (stale approved request vs. actually-free table).
2. Specifically confirm: do Table 1, 2, 3's UUIDs appear as `table_id` in any `reservation_requests` row with `status = 'approved'`? Report those exact rows (id, table_id, status, created/updated timestamps if available).
3. If the mismatch is NOT found — i.e. no stale approved rows exist for Table 1/2/3 — report that clearly and flag that the false-positive block has a different cause; re-examine the `approveRequest` overlap check logic itself (e.g. check if `table_id` comparison has a type/format mismatch, or if the `reservationRequests` array in state is stale/not refreshed after the DB reset).
4. If confirmed: report whether `reservation_requests` schema has (or needs) a lifecycle status beyond `approved` (e.g. `completed`/`seated`/`released`) to distinguish "still active" from "done", since `approved` currently seems to be treated as permanent.
5. Report options (don't implement, just lay out) for how the overlap check *should* determine "active": (a) check live `restaurant_tables` state instead of/in addition to `reservation_requests.status`, (b) add a proper lifecycle status to `reservation_requests` and ensure it's updated wherever a table gets released (Clear button, Mark Paid RPC, manual SQL resets), (c) both.

## Constraint
This is a verification/scoping prompt only. Do not modify `approveRequest`, `reservation_requests`, or `restaurant_tables` in this step. Report back actual DB findings (not assumptions) plus the options above — fix will be written as a separate instruction after Ayush decides the approach.
