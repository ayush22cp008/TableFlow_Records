# Node 10 - Fix A Result: Reservation RLS Security Gap

## 1. RLS Policy Changes (Manual SQL)
The following SQL needs to be run manually in Supabase to secure the `reservation_requests` table.

```sql
-- 1. Drop the existing open public policy
DROP POLICY IF EXISTS "Allow public update" ON reservation_requests;

-- 2. Create a new UPDATE policy scoped to manager role only
CREATE POLICY "reservation_requests_manager_update" ON reservation_requests 
FOR UPDATE 
USING (has_role(ARRAY['manager'])) 
WITH CHECK (has_role(ARRAY['manager']));
```

## 2. Frontend Changes
- Modified `app/dashboard/tables/page.tsx`.
- Imported `useAuth` and extracted `role`.
- Wrapped the "Approve" and "Reject" buttons (and the table selection dropdown) inside the pending reservations map with `{role === 'manager' && ( ... )}` so that only the Manager role can see and click them. Owner will now only see the pending requests (read-only).

## 3. Build Result
- `npm run build` ran successfully.
- No compile or linting errors.

## 4. Rollback SQL (If needed)
If this breaks reservation approve/reject after deploy, run this in Supabase SQL editor:

```sql
-- Revert: drop the new scoped policy and restore the original open policy
DROP POLICY IF EXISTS "reservation_requests_manager_update" ON reservation_requests;

CREATE POLICY "Allow public update" ON reservation_requests
FOR UPDATE TO public USING (true);
```

**Status:** Fix complete locally. Ready for Ayush to run the manual SQL in Supabase, review, and push to GitHub.
