# Fix Plan: Re-Invite Signup Ignores Password (Bug 2)

**Type:** Fix (implementation) — investigation already confirmed root cause in `Chat10_Investigation_DeactivateReinviteBugs_Result.md`.

## Problem
In `app/api/auth/staff-signup/route.ts`, the `existingProfile` re-activation block updates `profiles` (role, is_active, is_logged_in) and syncs `user_metadata.role`, but never sets the password on the Auth user. For accounts originally created via Google OAuth (no password identity), this leaves them without a password — so the immediate `signInWithPassword` call right after signup fails with "Invalid login credentials."

## Fix
### [MODIFY] `app/api/auth/staff-signup/route.ts` — `existingProfile` block
- Include the `password` field (already received in the signup request body) in the `supabaseAdmin.auth.admin.updateUserById()` call for the existing user, alongside the existing `user_metadata` update.
- This sets a password identity on the account without removing the existing Google OAuth identity — user will be able to sign in with either method going forward.

## Verification Plan
### Automated
- `npm run build` to confirm no type/syntax errors.

### Manual (Ayush)
- Deactivate a staff member whose original account was Google OAuth-based.
- Re-invite the same email to any role.
- Complete signup with invite code + new password.
- Confirm auto-login succeeds (no "Invalid login credentials" error) and user lands on the correct role dashboard.
