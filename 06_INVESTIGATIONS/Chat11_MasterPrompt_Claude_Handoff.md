# TableFlow — Claude-Side Master Prompt (Chat 11 Handoff)

## Node Map
- ✅ LOCKED: Node 1, 2b, Routing, Cook Dashboard, Manager Dashboard, Waiter Dashboard, Node 4 (double-booking), Reservation Requests panel cleanup
- ✅ LOCKED: Owner Side Staff Management (navbar, Staff Details, force logout, delete) — pushed
- ✅ LOCKED: Deactivate → Full Hard Delete — pushed
- ✅ LOCKED: Invite Codes Cleanup — pushed, verified
- ✅ LOCKED: Deactivation Email + Welcome Email — pushed, verified this chat (see below)
- ✅ LOCKED: Invite Code Verification Bugs (password re-activation + Google OAuth bypass) — pushed, verified this chat (see below)
- ⬜ NOT STARTED: Notifications (major node, next up — not yet scoped)
- ⬜ DEFERRED: Owner-editable custom onboarding guidelines in welcome email — explicitly out of scope, over-engineering for current scale

## What Got Done This Session (Chat 11)

### 1. Deactivation Email + Welcome Email node — closed out
Picked up from Chat 10's pending investigation. Two bugs found and fixed:
- **Duplicate invite email:** `AuthForm.tsx` "Continue" button missing `disabled={loading}` (double-click race) + `select-role/page.tsx` `useEffect` double-firing in Strict Mode with an async state check that couldn't block the second fire in time. Fixed: button disabled during loading; state check replaced with a synchronous `useRef` guard.
- **Missing welcome email:** Only existed in `staff-signup/route.ts` (fresh `/signup` flow). The `verify-invite/route.ts` path (already-logged-in user doing a role exchange via `/auth/select-role`) had no welcome-email logic at all. Ported the same Resend logic into `verify-invite/route.ts`. Also added Resend `{data, error}` checking with `console.error` logging in both routes so future email failures aren't silent — logging only, doesn't block the main flow.
- **Verified:** fresh `/signup` flow — single invite email, welcome email received, deactivation email still correct (regression check passed).

### 2. New bugs surfaced during that verification → separate investigation + fix cycle
While testing, two more bugs turned up around invite code verification (not part of the original email scope, tracked separately):
- **Bug A — manual re-activation showing "Invalid, unused, or expired invite code":** Root cause was `staff-signup/route.ts`'s re-activation branch updating the user's role/metadata via `updateUserById` but never updating their password to the one just typed. Frontend's immediate auto-login (`AuthForm.tsx` line ~127) then failed silently, user retried, and by the second attempt the invite code was already marked "used" — producing a misleading error. Fixed by including `password` in the `updateUserById` call.
- **Bug B — Google OAuth bypassing invite code verification entirely (security gap):** `app/auth/callback/route.ts` was unconditionally restoring `user_metadata.role` into `profiles.role` on every Google sign-in, regardless of whether a valid invite code existed. This meant a previously-deactivated staff member's old role could silently come back via Google login with zero invite-code check. Fixed: removed the blind restore; OAuth logins now check for a pending unused invite code for that email and route through `/auth/select-role` to redeem it properly, exactly like the manual flow.
  - Confirmed this fix did NOT require touching the LOCKED hard-delete deactivation node — hard delete already wipes `user_metadata` entirely, so no stale-role residue exists for accounts that went through proper deactivation.

**Verified (both, this session, with screenshots):**
- Manual signup (Cook role) — succeeded first try, no error, welcome email delivered, dashboard access correct.
- Google OAuth role-exchange (Manager role) — login correctly redirected to `/auth/select-role` with invite code pre-filled instead of skipping straight to dashboard; code redeemed properly; role/status updated correctly in Staff Management.

**Pushed to GitHub — confirmed by Ayush (done outside the Drive bridge).**

## Standing Reminders Still Active
- Investigation and fix always separate prompts.
- All manual DB/migration changes logged in `04_Logs/` immediately.
- No GitHub push without Ayush's explicit go-ahead (this session's push already happened, confirmed).
- Supabase CLI unavailable — all migrations are manual SQL Editor runs.
- Antigravity handles execution/build-check only (`npm run build` evidence); Ayush does all manual browser UI verification with screenshots.
- When a bug surfaces mid-verification that's outside the current node's original scope (like Bugs A/B above), it gets its own investigation → fix cycle rather than being folded into the original node silently.

## Immediate Next Task (start of Chat 12)
Notifications node is next but **not yet scoped**. Before writing any investigation or instruction, need a scoping discussion with Ayush covering:
- What "Notifications" means for TableFlow — in-app toast/banner? Persistent notification center? Push notifications?
- Which roles see which notifications (owner/manager/waiter/cook)?
- Which events trigger them (new order, order status change, low stock, reservation, staff activity, etc.)?

No code/investigation work should start on this node until that scoping conversation happens.
