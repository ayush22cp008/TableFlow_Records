# Investigation: Deactivate Role-Downgrade + Re-Invite Login Bugs

**Type:** Investigation only — no code changes.

## Bug 1 — Confirm role downgrade is correct (not actually a bug, verify only)
When a staff member is deactivated, they immediately show `role: 'customer'` and disappear from Active Staff. Confirm this matches the implemented plan (`Chat9_Implementation_FixRemoveBanProperRoleGate_Result.md`) — i.e. this is expected behavior, not a side effect of something else. No fix needed if confirmed correct — just state confirmation.

## Bug 2 — Re-invite signup fails with "Account created, but auto-login failed: Invalid login credentials"

**Repro sequence:**
1. Staff member "kl" was originally created via **Google OAuth** (not email/password).
2. Owner deactivated "kl" (role → customer), then re-invited the same email as a **cook**.
3. User received invite code via email, went to signup, entered invite code + set a password.
4. Result: "Account created, but auto-login failed: Invalid login credentials."

**Investigate:**
1. Look at `app/api/auth/staff-signup/route.ts` — specifically the "existing email found" branch added in the recent fix.
2. Check what auth method the original account used (`kl` — Google OAuth identity in `auth.users`/`auth.identities`).
3. Determine whether the "existing email" update path assumes/requires a password-based account, and whether it's trying to set a password on an account that has no password identity (OAuth-only), causing the login attempt right after to fail with credential mismatch.
4. Confirm exact failure point: does `updateUserById`/password-set succeed but the subsequent `signInWithPassword` (or equivalent) fail because the account's actual auth method is still Google OAuth, not password?

## Output
Report findings only — relevant code snippets, the exact mismatch/gap causing Bug 2, and confirmation on Bug 1. No fix yet.
