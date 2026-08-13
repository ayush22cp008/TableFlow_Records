# Instruction — Fix: Remove Permanent Ban, Implement Proper Role-Based Deactivation

**Chat #9 — Fix only. Investigation already complete (see `Chat9_Investigation_RoleGateForDeactivatedStaff_Result.md`). Do not re-investigate.**

## Confirmed gaps (context)
1. `middleware.ts` and DB `has_role()` only check `role`, not `is_active` — deactivated staff would bypass deactivation entirely if just un-banned.
2. Re-invite for a previously deactivated email crashes at signup (`createUser()` fails — "User already registered") since there's no re-activation path.
3. Deactivation leaves `role` unchanged, so a deactivated staff member trying to use the app as a customer gets wrongly redirected to their old staff dashboard.

## Fix — 3 parts

### 1. Remove the permanent ban
In `app/api/staff/deactivate/route.ts`, remove the `supabaseAdmin.auth.admin.updateUserById(id, { ban_duration: '876000h' })` call. Keep `profiles.is_active = false` and `profiles.is_logged_in = false` as before.

**Also downgrade role on deactivation**: in the same deactivate flow, set `profiles.role = 'customer'` when deactivating (per gap #3 — a deactivated staff member should become a normal customer, not a role-less/broken state).

### 2. Enforce `is_active` at route + RLS level
- In `middleware.ts`: when routing based on `profiles.role`, also check `profiles.is_active`. If `is_active = false`, treat the user as a `customer` for routing purposes regardless of their stored `role` (defense in depth alongside #1's role downgrade).
- In the DB `has_role()` function: add `AND is_active = true` to the existing role check, so RLS policies relying on `has_role()` also respect deactivation even if the role somehow wasn't downgraded.

### 3. Fix re-invite/re-activation flow
In `app/api/auth/staff-signup/route.ts`: before calling `supabaseAdmin.auth.admin.createUser()`, check if the email already exists in `auth.users`/`profiles`. If it does (this is a re-invite of a previously deactivated staff member):
- Do NOT call `createUser()` (it will fail on duplicate).
- Instead, update the existing `profiles` row: set `role` to the new invited role, `is_active = true`, `is_logged_in = false` (they haven't logged in yet under the new role).
- Mark the invite code as `used` as normal.
- If the email doesn't exist yet, proceed with the existing `createUser()` flow unchanged.

## Constraints
- Do not touch Node 3, Node 4, or other locked/done work.
- Keep the fix minimal — don't refactor unrelated parts of middleware or the signup route.
- Migration file only if any DB function changes are needed (e.g. `has_role()` update) — do not run `db push`. Ayush will run manually via SQL Editor.
- No manual UI testing on your side — Ayush will test manually and confirm before push. He will specifically test: (a) deactivate a staff member, (b) confirm they can log in as a customer afterward, (c) confirm the Owner can re-invite them to any role and it works without crashing.
- Report back: files touched, exact diff/snippet per part (1-3), migration content if any, and build/compile result.
