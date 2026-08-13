# Investigation: Migration Scope Check

## 1. Context
We previously discovered that the `invite_codes` table is missing on the live Supabase instance because the migration `20260804000001_node2b_schema_rls.sql` was never executed. We suspected that this missing migration—and potentially its sibling `20260804000002_node2b_cancellation.sql`—also contained the entire RLS overhaul and cancellation system.

## 2. Findings (Direct Database Query)
I queried the live PostgreSQL database directly to verify the existence of all objects created by these two migrations.

### RPCs (Functions)
- `cancel_active_orders`: **MISSING**
- `mark_expired_invite_codes`: **MISSING**
- `delete_expired_invite_codes`: **MISSING**
- `has_role`: **MISSING**

### Columns
- `orders.cancellation_reason`: **MISSING**
- `orders.cancellation_category`: **MISSING**

### RLS Policies
The advanced paired-transition RLS policies are **MISSING**. The database is currently running the old, simplistic placeholder policies from earlier nodes:
- `orders`: `orders_own_read`, `orders_own_insert`, `orders_owner_update`
- `restaurant_tables`: `tables_public_read`, `tables_owner_write`
- `menu_items`: `menu_public_read`, `menu_owner_write`

## 3. Conclusion
Neither of the Node 2b migrations were ever applied to the live database. The entire Node 2b Part 1 scope—which includes the Staff Roles infrastructure, the comprehensive RLS overhaul for orders/tables/menu, and the emergency cancellation system—is currently absent in production.

**Status**: Awaiting approval to execute these missing migrations against the live database.
