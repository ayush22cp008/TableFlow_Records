# Log — Node 10 Fix A: Reservation RLS Security Gap

**Date:** 2026-08-11
**Run by:** Ayush (manual, Supabase SQL Editor)
**Status:** VERIFIED — "Success. No rows returned"

## SQL executed
```sql
-- 1. Drop the existing open public policy
DROP POLICY IF EXISTS "Allow public update" ON reservation_requests;

-- 2. Create a new UPDATE policy scoped to manager role only
CREATE POLICY "reservation_requests_manager_update" ON reservation_requests
FOR UPDATE
USING (has_role(ARRAY['manager']))
WITH CHECK (has_role(ARRAY['manager']));
```

## Effect
- Closed the open public UPDATE policy on `reservation_requests` (previously `USING (true)` — any request could update it).
- Reservation approve/reject now scoped to `manager` role only at the DB level.
- Frontend (`app/dashboard/tables/page.tsx`) already gated to `role === 'manager'` per Fix A instruction — Owner is read-only.

## Rollback (if needed post-deploy)
```sql
DROP POLICY IF EXISTS "reservation_requests_manager_update" ON reservation_requests;

CREATE POLICY "Allow public update" ON reservation_requests
FOR UPDATE TO public USING (true);
```

## Next
GitHub push pending — Ayush's explicit go-ahead required before Antigravity pushes.
