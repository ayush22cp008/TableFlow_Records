# Investigation: Switch Deactivate from Role-Downgrade to Full Delete

**Type:** Investigation only — no code changes.

## Context
Current deactivate behavior sets `role: 'customer'` on the profile. Ayush wants to change this: deactivation should **delete the staff member entirely** instead of converting them into a customer. If that person wants to be a customer, they sign up fresh on their own. If they need staff access again, the owner issues a brand new invite code — same as onboarding a new person.

## Investigate

1. **What deleting fully requires:**
   - Confirm whether deleting only `public.profiles` row is enough, or whether `auth.users` (Supabase Auth) must also be deleted via `supabaseAdmin.auth.admin.deleteUser()`.
   - If `auth.users` is not deleted, check whether that email stays "taken" and blocks a fresh signup later (either as customer or via new invite).

2. **Foreign key / data dependency check:**
   - Search the schema for any tables referencing `profiles.id` (orders, assigned tables, reservations, etc.) via foreign key.
   - For each, determine current `ON DELETE` behavior (CASCADE / RESTRICT / SET NULL / no constraint).
   - Flag anything that would break, error out, or silently orphan data if the profile row is hard-deleted (e.g. a waiter's past order history referencing their `profiles.id`).

3. **Current deactivate route dependencies:**
   - Check `app/api/staff/deactivate/route.ts` — anything else that reads/relies on `is_active`/`role: 'customer'` elsewhere in the codebase (e.g. RLS policies, middleware, `has_role()` function from the recent migration) that would need to change if the row simply won't exist anymore.

4. **Re-invite path impact:**
   - Confirm that if the profile + auth user are both deleted, `staff-signup/route.ts`'s `existingProfile` branch would no longer trigger for that email — it would go through the normal `createUser()` path as a completely new signup.

## Output
Report findings only — what breaks or needs consideration if we switch to full delete, and what's safe. No fix/plan yet.
