# Manager Dashboard Implementation Plan (Chat 5, Node 3)

This document outlines the implementation plan for the Manager Dashboard (Node 3) per the provided instruction.

## 1. RLS Policy Conflict (FLAGGED)
The instruction states that marking an order as paid should transition it from `served` to `completed`. However, during investigation, I found the existing RLS policy in `20260804000002_node2b_cancellation.sql`:
```sql
CREATE POLICY "manager_served_to_billed" ON orders FOR UPDATE
USING ( has_role(ARRAY['manager']) AND status = 'served' )
WITH CHECK ( has_role(ARRAY['manager']) AND status = 'billed' );
```
This policy requires the status to transition to `billed`, not `completed`. 
**Flag:** Per the "investigation/fix separation rule", I am flagging this issue and will not write the new RLS policy myself. Please confirm if we should update the UI to transition to `billed` instead, or if Claude will provide a new RLS migration for the `completed` state.

## 2. PDF Generation Strategy
There is no PDF library (like `jspdf` or `@react-pdf`) currently in `package.json`. To adhere to the instruction to use a lightweight option and avoid heavy dependencies, I propose using the browser's native `window.print()` combined with `@media print` CSS. This allows the user to use the standard "Save as PDF" browser dialog natively.

## 3. Reusability Investigation (Task 5)
I have investigated the existing Tables, Reservations, and Menu management UI:
- **UI Exists:** `app/dashboard/tables` and `app/dashboard/menu` already exist.
- **RLS Access:** The manager **already has R/W access** at the database level! Policies like `tables_write`, `menu_write`, and `waitlist_update` explicitly include the `manager` role.
- **Nav Blocked:** The only barrier is UI visibility. `components/Navbar.tsx` currently strictly checks `role === 'owner'` for displaying these links.
**Plan:** I will NOT duplicate these pages. Instead, I will update `components/Navbar.tsx` to expose the Menu and Tables links to the `manager` role as well.

---

## Proposed Changes

### Database Schema
**[NEW]** `supabase/migrations/20260804000003_node3_manager_schema.sql`
- Add the `payment_method` column to the `orders` table.
- Enum check constraint: `'cash' | 'card' | 'upi'`.
- Nullable by default.

### Routing & Navigation
**[MODIFY]** `middleware.ts`
- Add branch for `role === 'manager'` to redirect them to `/dashboard/manager` upon login/signup or when trying to access root.

**[MODIFY]** `app/auth/callback/route.ts`
- Add branch for `role === 'manager'` in the magic link auth callback to redirect to `/dashboard/manager`.

**[MODIFY]** `components/Navbar.tsx`
- Expose `/dashboard/menu` and `/dashboard/tables` links to the `manager` role.

### Manager UI
**[NEW]** `app/dashboard/manager/page.tsx`
- Implement the two-column/tab layout based on the `app/dashboard/orders/page.tsx` Kanban pattern.
- **Intake Queue:** Query `status = 'placed'`. "Accept" button transitions to `preparing`.
- **Billing Queue:** Query `status = 'served'`. Dropdown for Payment Method + "Mark Paid" button (action depends on RLS resolution).
- **Print Receipt:** "Download PDF" button that triggers `window.print()`.

---
**Status:** Waiting for Ayush's decision on the RLS Policy conflict before executing the UI and Schema changes.
