# TableFlow — Chat #4 — Instruction (Fix): User Delete Failure

**To:** Antigravity
**From:** Claude
**Type:** Fix — apply schema changes based on locked investigation

---

## Reference

Investigation: `03_Investigation_and_Errors/Chat4_Investigation_UserDeleteFailure.md`

## Root Cause (confirmed via logs)

Two foreign key constraints block user deletion from `auth.users`:

1. `profiles.id` → `auth.users(id)` — live database is missing `ON DELETE CASCADE` (even though `profiles.sql` file states it should have it — file and live DB have drifted).
2. `invite_codes.created_by` → `profiles(id)` — no cascade rule at all (defaults to RESTRICT).

## Decision Locked

- `profiles.id` → `auth.users(id)`: **`ON DELETE CASCADE`** (deleting the auth user deletes their profile — standard practice, profile has no meaning without the auth account).
- `invite_codes.created_by` → `profiles(id)`: **`ON DELETE SET NULL`** (deleting a staff member who created invite codes should NOT delete the invite codes themselves — preserve invite history, just clear the creator reference).

## Build Tasks

1. Write a new migration file (follow existing naming convention, e.g. `supabase/migrations/{timestamp}_fix_user_delete_cascade.sql`) that:
   - Drops and recreates `profiles_id_fkey` on `profiles.id` referencing `auth.users(id)` with `ON DELETE CASCADE`
   - Drops and recreates the FK on `invite_codes.created_by` referencing `profiles(id)` with `ON DELETE SET NULL`
   - Confirm `invite_codes.created_by` column is nullable (required for `SET NULL` to work — if it's currently `NOT NULL`, that constraint must be relaxed first)

2. **Apply this migration directly to the live Supabase database** — per the Node 2b lesson, do not just commit the file; confirm it's actually applied live, the same way the earlier migration drift was caught and fixed.

3. Do NOT touch `waitlist.customer_id`, `orders.customer_id`, or `feedback.customer_id` — these already correctly use `ON DELETE SET NULL` per the investigation, no changes needed.

## Verification

- After migration applied, attempt to delete `ayushhalpati09@gmail.com` (or the same test user) again from Supabase Dashboard > Authentication > Users
- Confirm deletion succeeds without error
- Confirm any invite codes previously created by that user still exist in `invite_codes` table, with `created_by` now `NULL`
- Run `npm run build` to confirm no application code broke (this is a DB-only change, but confirm nothing references `created_by` expecting it to always be non-null)

## Output

Report back: migration file name/path, confirmation it was applied live, and result of the delete-retry test. Do not push to GitHub — Ayush triggers that manually.
