# TableFlow — Chat #4 — Instruction (Fix): Google Auth Staff Role + Invite Email

**To:** Antigravity
**From:** Claude
**Type:** Fix — implement based on two locked investigations

---

## Reference

- `03_Investigation_and_Errors/Chat4_Investigation_GoogleStaffSignup.md`
- `03_Investigation_and_Errors/Chat4_Investigation_GoogleInviteEmailNotSent.md`

## Root Cause (confirmed, single root cause, two symptoms)

`app/auth/select-role/page.tsx` (the Google-auth path) is an incomplete page:
- It shows the role picker + "Staff Invite Code" input, but has no working logic behind it
- It never calls `/api/send-invite` (unlike `components/AuthForm.tsx`, which does)
- It never properly validates the invite code server-side — the current approach (if any) is client-side and blocked by RLS, since the Google-created profile defaults to `customer` role, and only `owner` can validate/consume invite codes

Meanwhile, `/signup` (non-Google path, `components/AuthForm.tsx`) already has this fully working: calls `/api/send-invite`, validates codes server-side via an admin-key API route, and correctly assigns roles.

## Fix Approach (locked with Ayush)

**Do NOT copy `/signup`'s logic wholesale** — it creates a brand-new user from scratch. On the Google path, the user already exists (Google auth already created their `auth.users` + `profiles` row with default `customer` role) by the time they reach `/auth/select-role`. The fix must **update** the existing profile's role, not create a new user.

### Scope (strict — confirmed with Ayush)
- Fix applies ONLY to staff roles (Cook/Waiter/Manager) on `/auth/select-role`
- Do NOT touch Customer or Owner role handling on this page or anywhere else
- Do NOT touch `/signup` or `components/AuthForm.tsx` — those already work correctly

## Build Tasks

1. **On `/auth/select-role`, when a staff role (Cook/Waiter/Manager) is selected:**
   - Reuse the same underlying server-side/admin-key validation logic that `/signup`'s invite code flow uses (likely refactor into a shared function/API route both pages call, rather than duplicating code) — but adapt it to **update** the current authenticated user's existing profile (set `role` to the selected staff role) instead of creating a new user.
   - This server-side route uses the Supabase service-role key, bypassing the RLS block that currently prevents a `customer`-role user from validating/consuming an invite code.

2. **Wire up the missing invite email trigger:**
   - When the Google-authenticated user's email matches a pending invite code (same auto-detect logic `/signup` already has), call `/api/send-invite` the same way `AuthForm.tsx` does.
   - Confirm this fires correctly for the Google path specifically — this was the exact gap found (zero send attempts previously).

3. **Mark invite code as used** upon successful role assignment, same as the existing working flow does — prevents reuse.

## Do NOT

- Do not touch Customer or Owner logic anywhere
- Do not modify `/signup` or `components/AuthForm.tsx` (already correct, working reference implementation)
- Do not change the visual UI of `/auth/select-role` — it already renders correctly, only the logic behind it needs fixing

## Verification Before Reporting Back

- `npm run build` — 0 errors
- Test: sign up via Google with a NEW test Gmail account, select "Cook," enter a valid pending invite code tied to that email — confirm role gets correctly updated to `cook` in `profiles` table, invite code marked used, and (if applicable per the auto-detect flow) confirm invite email logic doesn't errantly re-fire after code is already entered manually
- Test: confirm a Google sign-up with an email that has a PENDING invite (not yet entered) receives the invite email automatically, matching what `/signup` already does
- Confirm Customer/Owner Google sign-up flows are unaffected (regression check)

## Output

Report back: files touched, confirmation of test results above. Do NOT push to GitHub — Ayush triggers that manually after review.
