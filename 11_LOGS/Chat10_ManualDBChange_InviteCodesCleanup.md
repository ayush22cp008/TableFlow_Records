# Manual DB Change Log: One-Time Invite Codes Cleanup

**Type:** Manual DB Change Record (Chat 10)
**Executed by:** Ayush, via Supabase SQL Editor (Production DB)
**Date:** 2026-08-10

## Change
```sql
DELETE FROM public.invite_codes WHERE status = 'used';
```

## Reason
`invite_codes` history had accumulated 40+ old entries from VibeAthon-era testing and later staff onboarding, all with `status = 'used'`. Used codes are single-use and have no further function once consumed — kept only cluttering the Staff Management "Invite Codes History" view. Decision: one-time wipe of used codes, leave unused/pending codes untouched.

## Result
Query executed successfully. `invite_codes` table came back completely empty — confirming all existing rows at the time were `status = 'used'` (no unused/pending codes existed to preserve). Verified via Supabase Table Editor and the Staff Management UI ("No invite codes generated yet.").

## Scope / Safety
Per investigation (`Chat10_Investigation_InviteCodesCleanup_Result.md`): no foreign keys reference `invite_codes`, so this delete had zero effect on `profiles`, `auth.users`, or any other table. Purely a history/audit-log cleanup.

## Follow-up (not yet implemented)
- Postgres trigger for auto-deleting batches of 10 "used" codes going forward.
- Manual per-row delete button on the Invite Codes History UI (owner-only, via existing RLS policy — no new API route needed).
