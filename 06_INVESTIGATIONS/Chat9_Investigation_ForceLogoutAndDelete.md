# Chat 9: Investigation — Force Logout & Delete Staff

## A. Force Logout (End of Day) Feasibility
1. **Supabase SDK Limitations:** The Supabase Admin JS SDK does **not** have a simple `admin.signOut(userId)` method to kill sessions by ID. The `signOut` method requires the exact JWT token of the active session, which is not easily accessible server-side.
2. **Fetching active sessions:** Active sessions live in the internal database schema (`auth.sessions`), which the JS SDK cannot directly query.
3. **The Best Solution (Postgres RPC):** Instead of a new API route, the most robust and elegant way to achieve an "End of Day" logout is to write a PostgreSQL function (`SECURITY DEFINER`) that directly deletes the active sessions from the database:
   ```sql
   DELETE FROM auth.sessions 
   WHERE user_id IN (SELECT id FROM profiles WHERE role IN ('waiter', 'cook', 'manager'));
   ```
   We can wrap this in an RPC function (e.g., `supabase.rpc('force_logout_all_staff')`) restricted to the Owner role. The Owner presses a button, the RPC runs, and all staff are instantly logged out on their devices as soon as their token is checked.

## B. Delete Staff Feasibility
1. **Foreign Key Constraints (Blocker for Hard Delete):** The `profiles` table is linked to other tables. Most notably, `invite_codes.created_by` has a strict constraint referencing `profiles.id`. If you try to hard-delete a Manager who generated invite codes, the database will throw a foreign key violation and abort the deletion to prevent orphaned data.
2. **Auth Deletion Cascades:** Using `supabaseAdmin.auth.admin.deleteUser(id)` attempts to delete the user from the `auth.users` table, which cascades to `profiles`, hitting the exact same constraints and failing.
3. **The Safest Approach (Soft Delete + Ban):** 
   A "soft delete" is highly recommended over a hard delete. 
   - **Database level:** We add an `is_active = false` flag (or `status = 'deleted'`) to the `profiles` table. This hides them from the UI but preserves their historical data (like which invite codes they generated).
   - **Auth level:** We use the Admin API (`supabaseAdmin.auth.admin.updateUserById(id, { ban_duration: '876000h' })`) to permanently ban their account, ensuring they can never log back in.
4. **Historical Cleanup:** Their `invite_codes` rows should be left exactly as they are for historical auditing.

**Conclusion:** Both features are entirely feasible but require specific architectural approaches (RPC for force logout, and Soft-Delete + Ban for removal). 
Awaiting your approval to propose an implementation plan that combines all these new Staff Management features!
