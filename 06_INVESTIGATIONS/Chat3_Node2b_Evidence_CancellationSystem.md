# Node 2b Cancellation System Evidence

Chat #3 | Node 2b

## 1. Build Verification
The `npm run build` completed successfully, ensuring the new UI components and schema field additions are type-safe.

```
 ✓ Compiled successfully
   Linting and checking validity of types ...
   Collecting page data ...
   Generating static pages (23/23)
   Finalizing page optimization ...
   Collecting build traces ...
```

## 2. Final Exact RLS Policies for `orders`
As required by the evidence rule, here are the **exact paired policies** currently applied to the `orders` table. The old `orders_update_*` policies were dropped in favor of these explicitly mapped pairs.

### Cook
```sql
CREATE POLICY "cook_prep_to_ready" ON orders FOR UPDATE
USING ( has_role(ARRAY['cook']) AND status = 'preparing' )
WITH CHECK ( has_role(ARRAY['cook']) AND status = 'ready' );

CREATE POLICY "cook_cancel" ON orders FOR UPDATE
USING ( has_role(ARRAY['cook']) AND status IN ('placed','preparing','ready') )
WITH CHECK ( has_role(ARRAY['cook']) AND status = 'cancelled' AND cancellation_reason IS NOT NULL );
```

### Waiter
```sql
CREATE POLICY "waiter_ready_to_served" ON orders FOR UPDATE
USING ( has_role(ARRAY['waiter']) AND status = 'ready' )
WITH CHECK ( has_role(ARRAY['waiter']) AND status = 'served' );

CREATE POLICY "waiter_cancel" ON orders FOR UPDATE
USING ( has_role(ARRAY['waiter']) AND status IN ('placed','preparing','ready') )
WITH CHECK ( has_role(ARRAY['waiter']) AND status = 'cancelled' AND cancellation_reason IS NOT NULL );
```

### Manager
```sql
CREATE POLICY "manager_placed_to_prep" ON orders FOR UPDATE
USING ( has_role(ARRAY['manager']) AND status = 'placed' )
WITH CHECK ( has_role(ARRAY['manager']) AND status = 'preparing' );

CREATE POLICY "manager_served_to_billed" ON orders FOR UPDATE
USING ( has_role(ARRAY['manager']) AND status = 'served' )
WITH CHECK ( has_role(ARRAY['manager']) AND status = 'billed' );

CREATE POLICY "manager_cancel" ON orders FOR UPDATE
USING ( has_role(ARRAY['manager']) AND status IN ('placed','preparing','ready') )
WITH CHECK ( has_role(ARRAY['manager']) AND status = 'cancelled' AND cancellation_reason IS NOT NULL );
```

### Owner
```sql
CREATE POLICY "owner_all_updates" ON orders FOR UPDATE
USING ( has_role(ARRAY['owner']) )
WITH CHECK ( has_role(ARRAY['owner']) );
```

## 3. Other Tasks Completed
1. **Schema File**: The migration is fully scripted in `supabase/migrations/20260804000002_node2b_cancellation.sql`.
2. **RPC Function**: `cancel_active_orders(p_reason, p_category, p_order_ids)` was created to securely perform bulk atomic updates (and table clearing) directly in the database.
3. **Frontend UI**: `app/dashboard/orders/page.tsx` was rewritten to include the mandatory reason Single-Cancel modal for all staff, and the Bulk Emergency Stop modal for owners.
