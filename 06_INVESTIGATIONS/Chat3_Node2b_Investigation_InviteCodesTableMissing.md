# Investigation: Missing `invite_codes` Table Error

## 1. Direct Database Query Results
**Finding:** The table `public.invite_codes` **does NOT exist** in the live database.
I bypassed the REST API and connected directly to the PostgreSQL database via a Node.js `pg` client using the database password. Querying `information_schema.tables` for the `public` schema returned the following existing tables:
`daily_order_counters`, `feedback`, `profiles`, `menu_items`, `restaurant_tables`, `waitlist`, `orders`, `order_items`, `reservation_requests`.
`invite_codes` is entirely missing.

## 2. Schema Cache Check
**Finding:** This is **not** a PostgREST schema cache staleness issue. The REST API returned the "not in schema cache" error simply because the underlying PostgreSQL table does not exist.

## 3. Migration Trace-back
**Finding:** The migration file was written but never pushed to the live database.
Inside `supabase/migrations/20260804000001_node2b_schema_rls.sql`, the SQL to create the `invite_codes` table, define its RLS policies, and set up the `pg_cron` expiry mechanisms is fully documented. 
However, this migration was never executed against the live Supabase project.

## 4. RLS Policies
**Finding:** Not applicable. Since the table itself doesn't exist, RLS is not the cause of the error.

## 5. Current Schema State
- **Table:** `invite_codes` (DOES NOT EXIST)
- **Columns:** None
- **RLS Policies:** None applied.

**Conclusion:** The root cause is that the `20260804000001_node2b_schema_rls.sql` migration file was committed locally but never applied to the remote database instance. Once that migration is run, the Staff Management page errors will disappear.
