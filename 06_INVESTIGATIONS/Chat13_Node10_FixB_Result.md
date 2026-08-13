# Node 10 - Fix B Result: Owner Read-Only Operations

## 1. RLS Policy Changes (Manual SQL)

The following SQL must be run manually in Supabase to restrict the Owner role from making direct DB updates to Orders, Tables, and Waitlist. 

*(Note: The Bulk Emergency Stop RPC (`cancel_active_orders`) was verified. It uses `SECURITY DEFINER` and explicitly bypasses RLS, so dropping the table-level orders policy will **not** break the Owner's ability to trigger a bulk emergency stop).*

```sql
-- 1. Orders: Drop owner's all_updates policy completely
DROP POLICY IF EXISTS "owner_all_updates" ON orders;

-- 2. Tables: Replace original policy with one scoped to Waiter and Manager only
DROP POLICY IF EXISTS "tables_write" ON restaurant_tables;
CREATE POLICY "tables_write_staff" ON restaurant_tables FOR ALL
USING (has_role(ARRAY['waiter', 'manager'])) 
WITH CHECK (has_role(ARRAY['waiter', 'manager']));

-- 3. Waitlist: Replace original policy with one scoped to Waiter and Manager only
DROP POLICY IF EXISTS "waitlist_update" ON waitlist;
CREATE POLICY "waitlist_update_staff" ON waitlist FOR UPDATE 
USING (has_role(ARRAY['waiter', 'manager']));
```

## 2. Frontend Changes Confirmed
The following files were updated to apply conditional rendering so that operations are disabled or hidden when the user is an 'owner':
- **`app/dashboard/orders/page.tsx`**: `advanceStatus` (moving orders through stages) and `submitCancel` are hidden from the Owner. The bulk emergency modal is unaffected.
- **`app/dashboard/billing/[orderId]/page.tsx`**: Completely blocked for the Owner with an "Access Restricted" message.
- **`app/dashboard/tables/page.tsx`**: `cycleTableStatus`, `setReservation`, `clearReservation`, `seatWaitlistEntry`, and `cancelWaitlistEntry` are conditionally rendered so that only Managers can use them.

## 3. Build Result
- `npm run build` ran successfully.
- No compile or linting errors.

## 4. Rollback SQL (If needed)

```sql
-- Orders
DROP POLICY IF EXISTS "owner_all_updates" ON orders;
CREATE POLICY "owner_all_updates" ON orders FOR UPDATE
USING ( has_role(ARRAY['owner']) )
WITH CHECK ( has_role(ARRAY['owner']) );

-- Tables
DROP POLICY IF EXISTS "tables_write_staff" ON restaurant_tables;
CREATE POLICY "tables_write" ON restaurant_tables FOR ALL
USING (has_role(ARRAY['waiter', 'manager', 'owner'])) 
WITH CHECK (has_role(ARRAY['waiter', 'manager', 'owner']));

-- Waitlist
DROP POLICY IF EXISTS "waitlist_update_staff" ON waitlist;
CREATE POLICY "waitlist_update" ON waitlist FOR UPDATE 
USING (has_role(ARRAY['waiter', 'manager', 'owner']));
```

**Status:** Fix complete locally. Ready for Ayush to run the manual SQL in Supabase, review, and push to GitHub.
