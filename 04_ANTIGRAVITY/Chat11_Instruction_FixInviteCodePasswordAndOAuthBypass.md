# Fix Instruction: Invite Code Verification Bugs (2 issues)

**Context:** Chat 11 — fixes based on confirmed root causes in `Chat11_Investigation_InviteCodeBugs_Result.md`. Implement both fixes below.

## Fix A: Password not updated on manual re-activation signup — `app/api/auth/staff-signup/route.ts`

**Root cause:** For an existing user going through the "re-activation" branch (role exchange via manual `/signup`), the backend calls `updateUserById` but only updates `user_metadata`/role — it never updates the user's password to the one they just typed in the form. Frontend then attempts `signInWithPassword` with the new password, which fails because the account still has the old password. User retries, but by then the invite code is already marked "used", producing the misleading "Invalid, unused, or expired invite code" error.

**Fix:**
- In the re-activation branch of `staff-signup/route.ts` (~line 70), update the `updateUserById` call to also set the new password (`password: <the password from the request body>`), not just metadata/role.
- Confirm this only happens in the correct branch (existing-user re-activation) and doesn't affect brand-new user creation, which should be unaffected.

## Fix B: Google OAuth callback blindly restores stale roles, bypassing invite verification — `app/auth/callback/route.ts`

**Root cause:** At the callback (~lines 22-25), if `user.user_metadata.role` (`metadataRole`) has any value left over from a prior role (even if the user was later deactivated/demoted to customer), the callback unconditionally writes this stale role into `profiles.role` and grants dashboard access — without ever checking for a valid, unused invite code. This means a previously-deactivated staff member can regain their old role automatically via Google sign-in, with no invite code required.

**Fix:**
- Remove the unconditional `if (metadataRole) { update profiles.role = metadataRole }` behavior.
- Role assignment via Google OAuth must only happen through the same invite-code verification path used elsewhere (`verify-invite` logic) — i.e., the callback should check whether there's a valid, unused invite code for this email before assigning any non-customer role. If no valid invite code exists, the user's `profiles.role` should remain as their current DB-stored role (default `customer` if none), not whatever is stuck in stale `user_metadata`.
- As part of this fix, also ensure `user_metadata.role` gets cleared/reset whenever a user is deactivated (tie this into the existing deactivation logic if it isn't already happening) so stale role data doesn't linger for future OAuth logins. Flag this as a follow-up if it requires touching the LOCKED deactivation node — do not modify that node directly without flagging it first.

## Constraints

- Do not touch the deactivation/hard-delete logic itself (LOCKED node) — if Fix B requires a metadata-clearing addition there, stop and flag it as a separate confirmation needed, don't modify silently.
- Do not touch the already-verified email fixes (Fix 1-4 from `Chat11_Instruction_FixDuplicateInviteAndMissingWelcomeEmail.md`).
- After implementing, run `npm run build` and report clean output as evidence — no manual UI testing needed from Antigravity side, Ayush will do that.

## Deliverable

Build result saved to `03_Investigation_and_Errors/` as `Chat11_Evidence_BuildResult_InviteCodeFixes.md`, plus a short note listing exactly which files were changed, and explicitly flagging if Fix B required (or was blocked by) touching the deactivation node.
