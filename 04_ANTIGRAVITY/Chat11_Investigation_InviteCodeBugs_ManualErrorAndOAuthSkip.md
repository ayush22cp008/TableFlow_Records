# Investigation: Invite Code Verification Bugs (2 issues)

**Context:** Chat 11 — during manual testing of the select-role/role-exchange flow (already-logged-in user being invited to a staff role), two separate issues surfaced. Investigation only — do NOT fix yet.

## Test scenario (role exchange for existing account)

Staff email already has an account (was previously a customer or had a prior role). Owner generates a new invite code for this email for a new role. User is expected to go through `/auth/select-role` (already logged in) to accept the role — this is a *role exchange*, not a new account creation.

## Bug A: Manual signup path — "Invalid, unused, or expired invite code" on first attempt

**Repro steps (as reported):**
1. Owner generates invite code for an email that has a prior/existing account.
2. User attempts to complete signup manually (email/password form) using the invite code.
3. Error shown: **"Invalid, unused, or expired invite code."** — signup fails, user does not reach the role dashboard.
4. Exact retry conditions (same code vs new code, refresh vs no refresh) — user could not confirm precisely, so do not assume; check logs/DB state instead of guessing.

**Investigate:**
- Where does "Invalid, unused, or expired invite code" get thrown from? Find the exact validation check (likely checking `status = unused` and/or expiry timestamp on the invite_codes table).
- Check for a race condition or premature status flip: is the code being marked "used" or expired before the manual-signup flow's second verification step completes?
- Check whether this only reproduces for emails with a **pre-existing account/prior role** (role exchange case) vs a brand-new email — this may be the actual trigger, not the auth method itself.
- Pull server logs / DB row for the specific invite code from this test session if available.

## Bug B: Google OAuth path — invite code verification is being skipped entirely

**Repro steps (as reported):**
1. Same scenario — email has an invite code generated for a new role.
2. User authenticates via **Google OAuth** instead of manual email/password.
3. The invite-code-entry page/step **is skipped entirely** — user lands directly on the new role's dashboard, without ever being prompted for or validating the invite code.

**This is a potential security gap, not just a UX bug.** If invite code validation is bypassed for Google OAuth logins, investigate:
- Which route/callback handles Google OAuth sign-in for existing users (likely a Supabase Auth callback or `/auth/callback` route) — does it check for a pending/valid invite code for that email before assigning a new role?
- Confirm whether role assignment via Google OAuth is currently keyed off *just matching email* rather than requiring a valid, unused invite code — if so, this means anyone whose email happens to match an invited email could get a role via Google sign-in without possessing the actual invite code from their inbox.
- Trace the full OAuth callback code path and identify exactly where (if anywhere) invite code state is checked.

## What NOT to do

- Do not fix either bug in this pass — investigation only, per standing rule.
- Do not touch deactivation/hard-delete logic or the already-verified email-sending fixes (Fix 1-4 from `Chat11_Instruction_FixDuplicateInviteAndMissingWelcomeEmail.md`) — those are locked/verified.

## Deliverable

Root-cause findings for both bugs (relevant code snippets, DB/log evidence, exact file + line references) — saved to `03_Investigation_and_Errors/` as `Chat11_Investigation_InviteCodeBugs_Result.md`. No code changes in this pass.
