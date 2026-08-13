Chat #3 | Node 2b | Instruction — Cleanup Temp Scripts + Apply Missing Migrations

Per investigation reports (InviteCodesTableMissing + MigrationScopeCheck): both `20260804000001_node2b_schema_rls.sql` and `20260804000002_node2b_cancellation.sql` were committed locally but never applied to the live Supabase database. Full Node 2b Part 1 scope (invite_codes, cancellation system, RLS overhaul) is missing in production.

## Step 1: Cleanup temporary investigation scripts
Delete these two files — they were one-off DB inspection scripts, not part of the application:
- `check_db.js`
- `check_scope.js`

Confirm they're deleted from the working directory and were never committed (check `git status` / `git log` to be sure neither was accidentally pushed in an earlier commit — if either was committed, note that so we can decide whether to clean git history or just remove going forward).

## Step 2: Apply both missing migrations to the live Supabase database
Run both migrations against the live database, in order:
1. `20260804000001_node2b_schema_rls.sql`
2. `20260804000002_node2b_cancellation.sql`

Use whatever mechanism is standard for this project (Supabase CLI `db push`, or direct SQL execution via the same `pg` client connection used for investigation — whichever is safer and already proven to work).

## Step 3: Verify (re-run the same checks from the investigation)
After applying, confirm via direct DB query that ALL of the following now exist:
- `invite_codes` table
- `mark_expired_invite_codes`, `delete_expired_invite_codes`, `cancel_active_orders`, `has_role` RPCs
- `orders.cancellation_reason`, `orders.cancellation_category` columns
- New RLS policies replacing the old Chat 1 placeholder policies on `orders`, `restaurant_tables`, `menu_items`

## Evidence required
Report to: `03_Investigation_and_Errors/Chat3_Node2b_Evidence_MigrationsApplied.md`

Include:
- Confirmation both temp scripts are deleted and git-clean
- Migration execution result (success/failure, any errors)
- Full verification query results confirming every object above now exists
- Confirm this did NOT touch any existing data (this should be additive schema changes only — no data loss)

No git commit/push yet — this is a database-only change. We'll re-test the Staff Management page live first (schema changes take effect immediately, no redeploy needed), then decide if any code-side commit is needed separately.
