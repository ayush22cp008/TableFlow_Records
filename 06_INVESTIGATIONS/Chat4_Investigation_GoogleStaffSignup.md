# Investigation: Google Auth Staff Signup

## 1. Files Involved
The Google Auth flow spans three primary files:
- **`components/AuthForm.tsx`**: Initiates the OAuth flow (`handleGoogleLogin`).
- **`app/auth/callback/route.ts`**: Intercepts the redirect after Google auth. If it detects a new user (`created_at` within 10 seconds), it redirects them to `/auth/select-role`.
- **`app/auth/select-role/page.tsx`**: Presents the "Who are you joining as?" UI, validates the invite code, and updates the profile's role.

## 2. Tracing the Staff Signup Flow (Cook/Waiter/Manager)
**Does it validate the code *before* creating the profile?**
No. By the time the user reaches the `select-role` page, their profile **has already been created**. When Supabase inserts the `auth.users` row upon successful Google auth, the Postgres trigger `handle_new_user()` fires instantly and creates a corresponding row in the `profiles` table with the default role of `customer` (because Google provides no role metadata). The `select-role` screen simply attempts to *update* this existing profile.

## 3. Invite Code Validation Logic (Diverged & Bugged)
The invite code validation logic is **diverged and duplicated**, which introduces a critical bug:
- **Non-Google Path:** Handled in `/api/auth/staff-signup/route.ts` using `supabaseAdmin` (Service Role Key). This bypasses RLS and successfully updates the `invite_codes` table to mark the code as `used`.
- **Google Path:** Handled client-side in `app/auth/select-role/page.tsx` using the standard authenticated client. Because the user currently has a `customer` role profile, their attempt to update the invite code fails. The RLS policy on the `invite_codes` table (`invite_codes_owner_all`) explicitly demands the user be an `owner` to perform updates. Therefore, staff signup via Google is fundamentally broken; the RLS policy will block the code redemption.

## 4. Bypassing the Role Selection Screen
**Is it possible to complete Google sign-in and land with a default role, bypassing the screen entirely?**
Yes, in several ways:

1. **The 10-Second Race Condition:** In `app/auth/callback/route.ts`, the code identifies a "new" user with:
   `const isNewUser = (Date.now() - new Date(user.created_at).getTime()) < 10000`
   If a user takes longer than 10 seconds on the Google consent screen, `isNewUser` evaluates to `false`. They will bypass the `select-role` redirect entirely, landing on `/order` with a default `customer` role.
   
2. **Tab Closure Bypass:** Since the profile is created as `customer` instantly upon Google auth, a user redirected to `/auth/select-role` can simply close the tab or manually type `/order` in the URL bar. They retain a fully functional `customer` account without entering any code.

3. **Lingering Metadata (The Owner Bypass):** If a user previously signed up and obtained an `owner` role, but their `profiles` row was subsequently deleted (leaving their `auth.users` row intact with `user_metadata.role = 'owner'`), a new Google sign-in will evaluate `isNewUser` as false. `route.ts` will detect `metadataRole === 'owner'` and immediately redirect them back to `/dashboard`, bypassing role selection entirely and granting owner access without an invite code.
