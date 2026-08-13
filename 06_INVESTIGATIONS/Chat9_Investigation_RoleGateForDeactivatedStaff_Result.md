# Investigation Result: Role-Gate Enforcement for Deactivated Staff

**Context:** Investigating the impact of removing the permanent Auth ban (`ban_duration: '876000h'`) and relying solely on `is_active = false` for deactivated staff.

## 1. Route Protection
**Finding:** Currently, there is **no check** for `is_active = true` at the route or database policy level.
- `middleware.ts` only reads `profiles.role` to determine routing. It completely ignores `is_active`.
- The RLS function `has_role()` in the database only checks `role = ANY(allowed_roles)` and ignores `is_active`.

## 2. Behavior if ban is removed today
**Finding:** If the ban is lifted, a deactivated staff member (with `is_active = false`) would be able to log in normally.
- Upon login, `middleware.ts` will redirect them to their staff dashboard (e.g., `/dashboard/waiter`) because their profile role was never stripped.
- Since RLS only checks their role, they will have full read/write access to orders and perform their job normally, completely bypassing the deactivation intent.

## 3. Re-recruitment Flow
**Finding:** If the Owner generates a new invite code for a previously deactivated staff email, the code generation succeeds. However, the signup flow **errors out and crashes**.
- When the user submits the code at `/signup`, the API route (`/api/auth/staff-signup/route.ts`) calls `supabaseAdmin.auth.admin.createUser()`.
- Because the email already exists in `auth.users`, Supabase throws a "User already registered" error. 
- The system currently lacks an "upsert" or "re-activation" logic to handle existing dormant accounts during the invite flow.

## 4. Customer Access
**Finding:** A deactivated staff member cannot use the app as a normal customer.
- Deactivating a user currently only flips `is_active = false`. It leaves their `profiles.role` as `waiter` (or cook/manager).
- If they try to log in to order food, `middleware.ts` sees their old staff role and forcibly redirects them to the staff dashboard, preventing access to the customer menu/ordering views. 

**Conclusion:** Removing the permanent ban requires implementing middleware/RLS checks for `is_active`, upgrading the signup API to handle account re-activation, and ensuring deactivated staff roles are gracefully downgraded to `customer`.
