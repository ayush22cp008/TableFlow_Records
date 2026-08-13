# Instruction — Chat 14, Node 10, Fix C (continued) — Hide Menu nav/route for Manager

## Problem
Fix C made Menu owner-only via RLS + hid the `is_available` toggle for Manager. But Manager still sees the full "Menu" nav link and can open `/dashboard/menu`, where Edit/Remove buttons are visible but broken (RLS blocks the writes) and "Add Item" surfaces a raw Postgres RLS error to the user. Confusing, unprofessional UX.

## Decision
Since Menu is now Owner-exclusive, remove Manager's access to the Menu section entirely — nav link and route both — rather than showing a partially-broken page.

## Frontend changes
1. **Nav link**: In the main dashboard nav component (wherever "Overview / Menu / Tables / Orders / Analytics / AI Insights / Staff" links are rendered), wrap the "Menu" link with `{role === 'owner' && ...}` so Manager (and other non-owner roles, unless they need it — confirm Waiter/Cook/Customer never needed Menu access, likely true) don't see it at all.
2. **Route guard**: In `app/dashboard/menu/page.tsx`, add a role check at the top — if `role !== 'owner'`, show the same "Access Restricted" pattern used for Billing (Fix B), not the Menu Management UI. This covers direct URL access too, not just hiding the nav link.

## Scope boundary
- Only touch nav visibility + route guard for Menu. Do NOT touch RLS again (already correct from previous Fix C step) and do NOT touch any other page/nav item.
- If the nav component doesn't already have per-role conditional rendering elsewhere, check how Fix B implemented similar hiding (e.g. Billing) and follow the same pattern for consistency — don't invent a new pattern.

## Verification checklist (Ayush, manual browser test — screenshot each)
- [ ] Manager: no "Menu" link in nav at all
- [ ] Manager: direct URL `dashboard/menu` → Access Restricted page (not the broken Menu Management UI)
- [ ] Owner: Menu nav link present, full Menu Management page works as before (toggle, edit, remove, add item)
- [ ] Waiter/Cook/Customer: no regression (they likely never had Menu access — confirm no change)

## Standing rules (unchanged)
- No GitHub push without Ayush's explicit go-ahead after verification.
- Antigravity: code execution + build/compile check only. No browser testing.
