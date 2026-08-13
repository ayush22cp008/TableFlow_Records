# Instruction — Fix A: Reservation RLS Security Gap

**Scope:** ONLY the `reservation_requests` RLS policy. Do not touch orders/billing/tables/menu — those are separate fixes (B and C), coming later.

## Problem (from investigation)
`reservation_requests` table currently has:
```sql
"Allow public update" ON reservation_requests FOR UPDATE TO public USING (true)
```
This allows ANY request (even unauthenticated) to update reservation requests. This is a genuine security gap, not just an Owner/Manager overlap issue.

## Required change
Replace the open public policy with one scoped to **Manager only** for `approve`/`reject` actions (Owner becomes read-only — no update policy for Owner on this table going forward; Owner's Node 9 dependency is not affected by this).

## Steps
1. Run in Supabase SQL Editor (manual — CLI unavailable on Windows):
   - Drop the existing `"Allow public update"` policy on `reservation_requests`.
   - Create a new UPDATE policy scoped to `manager` role only (match the role-check pattern already used in other locked policies in this project, e.g. `owner_all_updates` on `orders`, for consistency).
2. Log this manual SQL change immediately in `04_Logs/` per project rule (investigation+fix must be captured, not deferred).
3. Frontend (`app/dashboard/tables/page.tsx`): confirm `approveRequest`/`rejectRequest` calls only fire for `userRole === 'manager'` — hide/disable these actions in the UI for Owner (Owner sees requests, read-only).
4. Build and confirm no compile errors. Do not run local dev server test — Ayush will test via Vercel deploy after push.

## Rollback (if this breaks reservation approve/reject after deploy)
```sql
-- Revert: drop the new scoped policy and restore the original open policy
DROP POLICY IF EXISTS "<new_policy_name_used_in_step_1>" ON reservation_requests;

CREATE POLICY "Allow public update" ON reservation_requests
FOR UPDATE TO public USING (true);
```
Antigravity: fill in the exact policy name you create in Step 1 into this rollback block before handing back, so it's ready to paste if needed.

## After Antigravity finishes
Report back (in `03_Investigation_and_Errors/Chat13_Node10_FixA_Result.md` or similar):
- Exact new policy name and SQL used
- Confirmation frontend change is scoped correctly
- Build result
- Confirm rollback SQL above has the correct policy name filled in

Do NOT push to GitHub yet — wait for Ayush's explicit go-ahead after he reviews.
