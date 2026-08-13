# Chat 11 — Node Lock: Deactivation/Welcome Emails + Invite Code Bugs

## Nodes LOCKED this chat

### Node: Deactivation Email + Welcome Email
- **Bug 1 (duplicate invite email):** Fixed — `disabled={loading}` guard in `AuthForm.tsx`, `useRef` guard replacing state check in `select-role/page.tsx`.
- **Bug 2 (missing welcome email on `verify-invite` path):** Fixed — welcome email logic ported into `app/api/auth/verify-invite/route.ts`; Resend error logging added (non-blocking) in both `staff-signup` and `verify-invite` routes.
- **Verified:** Fresh `/signup` flow (screenshots, 17:26-17:29) — single invite email, welcome email received, deactivation email unaffected (regression-checked).

### Node: Invite Code Verification Bugs
- **Bug A (manual re-activation — "Invalid, unused, or expired invite code"):** Root cause was password not being updated on re-activation (`staff-signup/route.ts`), causing auto-login failure → forced retry → code already marked used. Fixed by updating password in the `updateUserById` call.
- **Bug B (Google OAuth bypassing invite verification — security gap):** Root cause was `app/auth/callback/route.ts` blindly restoring stale `user_metadata.role` without checking for a valid invite code. Fixed — stale-role auto-restore removed; OAuth logins with a pending unused invite code are now routed through `/auth/select-role` to properly redeem it. Confirmed this did NOT require touching the locked hard-delete deactivation node (hard-delete already wipes `user_metadata`).
- **Verified (evidence, this session):**
  - Manual signup (Cook role, code `LC0OL9J`): succeeded on first try, no error, welcome email + dashboard access confirmed.
  - Google OAuth role-exchange (Manager role, code `W2MNFM9`): login correctly redirected to `/auth/select-role` with invite code pre-filled (not skipped to dashboard); code redeemed properly; role updated to Manager, status Active.

## Status

Both nodes: ✅ **LOCKED**. Pushed to GitHub (user-confirmed, done outside this bridge).

## Next

Awaiting next task/node for Chat 12.
