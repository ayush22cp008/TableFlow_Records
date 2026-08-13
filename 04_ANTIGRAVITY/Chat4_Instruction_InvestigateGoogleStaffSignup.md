# TableFlow — Chat #4 — Instruction (Investigation Only): Google Auth Staff Signup

**To:** Antigravity
**From:** Claude
**Type:** Investigation — NO code changes, NO fixes in this prompt

---

## Scope (strict)

Investigate ONLY the **Google-auth signup path for staff roles (Cook/Waiter/Manager)**.

**Explicitly OUT of scope — do not investigate or touch:**
- Customer signup (Google or otherwise)
- Owner signup (Google or otherwise)
- The non-Google "Create Account" path (the one showing "Staff Email" field) — that's a separate, already-working path, not part of this investigation

## Bug Report

1. User signed up via Google, selected a staff role on the "Create Account → Who are you joining as?" screen (this screen exists and renders correctly — confirmed via screenshot).
2. After Google auth, the screen for staff roles shows a **"Staff Invite Code"** field (different from the non-Google path, which shows "Staff Email" — this difference is expected/known, not the bug).
3. Separately, in an earlier incident, a user went through Google Sign-Up and ended up directly in the **Owner dashboard** with no role selection happening at all — suggesting the Google + staff-role code path may have a bug where role selection is sometimes skipped entirely, or the invite code validation doesn't correctly gate/assign the role.

## What to Investigate

1. Find the code handling the Google-auth signup flow specifically for staff roles (likely in `components/AuthForm.tsx` or wherever the "Create Account → Who are you joining as?" screen lives, filtered to the Google-auth branch).
2. Trace exactly what happens when a user picks Cook/Waiter/Manager AND signs up via Google:
   - Does it correctly require and validate the Staff Invite Code before creating the profile?
   - Is there any code path where Google auth completes and a profile gets created with a default role (e.g. `owner` or `customer`) WITHOUT going through the invite-code-gated staff flow?
3. Check whether the invite code validation logic is shared between the Google path and the non-Google path, or duplicated/diverged — if duplicated, that's likely where the bug lives (one path enforces it, the other doesn't).
4. Confirm: is it possible for someone to complete Google sign-in and land with a default role assigned, bypassing the "Who are you joining as?" screen and invite code check entirely? If yes, identify exactly which condition causes that bypass.

## Do NOT

- Do not fix anything yet — investigation and evidence only
- Do not touch Customer/Owner signup logic
- Do not touch the non-Google staff signup path

## Output

Save findings to: `03_Investigation_and_Errors/Chat4_Investigation_GoogleStaffSignup.md`

Include: exact file(s)/function(s) involved, the specific code path that lets role assignment get bypassed (if found), and whether invite code validation is properly enforced on the Google-auth branch.

Report back to Claude with the file path once done.
