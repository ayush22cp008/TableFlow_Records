# Instruction: Add customer_id to reservation_requests (DB Migration)

**Node:** 8 — Customer Dashboard Revamp
**Type:** FIX — Manual SQL migration only. No frontend code changes in this step.

## Context
`reservation_requests` currently has no link to the logged-in user — only a plain-text `customer_name`. App is fully login-gated (no anonymous access anywhere), so every reservation is created by an authenticated user. We're adding `customer_id` to make future reservations linkable to the customer's own dashboard.

## Change required

Run in Supabase SQL Editor (manual — CLI unavailable on Windows):

```sql
ALTER TABLE reservation_requests
ADD COLUMN customer_id UUID REFERENCES profiles(id);
```

- No NOT NULL constraint — existing rows will have `customer_id = NULL`, which is expected and acceptable (old reservations stay unlinked, no backfill needed).
- No default value needed.

## Permissions check
Confirm RLS policies on `reservation_requests` allow the customer to insert their own `customer_id` (should be covered by existing insert policy, but verify — per project rule, permissions must be set explicitly before this is used in code).

## Log requirement
Per project rule, log this manual DB change immediately in `04_Logs/` — a short entry noting table, column added, timestamp, reason (Node 8 reservation-customer linking).

## Output
Confirm column added successfully (screenshot or query result showing new schema), confirm RLS check result, and confirm the log entry was created. No frontend changes yet — that's a separate instruction.
