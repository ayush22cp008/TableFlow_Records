# Instruction — Investigate: last_sign_in Availability for Staff Status (Follow-up)

**Chat #9 — Investigation only. Do NOT write any fix.**

## Context
Following up on `Chat9_Investigation_NavbarAndStaffSchema.md`. Ayush confirmed:
- **Name**: already captured correctly in `invite_codes.staff_name` when the owner generates an invite code (visible in the Invite Codes History table — e.g. "op", "patel", "divyesh"). No new column needed — just needs to be joined/matched with `profiles` by email (where `invite_codes.status = 'used'`) when building the Staff Details view.
- **Status (active/inactive)**: Ayush's proposed approach — determine a staff member's active/inactive status based on whether they've logged in recently (e.g. within the last 24-48 hours), rather than a manually-set flag.

## Investigate

1. Does Supabase Auth's `auth.users` table already track `last_sign_in_at` for each user by default? Confirm this is available and query-able (Supabase Auth typically tracks this natively — verify for this project specifically).
2. Is `auth.users` currently joined/queried anywhere in the existing codebase (e.g. via `supabase.auth.admin.listUsers()` or a similar admin API call), or would this be a new integration point?
3. If `last_sign_in_at` is available: report what data format it's in, and confirm it can be fetched server-side for the Owner's Staff Management page (may require Supabase service role / admin API — flag if so, since this has security implications and should not be exposed client-side).
4. Report a simple proposed rule using this data: e.g. "Active" if `last_sign_in_at` within last 48 hours, else "Inactive" — confirm feasibility, don't implement.
5. If `last_sign_in_at` is NOT easily accessible (e.g. requires admin-only API not currently set up), report that clearly as a blocker and suggest the simplest alternative (e.g. a lightweight `last_login` timestamp column on `profiles`, updated via a trigger or client-side update on login).

## Output required
Pure findings — confirm whether login-based status is feasible with current setup, and what (if anything) is missing. No fix, no code changes. Report back before any fix is proposed.
