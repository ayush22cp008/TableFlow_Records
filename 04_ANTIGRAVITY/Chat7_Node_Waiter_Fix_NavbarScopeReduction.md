# Waiter Navbar — Remove Menu/Tables Links (Fix Instruction)

**Based on:** `Chat7_Node_Waiter_ScopeOverride_Node1Matrix.md` — Waiter scope reduced to orders queue only.

## Fix — `components/Navbar.tsx`

**Current (per `Chat7_Node_Waiter_Implementation_Result.md`):** Waiter role sees **Overview, Menu, Tables** links.

**Required:** Waiter role should see **only Overview** (the waiter dashboard link). Remove the `Menu` and `Tables` nav entries for the `waiter` role specifically — do not touch these links for any other role (owner/manager/cook all still need their existing links, unchanged).

## Safety check (no code change expected, just confirm)

Confirm no `waiter`-specific RLS policies exist on `tables` or `reservation_requests` tables that grant write access. If any do exist from earlier work, flag them to Ayush — do not silently drop RLS policies without approval, this instruction only covers removing the UI nav entries.

## Verification

1. `npm run build` — must compile clean.
2. Ayush to manually confirm: waiter login → Navbar shows only Overview + Sign Out, no Menu, no Tables link.
