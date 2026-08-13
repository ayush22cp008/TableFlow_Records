Chat #3 | Node 2b | Instruction — Apply Missing Migration + Verify Dependent Features

Per investigation (Chat3_Node2b_Investigation_InviteCodesTableMissing.md): `supabase/migrations/20260804000001_node2b_schema_rls.sql` was committed locally but never applied to the live Supabase database. `invite_codes` table is entirely missing as a result.

## Concern before fixing
This same migration file was described as containing the full Node 2b Part 1 scope: profiles.role extension, invite_codes table + RLS + pg_cron expiry, AND the full RLS overhaul (orders/tables/menu paired-transition policies) + order cancellation system (cancellation_reason/category columns, cancel_active_orders RPC). If this migration never ran, those features may also be missing or non-functional on the live DB, despite being reported as "build verified, pushed to GitHub" earlier.

## Step 1: Investigate scope first (still investigation, no fix yet)
Before applying anything, read the full contents of `20260804000001_node2b_schema_rls.sql` and report:
- Every table/column/policy/function/RPC it creates or alters
- For each, confirm via direct DB query (same `pg` client method used in the last investigation) whether it currently exists live or not
- Specifically check: does `cancel_active_orders` RPC exist? Do the `orders`/`tables`/`menu` RLS policies from the Permission Matrix exist? Does `orders.cancellation_reason`/`cancellation_category` exist?

Report this first to: `03_Investigation_and_Errors/Chat3_Node2b_Investigation_MigrationScopeCheck.md`

## Step 2: Apply migration (only after Step 1 report is reviewed and approved)
Do NOT run the migration yet in this same pass. Wait for confirmation after Step 1's findings are reported, since if other already-tested features turn out to also depend on this migration, we need to know the full blast radius before applying it live.

## Why this matters
Per evidence rule and engineering discipline (investigation and fix always separate), and because earlier work was marked "verified" that may not actually be live — this needs to be fully understood before any schema change is applied to production.
