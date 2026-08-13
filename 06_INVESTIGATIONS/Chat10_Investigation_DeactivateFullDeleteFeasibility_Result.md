# Investigation Result: Switch Deactivate from Role-Downgrade to Full Delete

**Type:** Investigation Result (Chat 10)

## Findings

### 1. What deleting fully requires
- **Requirement:** We **MUST** delete the user from Supabase Auth using `supabaseAdmin.auth.admin.deleteUser(id)`. 
- **Reason:** Deleting only the `public.profiles` row leaves the email registered in `auth.users`. If we don't delete `auth.users`, the email remains "taken", and the user will get a "User already registered" error if they try to sign up as a customer or if the owner tries to re-invite them (the signup route would fail on `createUser`). 
- *Note:* Deleting from `auth.users` typically cascades to `public.profiles` automatically, depending on the initial schema setup.

### 2. Foreign Key / Data Dependency Check
This is the biggest risk of a hard delete:
- **`invite_codes.created_by`**: References `profiles.id`. However, RLS policies dictate that only `owners` can create invite codes. Since we are only deleting staff (not owners), this constraint won't be violated.
- **`orders.customer_id`, `waitlist.customer_id`, `feedback.customer_id`**: These reference `profiles.id`. If a staff member has ever used the app as a customer (e.g., ordering food on their break, leaving feedback), a full delete will either:
  1. **Fail with an error** (if the FK is `ON DELETE RESTRICT`), blocking the owner from deactivating them.
  2. **Silently delete their entire order history** (if the FK is `ON DELETE CASCADE`). 
  
*Conclusion on dependencies:* Hard deleting staff destroys their data as a customer. The current "role downgrade" approach safely preserves their customer history while revoking staff access.

### 3. Current Deactivate Route Dependencies
If the profile row is completely gone, the rest of the codebase handles it gracefully:
- **Middleware:** `middleware.ts` safely handles a missing profile using optional chaining and fallbacks (`let role = profile?.role ?? 'customer'`). It won't crash; it will just treat them as a logged-out guest/customer.
- **RLS (`has_role`)**: Returns `false` if no profile is found. Safe.
- **Force Logout**: Skips them since they no longer have a staff role in the table. Safe.

### 4. Re-invite Path Impact
- If both `auth.users` and `public.profiles` are successfully hard-deleted, the `app/api/auth/staff-signup/route.ts` will **bypass** the `existingProfile` re-activation branch entirely. 
- It will flow down into the normal `createUser()` path just like a brand new user, which perfectly aligns with the requested behavior.

## Summary
Switching to a full hard-delete is technically feasible and the routing/signup paths will support it. **However, it risks data loss (or database constraint errors) if the staff member has any past orders or waitlist entries as a customer.** The current role-downgrade approach is safer for data integrity, but if Ayush prefers full deletion, we must ensure we handle the `orders.customer_id` foreign key correctly.
