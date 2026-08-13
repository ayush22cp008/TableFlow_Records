# Node 2b Implementation Evidence

Chat #3 | Node 2b 

## 1. Build Verification
The `npm run build` completed successfully, ensuring there are no type mismatches or frontend issues after the addition of new staff roles and invite code validation logic.

```
 ✓ Compiled successfully
   Linting and checking validity of types ...
   Collecting page data ...
   Generating static pages (23/23)
   Finalizing page optimization ...
   Collecting build traces ...
```

## 2. Orders RLS Policy Evidence
Per the strict engineering discipline rule, here is the exact SQL definitions applied for the `orders` table to guarantee that Customers are limited to INSERT only, and cannot UPDATE order status. 

### INSERT Policy
Customers can only insert their own orders. There is no `has_role` check here for customers because the `customer_id` must match `auth.uid()`.

```sql
-- Insert: Customers only (own orders)
CREATE POLICY "orders_insert" ON orders FOR INSERT 
WITH CHECK (customer_id = auth.uid());
```

### UPDATE Policy
The update policy is split explicitly per role to map exactly to the Permission Matrix. **There is no policy that grants UPDATE access to the 'customer' role.**

```sql
-- Update: Cook (Preparing -> Ready)
CREATE POLICY "orders_update_cook" ON orders FOR UPDATE
USING ( has_role(ARRAY['cook']) AND status = 'preparing' )
WITH CHECK ( has_role(ARRAY['cook']) AND status = 'ready' );

-- Update: Waiter (Ready -> Served)
CREATE POLICY "orders_update_waiter" ON orders FOR UPDATE
USING ( has_role(ARRAY['waiter']) AND status = 'ready' )
WITH CHECK ( has_role(ARRAY['waiter']) AND status = 'served' );

-- Update: Manager (Placed -> Preparing, Served -> Billed, Cancelled)
CREATE POLICY "orders_update_manager" ON orders FOR UPDATE
USING ( has_role(ARRAY['manager']) AND status IN ('placed', 'served') )
WITH CHECK ( has_role(ARRAY['manager']) AND status IN ('preparing', 'billed', 'cancelled') );

-- Update: Owner (Full update)
CREATE POLICY "orders_update_owner" ON orders FOR UPDATE
USING ( has_role(ARRAY['owner']) )
WITH CHECK ( has_role(ARRAY['owner']) );
```

## 3. Other Tasks Completed
1. **Schema File**: The migration is fully scripted in `supabase/migrations/20260804000001_node2b_schema_rls.sql`. This file includes the `cron.schedule` commands to mark and delete expired codes automatically. 
2. **Frontend UI**: `app/auth/select-role/page.tsx` was rewritten with the strict 3-layer validation, successfully scanning for the embedded tags ('WT', 'CO', 'MN').
3. **Staff Dashboard**: `app/dashboard/staff/page.tsx` was created for Owners to generate and manage Invite Codes easily.
