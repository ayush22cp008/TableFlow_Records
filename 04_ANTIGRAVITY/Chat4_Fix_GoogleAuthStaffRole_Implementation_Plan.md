# Fix Google Auth Staff Role + Invite Email

This plan addresses the missing invite email and broken RLS validation for staff roles signing up via Google Auth.

## Proposed Changes

### 1. Refactor Invite Validation into Shared Admin Logic
I will create a new secure API route specifically for authenticated users (like those coming from Google Auth) to validate and redeem invite codes.

#### [NEW] `app/api/auth/verify-invite/route.ts`
- **Purpose**: Authenticated endpoint to validate an invite code and assign a staff role.
- **Logic**:
  - Fetch the current user session to securely get `user.email` and `user.id`.
  - Use `supabaseAdmin` to query `invite_codes` (bypassing RLS).
  - Verify the code is `unused`, matches `user.email`, and matches the requested `role`.
  - If valid, use `supabaseAdmin` to update the `profiles` table to the new `role`.
  - Use `supabaseAdmin` to update the `invite_codes` table to `status = 'used'`.
  - Return success.

### 2. Update `app/auth/select-role/page.tsx`
#### [MODIFY] `app/auth/select-role/page.tsx`
- **Email Trigger (`useEffect`)**: 
  - Add a `useEffect` that listens to the selected `role`.
  - If the role is a staff role (`waiter`, `cook`, `manager`), and we haven't already sent an email for this role in this session, silently call `fetch('/api/send-invite')` with the user's email and selected role. This auto-detects pending invites and emails them, mirroring the behavior of `/signup` without altering the UI.
- **Validation (`handleRoleStep`)**:
  - Replace the broken client-side validation logic for staff roles.
  - Instead of querying and updating `invite_codes` client-side (which fails due to RLS), make a `POST` request to the new `/api/auth/verify-invite` endpoint.
  - If successful, redirect to `/dashboard` or `/order` based on the role.
  - **Constraint Respected**: Leave the `owner` and `customer` role logic exactly as it is.

## Verification Plan

### Automated Tests
- Run `npm run build` to ensure 0 compilation errors.

### Manual Verification (to be done by User after deployment)
- **Email Delivery**: Sign in via Google with an email that has a pending invite. Select the matching staff role on the `/auth/select-role` screen. Verify that the invite code email is sent.
- **Role Assignment**: Enter the invite code and click "Continue". Verify the user is redirected correctly, their `profiles.role` updates to the staff role, and the `invite_codes.status` updates to `used`.
- **Regression**: Verify that selecting `owner` with the master code still works, and selecting `customer` still works.
