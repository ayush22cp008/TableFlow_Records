# TableFlow — Claude-Side Master Prompt (Chat 10 Handoff)

## Node Map
- ✅ LOCKED: Node 1, 2b, Routing, Cook Dashboard, Manager Dashboard, Waiter Dashboard, Node 4 (double-booking), Reservation Requests panel cleanup
- ✅ LOCKED: Owner Side Staff Management (navbar, Staff Details, force logout, delete) — pushed
- ✅ LOCKED: Deactivate → Full Hard Delete (replaces old role-downgrade-to-customer approach) — pushed
- ✅ LOCKED: Invite Codes Cleanup — one-time manual wipe of used codes + batch-of-10 auto-delete trigger + manual per-row delete button — pushed, verified working
- 🔄 ACTIVE (investigation sent, awaiting result): Deactivation email + Welcome email on staff join
- ⬜ NOT STARTED: Notifications (major node, next after email work)
- ⬜ DEFERRED (explicitly out of scope for now): Owner-editable custom onboarding guidelines in welcome email — decided this is over-engineering for current scale; revisit only if it becomes a real need later

## Key Architectural Decisions This Session

1. **Deactivation redesigned from role-downgrade to full hard delete.**
   Original plan (`Chat9_Implementation_FixRemoveBanProperRoleGate`) set `role: 'customer'` on deactivate. Ayush decided this was wrong — deactivating staff should not auto-assign them a customer identity. New behavior: deactivate = delete `profiles` row + `auth.users` record entirely (via `supabaseAdmin.auth.admin.deleteUser()`). If the person wants to be a customer, they sign up fresh on their own. If they need staff access again, owner issues a brand-new invite code — goes through the normal `createUser()` signup path, no special-case handling.
   - This also fixed Bug 2 (re-invite signup failing with "Invalid login credentials" for Google OAuth-original accounts) as a side effect — the problematic `existingProfile` branch in `staff-signup/route.ts` no longer triggers since the record won't exist.
   - Confirmed safe: owner will never allow staff to use the app as customers, so the `orders.customer_id`/`waitlist.customer_id`/`feedback.customer_id` FK data-loss risk flagged in investigation does not apply.
   - Verified end-to-end by Ayush (screenshots): deactivated staff vanish from Active Staff, vanish from `auth.users` (checked in Supabase dashboard), old session invalidated, re-using same email via Google Auth correctly shows the "Create Account — who are you joining as?" flow as a brand new user.

2. **Invite Codes History cleanup — three-part decision:**
   - One-time: delete all `status = 'used'` rows (done manually by Ayush, logged in `04_Logs/Chat10_ManualDBChange_InviteCodesCleanup.md`). Table came back completely empty — all existing rows were already "used."
   - Ongoing: Postgres trigger (`AFTER UPDATE` on `invite_codes`, `FOR EACH STATEMENT`) auto-deletes oldest 10 whenever "used" count hits 10. Migration: `supabase/migrations/20260810000001_invite_codes_auto_delete.sql`. Run and verified by Ayush.
   - Manual: per-row "Delete" button added to Invite Codes History UI (works for both used and unused codes), direct Supabase client call relying on existing owner RLS policy — no new API route needed. Verified and pushed.

3. **New email notifications — investigation sent, result pending (next chat):**
   - Deactivation email: notify staff when their access is removed. Explicitly NOT triggered by "End Day — Log Out All Staff" (that's a routine session logout, not account removal — no email needed there).
   - Welcome email: sent when staff completes signup via invite code. Generic message only (restaurant name + role) — no owner-customizable guidelines field. That customization idea was discussed and explicitly deferred as over-engineering for current scale.
   - Investigation file sent: `02_Instructions/Chat10_Investigation_DeactivationAndWelcomeEmails.md`. Covers: reusing existing Resend email pattern, exact hook-in points in `deactivate/route.ts` (capture email BEFORE delete) and `staff-signup/route.ts` (after successful signup), and failure-handling recommendation (email failure should never block the main action).
   - **Next chat starts here: read this investigation's result file when Ayush shares it, then write the fix plan.**

## Standing Reminders Still Active
- Investigation and fix always separate prompts.
- All manual DB/migration changes get logged in `04_Logs/` immediately (not deferred).
- No GitHub push without Ayush's explicit go-ahead — confirmed pushed for all three locked items above.
- Supabase CLI unavailable — all migrations are manual SQL Editor runs, flagged clearly each time.
- Antigravity handles execution/build-check only; Ayush does all manual browser UI verification with screenshot (or verbal "maine check kiya hai") as evidence.

## Immediate Next Task (start of next chat)
Ayush will share the result of `Chat10_Investigation_DeactivationAndWelcomeEmails.md`. From that:
1. Review findings.
2. Write a fix/implementation instruction covering both emails (deactivation + welcome), following the recommended hook-in points and failure-handling approach.
3. After that's verified and pushed, move to the Notifications node (not yet scoped — will need its own discussion on what "Notifications" means for TableFlow: in-app? push? which roles/events?).
