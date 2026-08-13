# TableFlow — Staff Role System — Master Prompt (Claude Side)

Chat #3 → Chat #4 Handoff | Node 2b, Part 1 + Part 2

## Project
TableFlow Staff Role System. Live app: table-flow-nu.vercel.app. Bridge folder (Drive, coordination only, NOT source code): `TableFlow_Staff_Role_System`. GitHub = source of truth for code.

## Node-Map
- ✅ Node 1 — Permission Matrix (locked)
- ✅ Node 2a — Staff invite-code design (locked)
- 🔄 Node 2b Part 1 — Core schema + RLS overhaul + cancellation system — **code written and pushed, but NOT applied to live DB until this session** (see Critical Context below). Migration application was in progress as of this handoff.
- 🔄 Node 2b Part 2 — Invite Email Delivery Redesign — **built, deployed, and confirmed working on live signup UI** (all 5 roles render correctly, new email-based staff flow visible). Blocked on Part 1's missing migrations to actually generate/send codes.

## Critical Context — Read First

**The single biggest issue this session:** two migration files (`20260804000001_node2b_schema_rls.sql` and `20260804000002_node2b_cancellation.sql`) were written, committed to GitHub, and their evidence reports marked them "verified" — but neither was ever actually executed against the live Supabase database. This was discovered when testing the Staff Management page threw "Could not find the table 'public.invite_codes'".

Investigation (direct DB query via Node `pg` client, bypassing PostgREST cache) confirmed **completely missing from live DB**:
- `invite_codes` table + `mark_expired_invite_codes` / `delete_expired_invite_codes` RPCs
- `cancel_active_orders` RPC (bulk emergency stop)
- `orders.cancellation_reason` / `orders.cancellation_category` columns
- `has_role` RPC
- The full paired-transition RLS overhaul on `orders`/`restaurant_tables`/`menu_items` — live DB is still running old Chat #1 placeholder policies (`orders_own_read`, `tables_public_read`, etc.)

**Lesson learned for future sessions:** "build verified" / "npm run build passed" evidence only confirms code compiles — it does NOT confirm database migrations were applied to the live instance. Add an explicit live-DB-state check to the evidence checklist for any future migration work.

**Status as of this handoff:** The last instruction sent (`Chat3_Node2b_Instruction_CleanupAndApplyMigrations.md`) asked Antigravity to (1) delete two temporary investigation scripts (`check_db.js`, `check_scope.js`) that were never committed but existed locally, and (2) apply both missing migrations to the live DB, then verify. **Result of this instruction is UNCONFIRMED — no evidence file was reported back before this chat ended.** Next chat's first job: check Drive for `Chat3_Node2b_Evidence_MigrationsApplied.md`; if absent, resend the instruction.

## Also still open (lower priority, found during investigation, not yet fixed)
- `app/auth/select-role/page.tsx` (Google OAuth signup fallback path) still uses the OLD staff flow (direct code entry, no email step) — regressed/never updated to match the new `AuthForm.tsx` flow. Needs a separate scoped fix once migrations are confirmed applied.
- Owner invite code is still a hardcoded client-side string `'TableFlow12'` in `AuthForm.tsx` (security gap, parked from earlier — decide if this should be fixed as part of Node 2b or deferred further).

## Key locked decisions (do not re-litigate)
- Staff (Waiter/Cook/Manager) signup flow: role select → email entry → system looks up matching unused/non-expired `invite_codes` row (email+role match) → Resend sends code to that email → staff enters code+password → 3-layer validation → account created via Supabase Admin SDK `createUser()` with `email_confirm:true` → `invite_codes` row marked used → `profiles.role` updated → auto sign-in via `signInWithPassword`.
- Owner/Customer signup: untouched, still uses `supabase.auth.signUp()` + OTP verification (verified via separate investigation this session — confirmed unchanged and correct).
- Resend SDK (not raw fetch) for email. `RESEND_API_KEY` and `SUPABASE_SERVICE_ROLE_KEY` are both now present in Vercel Production env vars (previously missing `SUPABASE_SERVICE_ROLE_KEY` caused a full deployment failure earlier this session — now fixed).
- Service Role key strictly isolated to `lib/supabaseAdmin.ts`, used only in `app/api/send-invite/route.ts` and `app/api/auth/staff-signup/route.ts` — confirmed never in client bundle.
- Cancellation system: single-order cancel (any staff, reason required) + Owner bulk "Emergency Stop" (atomic RPC `cancel_active_orders`, category + optional reason). Exact paired RLS policies per role are documented in `Chat3_Node2b_Evidence_CancellationSystem.md`.
- Order status enum: `'placed' | 'preparing' | 'ready' | 'served' | 'billed' | 'cancelled'` (confirmed from `types/index.ts`).

## Files touched this session (Node 2b Part 2 build)
- `lib/supabaseAdmin.ts` (new)
- `app/api/send-invite/route.ts` (new)
- `app/api/auth/staff-signup/route.ts` (new)
- `components/AuthForm.tsx` (modified — new staff flow UI)
- `app/dashboard/page.tsx` (modified — added 7th "Staff Management" card, links to `/dashboard/staff`)
- `app/dashboard/orders/page.tsx` (modified — accidental scope creep via `git add .`, picked up a pre-existing local fix improving bulk-cancel error surfacing; reviewed, deemed harmless/beneficial, kept as-is)
- `supabase/migrations/20260804000001_node2b_schema_rls.sql` and `20260804000002_node2b_cancellation.sql` — exist in repo, application-to-live-DB status unconfirmed as of handoff (see Critical Context)

## Immediate next actions for Chat #4
1. Check `03_Investigation_and_Errors/` for `Chat3_Node2b_Evidence_MigrationsApplied.md`. If present, review it, then have Ayush re-test Staff Management page live (generate invite code → check email arrives → complete staff signup end-to-end with wrong-code/wrong-role/wrong-email/expired-code negative tests).
2. If evidence file is absent, resend `Chat3_Node2b_Instruction_CleanupAndApplyMigrations.md` to Antigravity.
3. Once migrations confirmed live and Staff Management page works, fix `app/auth/select-role/page.tsx` (separate scoped instruction).
4. After full manual testing passes, use the standard reusable commit+push instruction (`Copy of Reusable_Instruction_CommitAndPush_after_check_local_vs_github.md`) for any remaining code changes — note migrations themselves are DB-only, no code push needed for those.
5. Then close out Node 2b fully and move to next feature (Table Status Board or further refinements, per earlier roadmap notes).

## Standing rules (unchanged, see general-project-setup skill for full detail)
- Claude = architecture/diagnosis/spec-writing. Antigravity = code execution only. Ayush = manual browser testing + all final decisions.
- Investigation and fix always separate prompts.
- Evidence required before any merge/next-step — screenshots or Ayush's explicit manual verification claim also acceptable.
- No push to GitHub without Ayush's explicit go-ahead. No local testing — always build → push → Vercel auto-deploy → test on live site.
- Manual DB/cloud config changes (like this migration situation) must be captured into a committed file immediately going forward — this was the root cause of this session's biggest issue.
