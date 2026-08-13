# Fix Plan: Deactivate = Full Hard Delete (Supersedes role-downgrade + password-fix)

**Type:** Fix (implementation) — based on `Chat10_Investigation_DeactivateFullDeleteFeasibility_Result.md`.

**Supersedes:**
- The "role: 'customer'" downgrade behavior from `Chat9_Implementation_FixRemoveBanProperRoleGate.md`.
- `Chat10_Instruction_FixReinvitePasswordBug.md` — no longer needed, since the `existingProfile` re-activation branch this fix targeted will no longer trigger.

**Context:** Owner confirmed staff never use the app as customers, so the `orders.customer_id` / `waitlist.customer_id` / `feedback.customer_id` FK data-loss risk flagged in the investigation does not apply. Full delete is safe.

## Fix

### [MODIFY] `app/api/staff/deactivate/route.ts`
- Remove the current logic that sets `role: 'customer'`, `is_active: false`, `is_logged_in: false` on the profile.
- Replace with `supabaseAdmin.auth.admin.deleteUser(id)`.
- Confirm (via schema check during implementation) whether `public.profiles` has `ON DELETE CASCADE` tied to `auth.users`. If yes, no separate profile delete needed. If not, explicitly delete the `profiles` row in the same request after the auth user is deleted.

### [NO CHANGE NEEDED] `app/api/auth/staff-signup/route.ts`
- The `existingProfile` branch simply won't trigger anymore for deactivated emails, since the profile + auth user will no longer exist. Re-invited staff go through the normal new-user `createUser()` path — already working correctly.

### [NO CHANGE NEEDED] `middleware.ts`, `has_role()`, force logout
- Investigation confirmed all three already handle a missing profile gracefully (treated as logged-out/no role). No changes required.

## Verification Plan
### Automated
- `npm run build` to confirm no type/syntax errors.

### Manual (Ayush)
- Deactivate a staff member (any role).
- Confirm they disappear from Active Staff immediately.
- Confirm the same email can be freshly re-invited by the owner and completes signup as a brand-new user (no invalid-credentials error).
- Confirm a deactivated person's old email, if they visit the app directly, is treated as a logged-out guest (no crash, no lingering staff access).
