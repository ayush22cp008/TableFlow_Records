# Chat 7: Waiter Dashboard Implementation Plan (v2 — Pre-execution Investigation)

## 1. AuthForm.tsx Routing Fix
**Objective:** Add `waiter` routing branch so waiters are directed to `/dashboard/waiter`.
**File:** `components/AuthForm.tsx`
**Changes:**
- Update lines 137, 181, and 215 where redirect logic is written.
- Current pattern: `... role === 'manager' ? '/dashboard/manager' : '/order'`
- New pattern: `... role === 'manager' ? '/dashboard/manager' : role === 'waiter' ? '/dashboard/waiter' : '/order'`

## 2. Pre-execution Investigation Results

1. **Current state of Waiter Dashboard:** The directory `app/dashboard/waiter` and the file `page.tsx` **DO NOT EXIST**. It hasn't been built yet.
2. **Owner/Cook's priority-first sort logic and labeling:** 
   - Found in `app/dashboard/cook/page.tsx`.
   - **Labeling logic:**
     ```typescript
     const orderNumber = order.daily_number
       ? `${order.is_priority ? 'R' : 'W'}${order.daily_number}`
       : `#${order.id.slice(0, 6)}`
     ```
   - **Priority Sorting:**
     ```typescript
     .order('is_priority', { ascending: false })
     .order('created_at', { ascending: true })
     ```
3. **Existing RLS policies for waiter:** We need to verify if `waiter_ready_to_served` and `waiter_cancel` exist in the live database. (Ayush to check via Supabase SQL Editor).
4. **Tables/Reservations UI reuse feasibility:** Yes, the Owner's existing Tables UI (`app/dashboard/tables/page.tsx` and `app/dashboard/reservations/page.tsx` assuming it exists) can be reused for the Waiter via Navbar links. We will need to check the exact paths.

## 3. Implementation Steps
1. Create `app/dashboard/waiter/page.tsx`.
2. Implement the "Ready" orders queue with:
   - `orderNumber` labeling (W#/R# + Table Number). Example: `W3 · Table 2`
   - Priority sorting logic from the Cook dashboard.
   - "Mark Served" button to transition `Ready` -> `Served`.
3. Add Navbar links for Tables and Reservations if applicable.

## 4. Open Questions for Ayush
> [!IMPORTANT]
> **To Ayush:**
> 1. Please confirm if the `waiter_ready_to_served` and `waiter_cancel` RLS policies exist in the live database.
> 2. Should I go ahead and create `app/dashboard/waiter/page.tsx` and apply the `AuthForm.tsx` routing fix?
> 3. Does `app/dashboard/tables/page.tsx` and `app/dashboard/reservations/page.tsx` exist for reuse, or is everything inside `app/dashboard/tables`? (I can check the directory structure if you want).
