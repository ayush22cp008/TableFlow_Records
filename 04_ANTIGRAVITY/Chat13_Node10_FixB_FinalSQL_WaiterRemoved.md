# Fix B — Final SQL (Waiter Removed from Tables/Waitlist)

**Context:** Fix B result had `waiter` in tables_write/waitlist_update (pre-existing, not new). Confirmed via Waiter Dashboard UI screenshot that Waiter only touches `orders` (Mark Served flow) — never `restaurant_tables` or `waitlist`. So narrowed both policies to `manager` only, least-privilege.

## Run this in Supabase SQL Editor
```sql
-- 1. Orders: drop owner's access completely
DROP POLICY IF EXISTS "owner_all_updates" ON orders;

-- 2. Tables: manager only
DROP POLICY IF EXISTS "tables_write" ON restaurant_tables;
CREATE POLICY "tables_write_staff" ON restaurant_tables FOR ALL
USING (has_role(ARRAY['manager']))
WITH CHECK (has_role(ARRAY['manager']));

-- 3. Waitlist: manager only
DROP POLICY IF EXISTS "waitlist_update" ON waitlist;
CREATE POLICY "waitlist_update_staff" ON waitlist FOR UPDATE
USING (has_role(ARRAY['manager']));
```

## Rollback (if needed post-deploy — restores original waiter+manager+owner access)
```sql
-- Orders
DROP POLICY IF EXISTS "owner_all_updates" ON orders;
CREATE POLICY "owner_all_updates" ON orders FOR UPDATE
USING ( has_role(ARRAY['owner']) )
WITH CHECK ( has_role(ARRAY['owner']) );

-- Tables
DROP POLICY IF EXISTS "tables_write_staff" ON restaurant_tables;
CREATE POLICY "tables_write" ON restaurant_tables FOR ALL
USING (has_role(ARRAY['waiter', 'manager', 'owner']))
WITH CHECK (has_role(ARRAY['waiter', 'manager', 'owner']));

-- Waitlist
DROP POLICY IF EXISTS "waitlist_update_staff" ON waitlist;
CREATE POLICY "waitlist_update" ON waitlist FOR UPDATE
USING (has_role(ARRAY['waiter', 'manager', 'owner']));
```

Frontend already correctly gated in Fix B (`role === 'manager'` on tables.tsx, Owner blocked on orders/billing) — no code changes needed for this correction, DB-only.
