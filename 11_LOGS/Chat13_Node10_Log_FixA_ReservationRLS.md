# Manual DB Change Log: Reservation RLS Security Fix

- **Timestamp:** 2026-08-11T16:30:00+05:30
- **Table:** `reservation_requests`
- **Change:** 
  1. Dropped `"Allow public update"` policy.
  2. Created `"reservation_requests_manager_update"` policy.
- **Reason:** Node 10 (Fix A) — Prevent unauthenticated or unauthorized users from approving/rejecting reservation requests, restricting this action strictly to managers.

## SQL to run manually:
```sql
-- 1. Drop the existing "Allow public update" policy
DROP POLICY IF EXISTS "Allow public update" ON reservation_requests;

-- 2. Create a new UPDATE policy scoped to manager role only
CREATE POLICY "reservation_requests_manager_update" ON reservation_requests 
FOR UPDATE 
USING (has_role(ARRAY['manager'])) 
WITH CHECK (has_role(ARRAY['manager']));
```
