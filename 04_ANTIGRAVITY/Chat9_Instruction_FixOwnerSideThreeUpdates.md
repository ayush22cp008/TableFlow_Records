# Instruction — Fix: Owner Side 3 Updates (Navbar + Staff Details, Approach B)

**Chat #9 — Fix only. Investigation already complete (see `Chat9_Investigation_NavbarAndStaffSchema.md` and `Chat9_Investigation_LastLoginStaffStatus.md`). Do not re-investigate.**

## Scope — 3 updates

### Update 1: Add "Staff" link to navbar
In `components/Navbar.tsx`, add a `<Link href="/dashboard/staff">Staff</Link>` (or matching label) to the owner-role links block, alongside Overview, Menu, Tables, Orders, Analytics, AI Insights.

### Update 2: Add navbar to Staff Management page
In `app/dashboard/staff/page.tsx`, import and include `<Navbar />` at the top of the JSX, matching how it's done in `app/dashboard/menu/page.tsx` / `tables/page.tsx`. No other layout changes.

### Update 3: Staff Details section (Approach B — DB trigger)

**3a. Migration:**
- Add `last_login` (timestamptz, nullable) column to `profiles`.
- Add a Postgres trigger/function that updates `profiles.last_login` from `auth.users.last_sign_in_at` whenever a user signs in. (Use Supabase's standard pattern — e.g. a trigger on `auth.users` update, or an `on auth state change` hook if trigger-on-auth-schema isn't permitted; report whichever mechanism is used.)
- Since this is a new column, backfill `last_login` for existing profiles from their current `auth.users.last_sign_in_at` value (one-time backfill in the same migration) so existing staff don't show as "Inactive" incorrectly on first load.

**3b. Staff Details tab/section (same page, per Ayush's decision):**
- Add a new section/tab within `app/dashboard/staff/page.tsx`, separate from the existing "Invite Codes History" table.
- Data source: `profiles` table (id, email, role, created_at as join date, last_login), joined/matched with `invite_codes` (where `status = 'used'`) on email to pull `staff_name`.
- Columns to display: **Name, Email, Role, Join Date** (`profiles.created_at`), **Status** (Active if `last_login` within 48 hours, else Inactive — green/gray badge matching existing UI patterns like the "used" badge style).
- If a `profiles` row has no matching `invite_codes.staff_name` (edge case), display email as fallback for name — don't break the row.

## Constraints
- Do not touch Node 3, Node 4, or the Reservation Requests panel fix — all locked/done, not in scope.
- Keep the fix minimal — no unrelated refactors to Navbar or Staff page beyond what's described.
- Migration file only — do not run `db push` (Supabase CLI unavailable). Ayush will run it manually via SQL Editor.
- No manual UI testing on your side — Ayush will test manually and confirm before push.
- Report back: files touched, exact diff/snippet per update (1, 2, 3a, 3b), migration file content, and build/compile result.
