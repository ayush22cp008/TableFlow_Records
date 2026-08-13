# Manual DB Change Log: Fix B - Owner Read-Only

- **Timestamp:** 2026-08-11T17:31:00+05:30
- **Tables:** `orders`, `restaurant_tables`, `waitlist`
- **Change:** 
  1. Dropped `"owner_all_updates"` on `orders`.
  2. Dropped `"tables_write"` and replaced with `"tables_write_staff"` (Waiters/Managers only).
  3. Dropped `"waitlist_update"` and replaced with `"waitlist_update_staff"` (Waiters/Managers only).
- **Reason:** Node 10 (Fix B) — Owner becomes read-only for day-to-day operations to avoid race conditions and overlap with Manager/Staff. Bulk emergency RPC bypasses RLS and is unaffected.

## SQL to run manually:
```sql
-- 1. Orders: Drop owner's all_updates policy completely
DROP POLICY IF EXISTS "owner_all_updates" ON orders;

-- 2. Tables: Replace original policy with one scoped to Waiter and Manager only
DROP POLICY IF EXISTS "tables_write" ON restaurant_tables;
CREATE POLICY "tables_write_staff" ON restaurant_tables FOR ALL
USING (has_role(ARRAY['waiter', 'manager'])) 
WITH CHECK (has_role(ARRAY['waiter', 'manager']));

-- 3. Waitlist: Replace original policy with one scoped to Waiter and Manager only
DROP POLICY IF EXISTS "waitlist_update" ON waitlist;
CREATE POLICY "waitlist_update_staff" ON waitlist FOR UPDATE 
USING (has_role(ARRAY['waiter', 'manager']));
```
