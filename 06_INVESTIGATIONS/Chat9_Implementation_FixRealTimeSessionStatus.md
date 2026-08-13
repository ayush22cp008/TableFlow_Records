# Goal: Real-Time Session-Based Staff Status

This plan addresses the gap where staff members appear "Active" for 48 hours even after logging out. We will introduce a real-time `is_logged_in` tracking mechanism to ensure exact accuracy of staff availability on the Owner Dashboard.

## Proposed Changes

### 1. Database Migration
#### [NEW] [20260809000006_realtime_session_status.sql](file:///C:/Users/ayush/Desktop/vibethon_project/supabase/migrations/20260809000006_realtime_session_status.sql)
- **Column:** Add `is_logged_in` (boolean, default false) to the `profiles` table.
- **Login Trigger Update:** Modify `sync_last_sign_in_at()` to also set `is_logged_in = true` alongside `last_login`.
- **Backfill:** Update profiles to `is_logged_in = true` if their current `last_login` is within the last 48 hours (soft transition for active sessions).
- **Force Logout RPC Update:** Modify `force_logout_all_staff()` to not only delete `auth.sessions`, but also execute `UPDATE public.profiles SET is_logged_in = false WHERE role IN ('waiter', 'cook', 'manager');`.
- **RLS Policy:** To guarantee that the client-side logout hook works, add an explicit RLS policy `profiles_update_own` allowing users to update their own profile (`id = auth.uid()`).

### 2. Client-Side Logout Hook
#### [MODIFY] [AuthContext.tsx](file:///C:/Users/ayush/Desktop/vibethon_project/lib/AuthContext.tsx)
- Intercept the `signOut()` function.
- Execute `await supabase.from('profiles').update({ is_logged_in: false }).eq('id', user.id)` immediately before `supabase.auth.signOut()` is called, ensuring the database instantly registers the logout.

### 3. Staff Details Status Update
#### [MODIFY] [page.tsx](file:///C:/Users/ayush/Desktop/vibethon_project/app/dashboard/staff/page.tsx)
- **Data Fetching:** Select the new `is_logged_in` field when fetching `profiles`.
- **Status Logic:** Completely remove the "48-hour from last_login" logic. Replace it with `const isActive = staff.is_logged_in`.

### 4. Delete/Deactivate Flow Consistency
#### [MODIFY] [route.ts](file:///C:/Users/ayush/Desktop/vibethon_project/app/api/staff/deactivate/route.ts)
- During a staff deletion/deactivation, update the profile to set both `is_active: false` and `is_logged_in: false`, ensuring immediate status clearance in the UI.

## Verification Plan
### Automated Tests
- Run `npm run build` to ensure type-safety and syntax correctness across the modified TypeScript files.

### Manual Verification
- **Ayush will manually run the SQL migration** via the Supabase SQL Editor.
- Ayush will test logging in as a Waiter/Cook on a separate device and verifying the status immediately shows "Active".
- Ayush will test clicking "Log out" on that device and verifying the Owner Dashboard immediately updates to "Inactive".
