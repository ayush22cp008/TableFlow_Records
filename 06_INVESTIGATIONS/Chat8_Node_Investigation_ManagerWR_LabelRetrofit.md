# Chat 8: Manager Dashboard `W#`/`R#` Label Retrofit (Investigation)

## 1. Where is the label generated?
- **Source Files:** The logic is currently repeated inline in multiple dashboard files, including:
  - `app/dashboard/cook/page.tsx` (lines 66-68)
  - `app/dashboard/waiter/page.tsx` (lines 71-73)
  - `app/dashboard/orders/page.tsx` (lines 172-174)
- **The Logic:**
  ```typescript
  const orderNumber = order.daily_number
    ? `${order.is_priority ? 'R' : 'W'}${order.daily_number}`
    : `#${order.id.slice(0, 6)}`
  ```

## 2. Is it computed or stored?
- **Stored:** The raw components are stored in the DB: `daily_number` (integer) and `is_priority` (boolean) on the `orders` table.
- **Computed:** The final display string (`R3`, `W12`) is computed client-side via the ternary logic shown above.

## 3. Daily Reset Behavior
- **Is it intentional?** Yes, it is fully intentional and designed at the database schema level.
- **How it works:** The numbering relies on a dedicated counter table (`daily_order_counters`) which uses the current date (`counter_date`) as its primary key.
- **Location:** `supabase/migrations/20260804000000_track_b_priority_numbering.sql` (lines 33-42) inside the `place_order_and_occupy_table` RPC.
- **Reset Trigger:** When an order is placed on a new calendar day, the `current_date` doesn't match any existing row, so a new row is inserted. This naturally resets the `priority_count` and `walkin_count` to 1.

## 4. Why Manager Dashboard lacks it
- The Manager Dashboard (`app/dashboard/manager/page.tsx`) hardcodes the display of the raw UUID:
  - **Intake Queue:** `Order #{order.id.slice(0, 6)}` (line 150)
  - **Billing Queue:** `Order #{order.id.slice(0, 8)}` (line 198)
- Because the `W#`/`R#` label generation was never extracted into a shared utility function, it wasn't automatically inherited by the Manager Dashboard. The Manager's query fetches all the necessary data (including `is_priority`), but its render logic ignores it.

## 5. Recommended Reuse Approach
**Recommendation:** Extract the label generation logic into a shared utility function (e.g., in `lib/utils.ts` or similar):
```typescript
export function formatOrderNumber(order: { id: string; daily_number?: number | null; is_priority?: boolean }): string {
  return order.daily_number 
    ? `${order.is_priority ? 'R' : 'W'}${order.daily_number}` 
    : `#${order.id.slice(0, 6)}`;
}
```
Then, update `manager/page.tsx` (and optionally Cook/Waiter/Orders) to import and call this shared helper. This guarantees visual consistency across all staff dashboards and prevents future drift.

**Status:** Investigation complete. Awaiting fix instruction from Claude/Ayush. No code has been modified.
