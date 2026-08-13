# Chat 9: Investigation — Navbar Structure & Staff Data Schema

## 1. Navbar Structure Findings
- **Component Path:** `components/Navbar.tsx`
- **Structure:** The navbar is a shared UI component, but it is **not** rendered via a global Next.js layout (`app/dashboard/layout.tsx` does not exist). Instead, the `<Navbar />` component is manually imported and placed inside the `return` statement of each individual page (e.g., `dashboard/page.tsx`, `menu/page.tsx`, `tables/page.tsx`).
- **Why `/dashboard/staff` is missing it:** The `app/dashboard/staff/page.tsx` file simply forgot to import and include the `<Navbar />` component at the top of its JSX structure. It is an easy addition.
- **Why Staff isn't in the links:** In `Navbar.tsx`, the block rendering links for `role === 'owner'` currently hardcodes Overview, Menu, Tables, Orders, Analytics, and AI Insights. It simply lacks a `<Link href="/dashboard/staff">Staff</Link>`.

## 2. Staff Data Schema
The system uses a combination of two tables for staff:
### A. `invite_codes`
- **Columns:** `id`, `code`, `role`, `staff_name`, `staff_email`, `status`, `created_by`, `created_at`
- **What "status" means:** Here, `status` only refers to the invite code itself (`'unused'`, `'used'`, or `'expired'`). It does not represent whether the staff member is currently employed or suspended.
- **What "created_at" means:** The time the owner generated the invite code, NOT the time the staff actually joined.

### B. `profiles`
- **Columns:** `id`, `email`, `role`, `created_at`
- **What "created_at" means:** This is the actual **join date** (when the staff member signed up and their profile was created).
- **Missing Columns:** There is no `name` column here. There is also no `status` (active/inactive/suspended) column for actual staff accounts.

## 3. Data Availability for "Staff Details" Tab
To build a "Staff Details" view showing Name, Email, Role, Join Date, and Status, we are currently facing a few schema limitations:
1. **Name:** Not stored in `profiles`. To get the name, we would need to join `profiles.email` with `invite_codes.staff_email` (where status is 'used').
2. **Email & Role:** Available directly in `profiles`.
3. **Join Date:** Available directly in `profiles` as `created_at`.
4. **Status (Active/Inactive):** ❌ **Missing.** There is currently no way to mark a staff account as active vs suspended/inactive in the database. 

**Conclusion:** 
To properly implement a robust "Staff Details" view, we will likely need a small DB migration to add `name` and `status` (or `is_active`) directly to the `profiles` table.

Awaiting your instructions on how you'd like to proceed with the fixes!
