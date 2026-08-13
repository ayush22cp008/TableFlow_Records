# Instruction — Investigate: Force Logout All Staff + Delete Staff Feasibility

**Chat #9 — Investigation only. Do NOT write any fix. Follow-up to `Chat9_Instruction_FixOwnerSideThreeUpdates.md` (still valid, not yet executed).**

## Context — 2 new requirements from Ayush

**A. "End of Day" force logout:** Owner wants a button that, when pressed, force-logs-out ALL currently logged-in staff sessions at once (e.g. pressed when the restaurant closes for the night). This also makes the "Active" status (from the earlier Staff Details work) more accurate — if staff are properly logged out at close, `last_sign_in_at` won't falsely show them as "Active" the next day just because they forgot to log out.

**B. Delete Staff:** Owner wants a Delete button in the Staff Details section to remove a staff member entirely.

## Investigate

### A. Force logout feasibility
1. Does Supabase Admin SDK support invalidating/revoking all active sessions for a given user (or in bulk for multiple users)? Report the exact API (e.g. `supabaseAdmin.auth.admin.signOut(userId)` or similar) and whether it requires the Service Role key (server-side only).
2. Is there a way to fetch "currently logged in" staff specifically (vs all staff ever), or would this need to iterate over all staff profiles and sign each one out regardless of current session state?
3. Report whether this needs a new server-side API route (e.g. `/api/staff/logout-all`) since it requires the Admin SDK — confirm this can't be done from the client directly (security).
4. Report any rate limits or batch constraints if signing out many users in one action.

### B. Delete staff feasibility
1. What happens currently if a `profiles` row is deleted — are there foreign key constraints (e.g. orders, past reservations, staff-created records) that would block deletion or cause orphaned data?
2. Does deleting a `profiles` row also need to delete/deactivate the corresponding `auth.users` entry (via Admin SDK), or just the `profiles` row? Report the correct approach to fully remove a staff account vs just their profile data.
3. Report whether "delete" should be a hard delete or whether a soft-delete (e.g. `is_deleted` flag) is safer given existing foreign key relationships — don't decide, just report the tradeoffs based on actual schema constraints found.
4. Confirm whether the existing `invite_codes` row for that staff member should also be cleaned up/marked on deletion, or left as historical record.

## Output required
Pure findings for both A and B — exact APIs, constraints, and feasibility. No fix, no code changes. Report back before any fix is proposed.
