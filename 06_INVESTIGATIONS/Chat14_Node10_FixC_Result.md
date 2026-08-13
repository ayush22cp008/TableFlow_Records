# Node 10 - Fix C Result: Menu Reverse Case

## 1. RLS Policy Changes (Manual SQL)

The following SQL must be run manually in Supabase to restrict the Manager role from making direct DB updates to the `menu_items` table. The Owner keeps write access.

```sql
-- 1. Menu: owner only (reverse of other Fix B areas)
DROP POLICY IF EXISTS "menu_write" ON menu_items;
CREATE POLICY "menu_write_owner" ON menu_items FOR ALL
USING (has_role(ARRAY['owner']))
WITH CHECK (has_role(ARRAY['owner']));
```

## 2. Frontend Changes Confirmed
The following file was updated to apply conditional rendering so that operations are disabled or hidden when the user is not an 'owner':
- **`app/dashboard/menu/page.tsx`**: The `is_available` toggle button is now strictly wrapped with `{role === 'owner' && ...}` so that only the Owner can see and use it to toggle item availability. Other roles like Manager, Waiter, Cook will no longer see the toggle button.

## 3. Build Result
- `npm run build` ran successfully.
- No compile or linting errors.

## 4. Rollback SQL (If needed)

```sql
-- Menu
DROP POLICY IF EXISTS "menu_write_owner" ON menu_items;
CREATE POLICY "menu_write" ON menu_items FOR ALL
USING (has_role(ARRAY['owner', 'manager']))
WITH CHECK (has_role(ARRAY['owner', 'manager']));
```

**Status:** Fix complete locally. Ready for Ayush to run the manual SQL in Supabase, review in the browser, and then push to GitHub.
