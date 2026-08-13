# Instruction — Fix: Real-Time Session-Based Staff Status (is_logged_in)

**Chat #9 — Fix only. Investigation already complete (see `Chat9_Investigation_DeleteAndLogoutStatus.md` or equivalent). Do not re-investigate.**

## Confirmed gap
Current "Active"/"Inactive" badge is based on `last_login` within 48 hours — a login-recency signal, not real-time session state. Confirmed via manual test: Ayush logged out of a Cook account from the client (mobile), but the Owner's Staff Details table still showed that Cook as "Active" because `last_login` only updates on sign-IN, never on sign-OUT.

Ayush wants **real-time session-based status**: the moment a staff member logs out (client-side), their status should immediately flip to Inactive on the Owner's Staff Details view.

## Fix — add `is_logged_in` boolean, update on both login and logout

### 1. Migration
- Add `is_logged_in` (boolean, default `false`) to `profiles`.
- Update the existing sign-in trigger (`sync_last_sign_in_at` or equivalent from the earlier migration) to also set `is_logged_in = true` whenever `last_sign_in_at` updates.
- Backfill: for any profile where `last_login` is within the last 48 hours (i.e. likely still an active session), set `is_logged_in = true` as a one-time migration step; otherwise `false`.
- Update `force_logout_all_staff()` RPC: after deleting sessions from `auth.sessions`, also set `is_logged_in = false` for all affected staff profiles (`role IN ('waiter','cook','manager')`).

### 2. Client-side logout hook
Wherever staff sign-out currently happens (the Sign Out button/action used by Waiter/Cook/Manager dashboards), add a call that sets `profiles.is_logged_in = false` for the current user **before or alongside** `supabase.auth.signOut()`. Use a direct `.update()` call on `profiles` (RLS should allow a user to update their own row — confirm this, don't assume).

### 3. Staff Details status logic update
In `app/dashboard/staff/page.tsx`, change the Active/Inactive badge logic from "last_login within 48h" to: **Active if `is_logged_in = true`, else Inactive.** Remove the 48-hour time-window calculation — it's replaced by this boolean.

### 4. Delete/deactivate flow consistency
When Owner deletes/deactivates a staff member (`is_active = false` via `/api/staff/deactivate`), also explicitly set `is_logged_in = false` for that profile in the same operation, so status doesn't show stale "Active" between deactivation and their next login attempt (which will fail anyway due to the ban, but keep data consistent).

## Constraints
- Do not touch Node 3, Node 4, or the Reservation Requests panel fix — all locked/done.
- Confirm RLS policy allows a staff member to update their own `is_logged_in` field — if not, flag it rather than silently failing.
- Migration file only — do not run `db push`. Ayush will run manually via SQL Editor.
- Keep the fix minimal — no unrelated refactors.
- No manual UI testing on your side — Ayush will test manually (including cross-device logout test) and confirm before push.
- Report back: files touched, exact diff/snippet per part (1-4), migration file content, RLS confirmation, and build/compile result.
