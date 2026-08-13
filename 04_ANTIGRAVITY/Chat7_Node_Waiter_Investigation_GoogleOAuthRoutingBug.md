# Waiter Dashboard — Google OAuth Routing Bug (Investigation Request)

**Confirmed by Ayush (manual test):**
- Manual "Create Account" signup (email/password) with waiter invite code → correctly lands on `/dashboard/waiter`. Working.
- Google Sign-In with waiter invite code → incorrectly lands on Customer dashboard (`/order`), NOT `/dashboard/waiter`. Bug.

## Why this is suspected (pattern match to Manager's Chat 5 bug)

This is the same shape of bug as Manager's Bug A/B in Chat 5: manual signup and Google OAuth use **different code paths** for role-based redirect, and a role branch fix applied to one path doesn't automatically apply to the other.

Chat 7's `AuthForm.tsx` fix added the `waiter` branch in 3 locations: `handleStaffSignup`, `handleVerifyOtp`, `handleLogin`. These cover manual signup/login. They do NOT cover the Google OAuth callback flow.

**Suspected file (per Manager precedent):** `app/auth/select-role/page.tsx` — this is the page Manager's Bug (Chat 5) was traced to; it handles the Google OAuth invite-code-entry fallback and had its own separate hardcoded redirect ternary (`role === 'manager' ? '/dashboard' : ...`) that was missing branches independently of `AuthForm.tsx`. The `waiter` branch may be missing here too, same as `manager` was before Chat 5's fix.

**Also check:** `app/auth/callback/route.ts` — Manager's Bug 1 (Chat 5) was here too (stale `getUser()` call reading cached cookie). Worth a quick check that this isn't recurring for waiter specifically, though it was already fixed generically for all roles in Chat 5.

## Investigation task (investigation only, no fix yet — per standing rule)

1. Check `app/auth/select-role/page.tsx` for the redirect ternary — confirm whether `waiter` branch exists or falls through to `/order`.
2. Check `app/auth/callback/route.ts` for how it determines `role` and whether waiter is handled correctly post-Chat-5-fix.
3. Report back exact line numbers and current ternary logic, same format as `Chat5_Node3_Investigation_ManagerRoutingBug_Result.md`.
4. Do NOT apply any fix in this pass — investigation report only, fix comes as a separate instruction after Ayush reviews.
