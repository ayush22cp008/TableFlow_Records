# Investigation Result: Invite Codes Cleanup

**Type:** Investigation Result (Chat 10)

## 1. Schema Check
- **Table Structure:** The `invite_codes` table tracks status using exactly three values: `'unused'`, `'used'`, and `'expired'`.
- **Dependencies:** 
  - There are **no foreign keys** pointing *to* `invite_codes` from any other table.
  - The table has a `created_by` column that references `profiles(id)`, but deleting an invite code does not affect the creator's profile.
- **Safety:** Deleting rows from `invite_codes` is 100% safe. It has zero impact on `profiles`, `auth.users`, or any core system functionality. It merely clears history.

## 2. One-Time Cleanup Query
Ayush can safely run this exact query in the Supabase SQL Editor to wipe all currently used codes:
```sql
DELETE FROM public.invite_codes WHERE status = 'used';
```

## 3. Auto-Batch of 10 Auto-Delete: Recommended Approach
**Recommendation: Option A (Trigger-based in Postgres).**

*Reasoning:* 
- **Separation of Concerns:** If we put this logic in the app (Option B, specifically inside `app/api/auth/staff-signup/route.ts`), it adds a count/delete query to the critical signup path, slightly increasing latency and risk if the cleanup fails. 
- **Reliability:** A Postgres trigger that fires `AFTER UPDATE ON invite_codes` (when `status` changes to `'used'`) is atomic, invisible to the frontend, and bulletproof. It can cleanly check `IF (SELECT count(*) FROM invite_codes WHERE status = 'used') >= 10 THEN DELETE ... END IF;` without the Next.js API route ever needing to know.

## 4. Manual Delete UI/Route
- **Current State:** There is currently no API route for deleting invite codes (`app/api/staff` has no DELETE handlers). Invite codes are created directly from the frontend via `supabase.from('invite_codes').insert()`.
- **RLS Permissions:** The existing RLS policy on `invite_codes` is: 
  `CREATE POLICY "invite_codes_owner_all" ON invite_codes FOR ALL USING (has_role(ARRAY['owner'])) ...`
  This explicitly grants the **owner** permission to perform `DELETE` operations natively.
- **Implementation:** No new API route is required! The manual delete button on the frontend can just call `supabase.from('invite_codes').delete().eq('id', codeId)` directly, and RLS will securely allow it.
