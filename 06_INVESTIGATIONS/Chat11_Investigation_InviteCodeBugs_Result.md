# Investigation Results: Invite Code Verification Bugs

## Bug A: Manual signup path — "Invalid, unused, or expired invite code"
**Root Cause:**
This occurs during a "role exchange" for an **existing user** who uses the manual `/signup` flow (AuthForm).
1. The user reaches the `staff-verify` step and enters their invite code along with a password.
2. The form submits to `/api/auth/staff-signup/route.ts`. Because the user exists, it takes the "re-activation" branch. It successfully updates their profile and `user_metadata`, and **marks the invite code as used**.
3. However, the backend **does not update their password** (Admin SDK `updateUserById` only updates metadata here). 
4. The frontend (`AuthForm.tsx` line 127) immediately attempts auto-login using the password they just typed. Since it differs from their existing account password, auto-login fails.
5. The user is left on the form and inevitably clicks "Complete Signup" again.
6. The second request hits the backend, but the invite code is now `used`. The backend returns 400: `"Invalid, unused, or expired invite code."` (The code is no longer unused).

**File & Line Reference:**
- `components/AuthForm.tsx` (Line 127: `signInWithPassword` fails for existing users with new passwords).
- `app/api/auth/staff-signup/route.ts` (Line 70: `updateUserById` fails to sync the new password).

## Bug B: Google OAuth path — invite code verification is being skipped entirely
**Root Cause:**
This is a severe logic flaw in the OAuth callback that forcefully restores stale roles, bypassing the invite system entirely.
1. When a user authenticates via Google OAuth, they are routed to `app/auth/callback/route.ts`.
2. At line 23, the callback executes:
   ```typescript
   if (metadataRole) {
     await supabase.from('profiles').update({ role: metadataRole }).eq('id', user.id)
   }
   ```
3. If a user had a **prior staff role** before being deactivated/demoted, their `profiles.role` was changed to 'customer', but their `user.user_metadata.role` was **never cleared** in Supabase Auth.
4. When they log in via Google OAuth for their new invite, this callback sees their old staff role stuck in `user_metadata`. It blindly overwrites their `profiles.role` with this stale staff role and redirects them straight to the dashboard.
5. The invite code check (`verify-invite`) is completely bypassed because the callback forcefully grants the role based on historical metadata.

**File & Line Reference:**
- `app/auth/callback/route.ts` (Lines 22-25: Unconditional profile role update based on `metadataRole`).
