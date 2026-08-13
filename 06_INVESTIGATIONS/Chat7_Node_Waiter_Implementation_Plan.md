# Chat 7: Waiter Dashboard Implementation Plan

## 1. AuthForm.tsx Routing Fix
**Objective:** Add `waiter` routing branch so waiters are directed to `/dashboard/waiter`.
**File:** `components/AuthForm.tsx`
**Changes:**
- Update lines 137, 181, and 215 where redirect logic is written.
- Current pattern: `... role === 'manager' ? '/dashboard/manager' : '/order'`
- New pattern: `... role === 'manager' ? '/dashboard/manager' : role === 'waiter' ? '/dashboard/waiter' : '/order'`

## 2. Waiter Dashboard Implementation
**Objective:** Implement Waiter Dashboard (`app/dashboard/waiter/page.tsx` and related components) based on `Chat6_Node_Waiter_Decision_OrderLabeling_v2.md`.
**Requirements:**
- **Order Labeling:** Show both the daily-reset sequence label (`W#` or `R#`) and the real table number together (e.g. `W3 · Table 2`). This must reuse the existing `daily_order_counters` system.
- **Priority Sorting:** Sort `R` (reservation/priority) orders above `W` (walk-in) orders in the Waiter's queue (e.g., the "Ready" column). This sorting must happen regardless of arrival time, exactly matching the Owner/Cook implementations. Do NOT copy the Manager Dashboard sorting (which currently has bugs).

## 3. Pre-execution Investigation
Before modifying the Waiter Dashboard, we need to:
1. Check the current state of `app/dashboard/waiter/page.tsx`.
2. Inspect the Owner/Cook views (e.g., `app/dashboard/cook/page.tsx` or similar components) to extract the correct priority-first sort logic and order sequence numbering.

**Status:** Awaiting approval from Ayush to begin pre-execution investigation and apply the `AuthForm.tsx` fix.
