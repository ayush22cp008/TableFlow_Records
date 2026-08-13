# Investigation: Invite Codes Cleanup (One-Time + Auto-Batch + Manual Delete)

**Type:** Investigation only — no code changes.

## Context
"Invite Codes History" on the Staff Management page has accumulated many old entries (used and pending) going back to VibeAthon testing. We want three things, but need investigation before planning any fix:

1. **One-time cleanup:** Delete all existing invite codes with `status = 'used'`. Leave pending/unused codes untouched.
2. **Ongoing automatic rule:** Once "used" codes accumulate to a batch of 10, auto-delete that batch of 10.
3. **Manual delete:** Owner can manually delete any individual invite code row (used or pending) directly from the Invite Codes History table.

## Investigate

1. **Schema check — `invite_codes` table:**
   - Confirm exact column names (status values used, e.g. `'used'` vs `'pending'` vs `'expired'`).
   - Check for any foreign key relationships pointing to or from this table (e.g. does anything else reference `invite_codes.id`?).
   - Confirm deleting a row here has zero effect on `profiles` or `auth.users` (i.e. deleting the invite code history is safe and independent of the actual staff account).

2. **One-time cleanup query:**
   - Identify the safe SQL to delete all `status = 'used'` rows via Supabase SQL Editor (manual run, per standing constraint — CLI unavailable).

3. **Batch-of-10 auto-delete — feasible approaches:**
   - Option A: Trigger-based (Postgres trigger fires after insert/update, counts used rows, deletes oldest 10 when threshold hit).
   - Option B: App-level check (every time an invite code's status flips to "used", an API route checks count and deletes oldest 10 if ≥10).
   - Note pros/cons of each — reliability, complexity, whether it depends on the app being running (App-level only fires if that code path executes) vs DB-level (fires regardless of app).

4. **Manual delete UI/route:**
   - Confirm whether an existing API route handles invite code deletion, or if a new one is needed (e.g. `app/api/staff/invite-codes/[id]/route.ts` DELETE handler).
   - Check what RLS policies exist on `invite_codes` for delete operations — confirm owner role has delete permission, or if a policy needs adding.

## Output
Report findings only — schema details, safest approach for the batch auto-delete (recommend one, with reasoning), any RLS/permission gaps for manual delete, and the one-time cleanup SQL. No implementation yet.
