# Walkthrough: Role-Gate Enforcement for Deactivated Staff

The permanent ban logic has been removed and replaced with a robust role-based deactivation system. Staff can now be deactivated and reactivated seamlessly.

## 1. Permanent Ban Removed & Role Downgraded
Updated **[route.ts](file:///C:/Users/ayush/Desktop/vibethon_project/app/api/staff/deactivate/route.ts)**:
- Stripped out the `updateUserById` call that previously placed a permanent 100-year Auth ban.
- The `profiles` table is now updated during deactivation to set `role: 'customer'`, alongside `is_active: false` and `is_logged_in: false`.

## 2. Active State Enforced at Route & DB Level
Updated **[middleware.ts](file:///C:/Users/ayush/Desktop/vibethon_project/middleware.ts)**:
- The middleware now queries `is_active` alongside `role`.
- If `is_active === false`, the middleware forcefully overrides the user's role to `'customer'`, kicking them out of any staff dashboard and redirecting them to `/order`.

Created **[20260809000007_enforce_active_role.sql](file:///C:/Users/ayush/Desktop/vibethon_project/supabase/migrations/20260809000007_enforce_active_role.sql)**:
- Added an `AND is_active = true` condition inside the core `has_role()` PostgreSQL function.
- This serves as a secondary defense layer, ensuring deactivated staff lose all RLS table access even if their stored role hasn't fully updated yet.

> [!CAUTION]
> **Action Required**: Please run `20260809000007_enforce_active_role.sql` in your Supabase SQL Editor manually to enforce the new RLS policy.

## 3. Re-Activation Flow Fixed
Updated **[route.ts](file:///C:/Users/ayush/Desktop/vibethon_project/app/api/auth/staff-signup/route.ts)**:
- The staff signup route now intelligently checks if the user's email already exists in `profiles`.
- If it exists (e.g., re-inviting a deactivated staff member), it skips `createUser()` and instead updates the existing profile (`is_active: true`, `role: <newRole>`) and syncs the Auth metadata.
- If it's a completely new user, the standard `createUser()` flow proceeds.
- This completely prevents the "User already registered" crash during re-invites.

## Verification
- Next.js build compilation and type-checking passed seamlessly (`npm run build`).
