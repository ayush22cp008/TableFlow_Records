# Goal: Fix Role-Gate Enforcement for Deactivated Staff

This plan resolves the issues where deactivated staff could still bypass the system via their old role, and re-inviting deactivated staff crashed the signup process. We will remove the permanent ban and rely on `is_active = false` alongside a role downgrade.

## Proposed Changes

### 1. Remove Permanent Ban & Downgrade Role
#### [MODIFY] [route.ts](file:///C:/Users/ayush/Desktop/vibethon_project/app/api/staff/deactivate/route.ts)
- Remove the `supabaseAdmin.auth.admin.updateUserById(id, { ban_duration: '876000h' })` call completely.
- When deactivating, update the profile to: `is_active: false`, `is_logged_in: false`, and `role: 'customer'`.
- This ensures the user instantly loses their staff privileges on the server side and becomes a normal customer.

### 2. Enforce `is_active` at Route + RLS Level
#### [MODIFY] [middleware.ts](file:///C:/Users/ayush/Desktop/vibethon_project/middleware.ts)
- When evaluating the user's role, also fetch their `is_active` status.
- If `is_active === false`, override their role in memory to `'customer'` for routing logic, forcibly kicking them out of staff dashboards even if the database downgrade failed.
#### [NEW] [20260809000007_enforce_active_role.sql](file:///C:/Users/ayush/Desktop/vibethon_project/supabase/migrations/20260809000007_enforce_active_role.sql)
- Update the `has_role()` PostgreSQL function to require `AND is_active = true`. This provides defense-in-depth at the DB policy level.

### 3. Fix Re-Invite / Re-Activation Flow
#### [MODIFY] [route.ts](file:///C:/Users/ayush/Desktop/vibethon_project/app/api/auth/staff-signup/route.ts)
- Before calling `createUser()`, query the `profiles` table to see if the email already exists.
- **If existing:** Do not call `createUser()`. Instead, run an update on `profiles` to set `role: newRole, is_active: true, is_logged_in: false`. Also call `updateUserById` to sync `user_metadata.role`.
- **If new:** Proceed with the current `createUser()` logic.
- Both paths will mark the invite code as used upon success.

## Verification Plan
### Automated Tests
- Run `npm run build` to verify Next.js routing and API syntax correctness.

### Manual Verification
- **Ayush will manually run the SQL migration** via the Supabase SQL Editor.
- Ayush will deactivate an existing staff member and confirm they can no longer access their dashboard and are treated as a customer.
- Ayush will generate a new invite code for that same deactivated staff email and sign up, verifying the flow succeeds without crashing.
