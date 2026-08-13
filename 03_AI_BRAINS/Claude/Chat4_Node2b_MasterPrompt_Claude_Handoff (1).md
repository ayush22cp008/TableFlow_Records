# TableFlow — Chat #4 Master Prompt (Claude Side)
**Handoff from:** Chat #3
**Date locked:** 2026-08-06

---

## Node-Map Status

- ✅ **LOCKED** — Node 1: Core Staff Role System design (5-role RBAC, order lifecycle, invite code system spec)
- ✅ **LOCKED** — Node 2b Part 1: Schema + RLS overhaul + cancellation system (RPCs, invite_codes table, policies) — migrations applied to live Supabase DB and verified 2026-08-06
- ✅ **LOCKED** — Node 2b Part 2: Resend-based invite code email delivery — verified live end-to-end 2026-08-06
- ✅ **LOCKED** — Emergency cancellation system (single-order cancel + Owner bulk emergency stop) — **user manually clicked and tested both cancel flows live on production, confirmed working 2026-08-06**
- ⬜ **NOT STARTED** — Role-specific post-signup dashboard routing verification (see Open Items)
- 🔄 **ACTIVE (Chat #4)** — Node 3: Role-specific desktop/dashboard views for Manager, Cook, Waiter
- 🔄 **ACTIVE (Chat #4)** — Node 4: Notification system (polling/refresh-based, not push)

---

## What Was Closed Out Through Chat #3

### Critical bug found and fixed
Migration file `20260804000001_node2b_schema_rls.sql` was committed to git locally but **never applied** to the live Supabase database — the entire Node 2b Part 1 scope (invite_codes table, RLS overhaul, cancellation RPCs) was missing from production despite being marked complete earlier. Root cause: gap between "committed" and "deployed" for DB-only changes with no build/deploy step to force application.

### Resolution sequence (reusable pattern)
1. Investigation isolated the gap (temp scripts `check_db.js`, `check_scope.js`, deleted after use)
2. Instruction `Chat3_Node2b_Instruction_CleanupAndApplyMigrations.md` → Antigravity applied both `20260804000001_node2b_schema_rls.sql` and `20260804000002_node2b_cancellation.sql` directly to live Supabase
3. Evidence in `Chat3_Node2b_Evidence_MigrationsApplied.md`: table, 4 RPCs, cancellation columns, new RLS policies, zero data loss confirmed, schema cache reloaded via `NOTIFY pgrst, 'reload schema'`
4. **User manually tested live** (per evidence rule — browser verification, not Antigravity-reported):
   - Invite code generation (`/dashboard/staff`) → code `RWTS50B`, Waiter role
   - Resend email delivered correctly with code + role
   - Signup flow completed with code + password
   - Account created, logged in successfully
   - **Single-order cancellation** — tested live, works
   - **Owner bulk emergency stop (cancel all orders)** — tested live, works

No code changes were required for the migration fix — DB-only, so no git commit/push occurred in Chat #3. Repo and live DB are in sync.

---

## Chat #4 Scope (New — defined at handoff)

### Node 3 — Role-Specific Dashboards (Manager, Cook, Waiter)
Build dedicated desktop/dashboard views per role, replacing/supplementing the current landing behavior where staff appear to land on the generic `/order` (customer-facing menu) page after signup/login.

- **Manager dashboard:** scope TBD at Chat #4 kickoff (likely: staff oversight, order oversight, access to cancellation controls per existing RBAC)
- **Cook dashboard:** scope TBD (likely: incoming orders queue, order status updates — e.g. preparing → ready)
- **Waiter dashboard:** scope TBD (likely: table/order status, serving queue, ability to mark orders delivered)
- Must resolve the open redirect question below as part of this node — where each role should land post-login is core to this node's definition of done.

### Node 4 — Notification System
- Refresh-based (polling), **not** push notifications — explicitly not real-time push, re-check periodically
- Needs full detailed spec — user indicated more details will be provided at Chat #4 kickoff
- Scope to be defined: what events trigger a notification (new order, order cancelled, status change?), which roles see which notifications, UI placement, refresh interval

**Both nodes need full requirements gathering at the start of Chat #4** — this handoff captures intent and direction only, not final specs.

---

## Open Items Carried Into Chat #4

1. **Role-dashboard redirect not yet verified.** After Waiter signup, user landed on `/order` (visually the customer menu/ordering page — "Add to Cart", "Sign Out"), not a distinct Waiter view. This is now directly in scope for Node 3 — needs resolving as part of dashboard build, not as a separate bug fix.

---

## Reference — Unchanged Standing Context

- Stack: Next.js 14, TypeScript, Tailwind, Supabase (Postgres/Auth/Realtime/RLS), Gemini, Resend (tableflow.systems domain), Vercel
- Live site: table-flow-nu.vercel.app
- No local testing — sequence is always build → commit+push to GitHub → Vercel auto-deploy → user tests live
- Bridge folder (coordination only, not source): `TableFlow_Staff_Role_System`, root ID `16Xhn59dqZg2_XVMnqulmLeywO3CfhzyL`
- GitHub = source of truth for code; bridge folder = source of truth for coordination/decisions only
- Role division unchanged: Claude = architecture/diagnosis/specs; Antigravity = code execution/terminal only; User = manual browser testing + all final decisions
- Evidence rule: user sends screenshots or explicit manual-verification confirmation ("maine check kiya hai" / direct confirmation) as proof of live testing — this handoff itself was updated only after explicit confirmation that cancellation was live-clicked, not just DB-verified
