# Waiter Navbar — Remove ALL Nav Links (Fix Instruction, v2)

**Supersedes:** `Chat7_Node_Waiter_Fix_NavbarScopeReduction.md` (v1) — v1 mistakenly kept "Overview". Corrected here: waiter has exactly one page (the orders queue), so no nav links are needed at all, not even a self-link to that same page.

**Based on:** `Chat7_Node_Waiter_ScopeOverride_Node1Matrix.md` — Waiter scope reduced to orders queue only.

## Fix — `components/Navbar.tsx`

**Current (per `Chat7_Node_Waiter_Implementation_Result.md`):** Waiter role sees **Overview, Menu, Tables** links.

**Required:** Remove ALL nav links for the `waiter` role — Overview, Menu, and Tables. Only the TableFlow logo/title and **Sign Out** should remain for waiter. There is nothing to navigate to since the waiter dashboard IS the landing page after login — a self-referencing "Overview" link serves no purpose.

Do not touch nav links for any other role (owner/manager/cook all keep their existing links, unchanged).

## Safety check (no code change expected, just confirm)

Confirm no `waiter`-specific RLS policies exist on `tables` or `reservation_requests` that grant write access. If any do exist from earlier work, flag them to Ayush — do not silently drop RLS policies without approval, this instruction only covers removing UI nav entries.

## Verification

1. `npm run build` — must compile clean.
2. Ayush to manually confirm: waiter login → Navbar shows only the TableFlow logo/title and Sign Out. No Overview, no Menu, no Tables.
