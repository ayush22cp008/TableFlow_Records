# Node 2b: Staff Roles, Invite Codes, and RLS Implementation

This document outlines the architecture and execution steps for implementing the Staff Role System schema, the new `invite_codes` mechanism, and migrating the entire application's Row Level Security (RLS) to a granular, role-based model.

## Proposed Changes

### Database Schema Migration (`supabase/migrations/...`)
I will create a new SQL migration file that performs the following schema updates:

#### 1. Profiles Table Extension
- Drop the existing `CHECK` constraint on `profiles.role`.
- Add a new `CHECK` constraint to allow: `'customer'`, `'owner'`, `'waiter'`, `'cook'`, `'manager'`.

#### 2. Invite Codes Table (`invite_codes`)
- **Columns:**
  - `id` (uuid, primary key, default gen_random_uuid())
  - `code` (varchar(7), unique, not null)
  - `role` (text, not null, check in ('waiter', 'cook', 'manager'))
  - `staff_name` (text, not null)
  - `staff_email` (text, not null)
  - `status` (text, not null default 'unused', check in ('unused', 'used', 'expired'))
  - `created_by` (uuid references profiles(id), not null)
  - `created_at` (timestamptz, not null default now())
- **RLS:**
  - Owner: Full CRUD (Select, Insert, Update for fixing typos)
  - Public/Customer: Select (to validate the code on signup)

#### 3. pg_cron Expiry Mechanism
Since the decision dictates a real deletion mechanism using `pg_cron` for PII protection:
- **Step 1:** Create an SQL function `mark_expired_invite_codes()` that updates `status = 'expired'` where `used = false` and `created_at < now() - interval '30 minutes'`.
- **Step 2:** Create an SQL function `delete_expired_invite_codes()` that deletes rows where `status = 'expired'`.
- **Step 3:** Schedule these via the `cron` extension to run periodically (e.g. every 5 minutes).

> [!WARNING]
> Using `pg_cron` on Supabase requires the extension to be enabled and scheduled by a superuser (postgres role). The SQL migration will include the code, but you may need to execute it directly in the Supabase SQL Editor if standard migrations run with lesser privileges.

#### 4. RLS Policy Overhaul
Replace `is_owner()` policies with robust helper functions and per-role policies:
- **Helper Functions:** `has_role(allowed_roles text[])`
- **Orders:** 
  - Insert: Customers only.
  - Select: Customers (own), Waiters, Cooks, Managers, Owners.
  - Update: Cooks (Preparing->Ready), Waiters (Ready->Served), Managers (Placed->Preparing, Completed/Paid), Owners (Completed/Paid).
- **Order Items:**
  - Select/Update: Match Orders visibility.
- **Restaurant Tables & Waitlist (Reservations):**
  - Select: Public (for status), Waiter, Manager, Owner.
  - Update/Insert: Waiter, Manager, Owner.
- **Menu Items:**
  - Select: Public (active/available).
  - Update/Insert: Manager, Owner.
- **Profiles:**
  - Select: Self, Owner.
  - Update: Self.

---

### Application/Frontend Changes

#### [NEW] `types/index.ts` Updates
- Update `UserRole` type to include `'waiter' | 'cook' | 'manager'`.
- Add `InviteCode` type.

#### [MODIFY] Authentication UI (`app/auth/select-role` or `app/signup`)
- Expand role selection to include Waiter, Cook, Manager.
- Render conditional input for "7-character Invite Code" when a staff role is selected.
- **Validation Logic:** 
  - Fetch invite code from DB.
  - Ensure code exists, `status === 'unused'`, and `role` matches selected role.
  - Validate that `signup_email === staff_email` from the code.
- Mark code as `used` in the same transaction/flow after successful user creation.

#### [NEW] Staff Management Dashboard (`app/dashboard/staff/page.tsx`)
- Owner-only view to generate new invite codes.
- Forms to input staff name, email, and select role (generates a random 7-char code with 'WT', 'CO', or 'MN' embedded randomly).
- List of active (unused) codes to allow typo correction.

## Verification Plan

### Automated Tests
- Run `npm run build` to verify type compliance.

### Manual Verification
- **Code Validation:** Try to sign up with a mismatched email, mismatched role, or expired code and ensure rejection.
- **Staff Signup:** Generate an invite code as Owner, use it to sign up successfully, and verify the `profiles` table reflects the correct role.
- **RLS Verification:** Log in as a Cook and attempt to change a table's status (should fail). Attempt to change an order from Preparing to Ready (should succeed).

---
## User Review Required

> [!IMPORTANT]
> - Do you approve the `pg_cron` setup via migration, or would you prefer I just provide the SQL script for you to run manually in the dashboard?
> - The order status transitions in RLS will be strictly enforced using SQL `WITH CHECK` on the `status` column transition (e.g. checking `OLD.status` vs `NEW.status` combined with `has_role()`). Is this strict DB-level enforcement preferred?
