# Chat 8: Manager Dashboard Priority Sort Bug (Investigation)

## 1. Intake Queue (Placed Orders)
- **File Path:** `app/dashboard/manager/page.tsx`
- **Lines:** 22-27
- **Current Sort Logic:**
  ```typescript
  const { data: placedData } = await supabase
    .from('orders')
    .select('*, order_items(quantity, unit_price, menu_items(name)), restaurant_tables(table_number)')
    .eq('status', 'placed')
    .order('created_at', { ascending: true }) // Missing is_priority
  ```
- **Finding:** The `is_priority` sort clause is **completely missing**. It is only sorting by `created_at`.

## 2. Billing Queue (Served Orders)
- **File Path:** `app/dashboard/manager/page.tsx`
- **Lines:** 28-33
- **Current Sort Logic:**
  ```typescript
  const { data: servedData } = await supabase
    .from('orders')
    .select('*, order_items(quantity, unit_price, menu_items(name)), restaurant_tables(table_number)')
    .eq('status', 'served')
    .order('created_at', { ascending: true }) // Missing is_priority
  ```
- **Finding:** Similar to the Intake Queue, the `is_priority` sort clause is **completely missing**.

## 3. Conclusion & Comparison to Owner/Cook Pattern
- **Root Cause:** Both Bug 1 (Intake Queue) and Bug 2 (Billing Queue) share the exact same root cause: the absence of the `.order('is_priority', { ascending: false })` clause before the `created_at` sort.
- **Comparison:** The Owner/Cook queues explicitly include the `is_priority` sort, which guarantees Reservations/Priority ('R') orders appear above Walk-in ('W') orders. The Manager Dashboard was built without this clause, hence the default chronological sort.

**Status:** Investigation complete. Awaiting fix instructions. No code has been modified.
