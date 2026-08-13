Chat #3 | Node 2b | Instruction — Investigate Missing invite_codes Table Error (INVESTIGATION ONLY, no fix)

Manual testing on Staff Management page (`/dashboard/staff`) found two errors:
1. "Could not find the table 'public.invite_codes' in the schema cache" — when clicking Generate Code
2. "Failed to load invite codes." — when the Invite Codes History section loads

This is unexpected since invite_codes was reported as built in Node 2b Part 1 (locked/built per handoff notes — profiles.role extended, invite_codes table with status enum, pg_cron expiry).

## Investigate
1. Query Supabase directly (via SQL editor or MCP/CLI access if available) to confirm: does `public.invite_codes` actually exist as a table right now? Report exact result.
2. If it exists: check if this is a Supabase PostgREST schema cache staleness issue (common after table creation/migration — cache needs a reload). Check Supabase dashboard for a "Reload Schema" option (Database → API settings, or similar) and report whether this is likely the cause.
3. If it does NOT exist: trace back — was the migration for invite_codes ever actually applied to this Supabase project, or only written/planned? Check migration files in the repo (supabase/migrations or wherever they live) vs. what's actually live in the DB.
4. Check RLS policies on invite_codes (if it exists) — could a misconfigured policy cause a "table not found"-style error instead of a permission error, for the Owner role querying it?
5. Report exact current schema state: table exists/doesn't, columns if it exists, RLS policies if any.

## Output
Report to: `03_Investigation_and_Errors/Chat3_Node2b_Investigation_InviteCodesTableMissing.md`

Include exact findings — do not fix yet. If it's a simple schema-cache-reload situation, note that as the likely fix, but still report first per evidence rule before applying anything.
