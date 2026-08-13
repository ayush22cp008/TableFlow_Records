# Goal: Complete Owner Side Staff Management Updates

This plan implements the 5 final updates for the Owner's Staff Management functionality, enabling proper tracking of staff logins, secure session termination at the end of the day, and safe staff removal (soft-delete + ban).

## Proposed Changes

### 1. Database Migration
#### [NEW] [20260809000005_staff_management_updates.sql](file:///C:/Users/ayush/Desktop/vibethon_project/supabase/migrations/20260809000005_staff_management_updates.sql)
- **Columns:** Add `is_active` (boolean, default true) and `last_login` (timestamptz) to `public.profiles`.
- **Trigger:** Create a Postgres trigger on `auth.users` to automatically sync `last_sign_in_at` to `profiles.last_login` upon login.
- **Backfill:** Execute an `UPDATE` query to backfill `last_login` for existing profiles from `auth.users`.
- **RPC:** Create `force_logout_all_staff()` (SECURITY DEFINER), restricted to the Owner role, which runs `DELETE FROM auth.sessions WHERE user_id IN (...)` for all staff roles.

### 2. Backend Integration (Deactivate Staff)
#### [NEW] [route.ts](file:///C:/Users/ayush/Desktop/vibethon_project/app/api/staff/deactivate/route.ts)
- Create a new API route `/api/staff/deactivate`.
- Verify the caller is an authenticated Owner.
- Use `supabaseAdmin.auth.admin.updateUserById(id, { ban_duration: '876000h' })` to securely ban the staff account.

### 3. Frontend Updates
#### [MODIFY] [Navbar.tsx](file:///C:/Users/ayush/Desktop/vibethon_project/components/Navbar.tsx)
- Add `<Link href="/dashboard/staff">Staff</Link>` into the Owner's navigation block.

#### [MODIFY] [page.tsx](file:///C:/Users/ayush/Desktop/vibethon_project/app/dashboard/staff/page.tsx)
- **Navbar:** Import and wrap the page content with `<Navbar />` for seamless navigation.
- **Data Fetching:** Fetch active staff profiles (`is_active = true`) and join with used `invite_codes` (by email) to resolve the Staff Name.
- **Staff Details UI:** Build a new section displaying a table of active staff with columns: Name, Email, Role, Join Date, Status.
  - **Status Logic:** Green "Active" badge if `last_login` is within the last 48 hours; otherwise, gray "Inactive" badge.
- **Deactivate Button:** Add a "Delete" button per staff row (with a confirmation prompt) that triggers the `/api/staff/deactivate` route and locally sets `profiles.is_active = false`.
- **Force Logout Button:** Add an "End Day — Log Out All Staff" button with a confirmation prompt that calls `supabase.rpc('force_logout_all_staff')`.

## Verification Plan

### Automated Tests
- Run `npm run build` to ensure TypeScript compilation and Next.js page generation succeed without errors.

### Manual Verification
- **Ayush will manually run the SQL migration** via the Supabase SQL Editor.
- Verify the new "Staff" link appears in the Navbar for Owners.
- Verify the Navbar correctly renders on the Staff Management page.
- Verify the "Staff Details" list correctly calculates the Active/Inactive badge based on the 48-hour logic.
- Verify clicking "Delete" prompts for confirmation, hides the row, and correctly hits the ban API.
- Verify clicking "End Day" prompts for confirmation and executes the RPC successfully.
