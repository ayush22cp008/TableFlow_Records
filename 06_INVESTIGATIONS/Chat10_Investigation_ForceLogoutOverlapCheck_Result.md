# Investigation Result: Force Logout vs Deactivate Overlap Check

**Type:** Investigation Result (Chat 10)

## Context
Checking if the Force Logout feature overlaps with the Deactivate API route (`app/api/staff/deactivate/route.ts`).

## Findings

**1. Force Logout Implementation:**
- Force logout is implemented via a Postgres RPC function called `force_logout_all_staff()`.
- **Location:** Defined in `supabase/migrations/20260809000006_realtime_session_status.sql` and triggered from the frontend at `app/dashboard/staff/page.tsx`.
- **Mechanism:** It explicitly runs `DELETE FROM auth.sessions` for all staff members, and runs an `UPDATE` on the `profiles` table to set `is_logged_in = false` for everyone with a staff role.

**2. Deactivate Implementation:**
- The deactivate route (`app/api/staff/deactivate/route.ts`) is a separate API endpoint that uses the Supabase Admin SDK.
- **Mechanism:** It targets a single user profile and runs `.update({ is_active: false, is_logged_in: false, role: 'customer' })`. 

**3. Overlap Check Answer:**
Are there duplicate logic or shared functions? **No. They are fully independent code paths.**
- One is a global, database-level RPC that mass-clears sessions directly from `auth.sessions`.
- The other is an API-level profile update for a single user.
- *Note:* The deactivate route does not actually delete the user's token from `auth.sessions` like force logout does; it relies on the `role: customer` downgrade to kick them out during the next middleware check.

**Conclusion:** 
No refactor is needed to decouple them, as they are already separate concerns. We can safely proceed with the deactivate fix plan from Chat 9.
