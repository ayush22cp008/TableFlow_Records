# Chat #4 | Node 2b/3 | Master Prompt — Antigravity (Execution Handoff)

## 1. Node-Map Status
- ✅ **LOCKED:** Node 1 (Auth skeleton, email signups, repo setup)
- ✅ **LOCKED:** Node 2a (Core Supabase schema structure)
- ✅ **LOCKED:** Node 2b (Staff Roles, `invite_codes`, Resend email delivery, strict RLS paired-policies, bulk cancellation RPC, Staff Management UI wired)
- 🔄 **ACTIVE:** Transitioning to Node 3 (Role-based Dashboards & Redirects)

## 2. Execution Context & Files Touched
**Recently Built / Modified (Chat 3):**
- `components/AuthForm.tsx` (Integrated staff UI, routing, API calls)
- `app/api/auth/staff-signup/route.ts` & `app/api/send-invite/route.ts` (Backend invite handling)
- `lib/supabaseAdmin.ts` (Service role client)
- `app/dashboard/page.tsx` (Wired the Staff Management card)
- `app/dashboard/orders/page.tsx` (Added `bulkError` handling for cancellation)
- `supabase/migrations/20260804000001_node2b_schema_rls.sql` & `20260804000002_node2b_cancellation.sql` (Finally executed against live production DB using a direct Node.js `pg` script)

**Git Status:** 
- Clean. Last commit `726995d` ("feat: Add Staff Management card to Owner Dashboard navigation") successfully pushed to `origin main`.

## 3. Environment & Constraints
- **Vercel Deployments:** In Chat 3, Vercel deployments were failing. The root cause was identified as missing `RESEND_API_KEY` and `SUPABASE_SERVICE_ROLE_KEY` in Vercel's production environment settings. (Local `.env.local` is correct).
- **Tooling Gaps:** `psql` and `vercel` CLI are not available/linked locally. If direct database interaction is needed, we rely on running temporary Node.js scripts using the `pg` package and the Supabase DB password.
- **Testing:** No local browser testing by Antigravity. All testing is manual via Vercel live URLs.

## 4. Current State & Next Tasks (Node 3)
- The database schema is fully up-to-date and the schema cache was refreshed. The "missing `invite_codes` table" issue is resolved.
- **Open Item for Node 3:** Role-dashboard redirect is not yet fully implemented. Currently, after a Waiter signs up, they land on `/order` (the customer ordering view) instead of a dedicated Waiter dashboard. Building the distinct role dashboards and fixing this post-login redirect is the immediate next priority.
