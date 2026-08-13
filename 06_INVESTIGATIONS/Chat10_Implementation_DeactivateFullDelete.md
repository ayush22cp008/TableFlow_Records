# Goal: Fix Deactivate to Perform Full Hard Delete

**Type:** Implementation Plan (Chat 10)

This plan updates the staff deactivation flow to perform a full hard delete of the user instead of downgrading their role to 'customer'. This supersedes previous role-downgrade fixes and resolves the re-invite password issue since deactivated staff will now flow through the clean new-user signup path.

## Proposed Changes

### 1. Update Deactivate API Route

#### [MODIFY] `app/api/staff/deactivate/route.ts`
- Remove the current update logic (`is_active: false`, `is_logged_in: false`, `role: 'customer'`).
- To ensure no foreign key constraint errors occur (in case `public.profiles` lacks `ON DELETE CASCADE` tied to `auth.users`), explicitly delete the `public.profiles` row first.
- Next, call `supabaseAdmin.auth.admin.deleteUser(userId)` to completely remove the user from Supabase Auth.
- Add proper error handling for both delete operations.

### 2. No Changes Required
- **`app/api/auth/staff-signup/route.ts`**: Will naturally process re-invites for these emails as brand new signups because the `existingProfile` check will no longer match.
- **`middleware.ts`, `has_role()`, and force logout**: Already safely handle non-existent profile rows by defaulting to customer/logged-out states.

## Verification Plan

### Automated Tests
- Run `npm run build` to confirm no type/syntax errors.

### Manual Verification
- Deactivate a staff member from the Owner dashboard.
- Verify they are immediately removed from the "Active Staff" list.
- Re-invite the same staff member using their email.
- Complete the signup process with the new invite code and verify no "Invalid login credentials" error occurs.
- Verify that attempting to visit the dashboard directly with the old email treats the user as a logged-out guest.
