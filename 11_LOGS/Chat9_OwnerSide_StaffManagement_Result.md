# Walkthrough: Owner Side Staff Management Updates

The comprehensive staff management updates for the Owner's dashboard have been successfully implemented. 

## 1. Database Schema & RPCs

A new SQL migration file has been created:
**[20260809000005_staff_management_updates.sql](file:///C:/Users/ayush/Desktop/vibethon_project/supabase/migrations/20260809000005_staff_management_updates.sql)**

This migration adds critical backend capabilities:
- **`is_active` & `last_login`**: Added to the `profiles` table to track staff status natively.
- **Login Trigger (`sync_last_sign_in_at`)**: Automatically syncs `auth.users.last_sign_in_at` to `profiles.last_login` whenever a staff member signs in, ensuring the data is easily accessible on the client.
- **End of Day Logout RPC (`force_logout_all_staff`)**: A secure postgres function restricted to the Owner role that deletes all active sessions for waitstaff, cooks, and managers.

> [!CAUTION]
> **Action Required**: Please run this migration in your Supabase SQL Editor manually. 

## 2. Server-side Deactivation API

Created a new secure API route:
**[app/api/staff/deactivate/route.ts](file:///C:/Users/ayush/Desktop/vibethon_project/app/api/staff/deactivate/route.ts)**

This endpoint securely wraps the Supabase Admin SDK (`supabaseAdmin`). It checks that the caller is an authenticated owner, then applies a 100-year ban (`ban_duration: '876000h'`) on the selected staff member in the `auth.users` table.

## 3. Frontend Enhancements

### Navbar
- Added the `<Link href="/dashboard/staff">Staff</Link>` link in the Owner navigation block within **[Navbar.tsx](file:///C:/Users/ayush/Desktop/vibethon_project/components/Navbar.tsx)**.

### Staff Management Page
The **[app/dashboard/staff/page.tsx](file:///C:/Users/ayush/Desktop/vibethon_project/app/dashboard/staff/page.tsx)** file was completely overhauled:
1. **Integrated Navbar**: The page now includes the top navigation bar.
2. **Force Logout Button**: An "End Day — Log Out All Staff" button was added at the top. When clicked, it asks for confirmation and executes the `force_logout_all_staff` RPC.
3. **Active Staff Details Table**: 
   - A new primary data table was added above the invite codes section.
   - It fetches `profiles` (filtered by `is_active = true`) and joins them with `invite_codes` to display the staff member's real name.
   - Computes an **Active / Inactive** badge by verifying if the `last_login` timestamp occurred within the last 48 hours.
   - Provides a **Delete** action per row that prompts for confirmation, hits the new `/api/staff/deactivate` endpoint, and removes them from the UI.

## Verification
- The Next.js build compiled perfectly and type-checks passed.
- The UI handles errors gracefully with `confirm()` dialogs to prevent accidental logouts and deletions.

The system is fully operational from a codebase perspective!
