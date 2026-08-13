# Google Auth Staff Role Fix Completed

The fixes for the Google Auth staff signup flow have been successfully implemented and the build has passed with 0 errors.

## Changes Made

1. **New Admin API for Invite Verification:**
   - Created `app/api/auth/verify-invite/route.ts`.
   - This new endpoint retrieves the current authenticated user's session and uses the `supabaseAdmin` service role key to query the `invite_codes` table. By executing server-side with admin privileges, it bypasses the Row Level Security (RLS) restrictions that blocked the previous client-side queries for `customer`-level users.
   - Upon successful code validation, the endpoint updates the `profiles` table to assign the new role and marks the invite code as `used`.

2. **Triggering the Invite Email (`app/auth/select-role/page.tsx`):**
   - Added a `useEffect` hook to automatically call the `/api/send-invite` endpoint whenever a staff role (Cook/Waiter/Manager) is selected. 
   - A `sentRoles` state ensures the API is only called once per role selection, preventing duplicate email dispatch if the user clicks back and forth. 
   - This replicates the behavior of the non-Google signup flow, ensuring the email containing the invite code is sent as expected.

3. **Replacing Client-Side Validation (`app/auth/select-role/page.tsx`):**
   - Refactored `handleRoleStep` so that when a staff role is submitted, it posts the entered code to the new `/api/auth/verify-invite` endpoint instead of attempting RLS-blocked client-side updates.
   - Maintained the existing flow and logic for Owner and Customer selections, per requirements.

## Validation Results

- ✅ **Build Check:** Ran `npm run build` which passed successfully with 0 errors.

> [!TIP]
> The code is ready for manual testing. Please deploy these changes and run through the Google signup flow:
> 1. With a new Gmail account, select "Cook" and ensure the invite email is dispatched to your inbox.
> 2. Enter the valid invite code, submit the form, and confirm that your role updates correctly in the dashboard.
> 3. Verify that Owner and Customer signup via Google are unaffected.
