# Instruction — Chat 14, Node 10, Fix C (Menu — reverse case)

## Task
Menu access is the REVERSE of Fix A/B: **Owner keeps** `is_available` toggle access, **Manager loses** it.

## Frontend change
File: `app/dashboard/menu/page.tsx` (or shared menu component, same pattern as Tables/Orders/Waitlist in Fix B)

- Hide/disable the `is_available` toggle control for **Manager** role.
- Keep it fully visible and functional for **Owner** role.
- Do not touch any other Menu functionality (item CRUD, pricing, etc. — unless those were already role-gated elsewhere; if unsure, ask before changing).

## RLS change (Supabase SQL Editor — manual run by Ayush)

**Forward SQL:**
```sql
-- Menu: owner only (reverse of other Fix B areas)
DROP POLICY IF EXISTS "menu_write" ON menu_items;
CREATE POLICY "menu_write_owner" ON menu_items FOR ALL
USING (has_role(ARRAY['owner']))
WITH CHECK (has_role(ARRAY['owner']));
```

**Rollback SQL (if post-deploy issues):**
```sql
DROP POLICY IF EXISTS "menu_write_owner" ON menu_items;
CREATE POLICY "menu_write" ON menu_items FOR ALL
USING (has_role(ARRAY['owner', 'manager']))
WITH CHECK (has_role(ARRAY['owner', 'manager']));
```

> Note: confirm actual table name (`menu_items` assumed) and existing policy name/roles before running — verify via Supabase dashboard if `menu_write` differs from what's shown here. If the current policy already includes other roles (e.g. `waiter`), do not drop their access — only remove `manager`.

## Verification checklist (Ayush, manual browser test — screenshot each)
- [ ] Owner: Menu page shows `is_available` toggle, fully functional
- [ ] Manager: Menu page — toggle hidden/disabled, no way to change availability
- [ ] Waiter/Cook/Customer: unaffected (confirm no regression if they touch Menu at all)
- [ ] Owner directly hitting any menu edit route (if one exists beyond the toggle) — should work
- [ ] Manager directly hitting same route (if applicable) — should show Access Restricted or equivalent

## Standing rules (unchanged)
- Investigation and fix are separate prompts — this is fix-only; if the current policy state is unclear, investigate first before running SQL.
- No GitHub push without Ayush's explicit go-ahead after verification.
- Manual DB change must be logged in `04_Logs/` immediately after execution.
- Antigravity: code execution + build/compile check only. No browser testing.
