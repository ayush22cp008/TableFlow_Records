# Instruction — Fix: Owner Side Complete Staff Management Update (Navbar + Details + Force Logout + Delete)

**Chat #9 — Fix only. Investigations already complete:**
- `Chat9_Investigation_NavbarAndStaffSchema.md`
- `Chat9_Investigation_LastLoginStaffStatus.md`
- `Chat9_Investigation_ForceLogoutAndDelete.md`

**Do not re-investigate. This supersedes `Chat9_Instruction_FixOwnerSideThreeUpdates.md` — implement everything below instead (that earlier instruction was not yet executed).**

## Scope — 5 updates, all on Owner side

### 1. Navbar link
In `components/Navbar.tsx`, add a `<Link href="/dashboard/staff">Staff</Link>` to the owner-role links block.

### 2. Navbar on Staff page
In `app/dashboard/staff/page.tsx`, import and include `<Navbar />` at the top of the JSX, matching `menu/page.tsx` / `tables/page.tsx`.

### 3. Staff Details section (Approach B — DB trigger for status)
- Migration: add `last_login` (timestamptz, nullable) to `profiles`. Add a trigger that updates it from `auth.users.last_sign_in_at` on sign-in. Backfill existing profiles' `last_login` from their current `auth.users.last_sign_in_at` in the same migration.
- New section/tab in `app/dashboard/staff/page.tsx`, separate from "Invite Codes History".
- Data: `profiles` (id, email, role, created_at as join date, last_login, is_active — see #5), joined with `invite_codes` (status='used') on email for `staff_name`.
- Columns: Name, Email, Role, Join Date, Status.
- **Status logic:** Active if `last_login` within 48 hours AND `is_active = true`; otherwise Inactive. (48-hour window remains the fallback signal; #4's force logout makes this accurate in practice since staff get properly logged out at close.)
- Fallback to email if no matching `invite_codes.staff_name` found.

### 4. Force Logout All Staff (End of Day button)
- Create a Postgres RPC function `force_logout_all_staff()` (`SECURITY DEFINER`), restricted to Owner role, that runs:
  ```sql
  DELETE FROM auth.sessions
  WHERE user_id IN (SELECT id FROM profiles WHERE role IN ('waiter', 'cook', 'manager'));
  ```
  Confirm role-based access restriction inside the function (only callable by an authenticated Owner — check `auth.uid()` against `profiles.role = 'owner'` before executing the delete).
- Add a button (e.g. "End Day — Log Out All Staff") in the Staff Management page (owner-only visibility) that calls `supabase.rpc('force_logout_all_staff')`.
- Add a confirmation prompt before executing (this logs out everyone immediately — avoid accidental clicks).

### 5. Delete Staff (Soft Delete + Ban)
- Migration: add `is_active` (boolean, default `true`) to `profiles`.
- Delete button in the Staff Details section per row. On click (with confirmation prompt):
  - Set `profiles.is_active = false` for that user.
  - Call Admin API server-side (new route, e.g. `/api/staff/deactivate`) to ban the account: `supabaseAdmin.auth.admin.updateUserById(id, { ban_duration: '876000h' })`.
- Leave `invite_codes` rows untouched (historical record).
- Deactivated staff should not appear in the active Staff Details list (filter `is_active = true` for the default view) — but don't delete the row, just filter it out.

## Constraints
- Do not touch Node 3, Node 4, or the Reservation Requests panel fix — all locked/done.
- Server-side Admin SDK calls (ban) must go through a new API route — never expose the service role key client-side.
- Migration file(s) only — do not run `db push`. Ayush will run manually via SQL Editor.
- Keep fixes organized — clearly separate the migration(s) from frontend changes in your report.
- No manual UI testing on your side — Ayush will test manually and confirm before push.
- Report back: files touched, exact diff/snippet per update (1-5), migration file content, and build/compile result.
