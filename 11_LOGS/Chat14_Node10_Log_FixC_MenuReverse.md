# Manual DB Change Log: Fix C - Menu Reverse

- **Timestamp:** 2026-08-11T23:45:00+05:30
- **Table:** `menu_items`
- **Change:** Dropped `"menu_write"` and replaced with `"menu_write_owner"` (Owner only).
- **Reason:** Node 10 (Fix C) — Owner keeps access to toggle menu item availability, but Manager loses write access to the menu table.

## SQL to run manually:
```sql
-- Menu: owner only (reverse of other Fix B areas)
DROP POLICY IF EXISTS "menu_write" ON menu_items;
CREATE POLICY "menu_write_owner" ON menu_items FOR ALL
USING (has_role(ARRAY['owner']))
WITH CHECK (has_role(ARRAY['owner']));
```
