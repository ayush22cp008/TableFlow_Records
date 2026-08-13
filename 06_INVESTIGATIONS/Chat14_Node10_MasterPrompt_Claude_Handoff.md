# Master Prompt — Claude Side — Chat 14 (Node 10 continued)

## Project
TableFlow — restaurant management system, role-based dashboards (Owner, Manager, Waiter, Cook, Customer).

## Node Map
- Nodes 1–8: ✅ LOCKED
- Node 9 (Notifications System): 🔄 PAUSED — blocked on Node 10
- Node 10 (Owner/Staff Role Overlap Cleanup): 🔄 ACTIVE
  - Fix A (Reservation RLS security gap): ✅ **LOCKED** — verified working (Manager approve/reject tested live, Owner read-only confirmed)
  - Fix B (Owner read-only: Orders, Billing, Tables, Waitlist): 🔄 **SQL run pending** — Ayush executing final SQL now, result to be reported in next chat
  - Fix C (Menu — reverse case: Owner keeps access, Manager loses it): ⬜ NOT STARTED — blocked on Fix B completion

## Key architectural decision (Node 10)
Manager becomes sole operational authority across Orders, Billing, Tables, Reservations, Waitlist. Owner becomes read-only in these areas, retaining only "Bulk Emergency Stop" (separate `SECURITY DEFINER` RPC path, unaffected by RLS changes). **Exception: Menu is reversed** — Owner keeps `is_available` toggle access, Manager loses it (Fix C, not yet started).

## Fix B — current status (as of end of Chat 13)
Frontend changes already applied and built clean by Antigravity (confirmed in `Chat13_Node10_FixB_Result.md`):
- `app/dashboard/orders/page.tsx` — advanceStatus/submitCancel hidden for Owner, Bulk Emergency Stop untouched
- `app/dashboard/billing/[orderId]/page.tsx` — fully blocked for Owner ("Access Restricted")
- `app/dashboard/tables/page.tsx` — cycleTableStatus/setReservation/clearReservation/seatWaitlistEntry/cancelWaitlistEntry all Manager-only conditional render

**Correction made in Chat 13 (important — supersedes original Fix B SQL):** Investigation (`Chat13_Node10_Investigation_WaiterRLS.md`) confirmed `waiter` role was pre-existing (not newly added) in `tables_write`/`waitlist_update` policies. Waiter Dashboard UI (screenshot-verified) only touches `orders` table (Mark Served flow) — never touches `restaurant_tables` or `waitlist`. Per Ayush's decision, narrowed both policies to `manager`-only for least-privilege.

**Final SQL Ayush is running (saved in `02_Instructions/Chat13_Node10_FixB_FinalSQL_WaiterRemoved.md`):**
```sql
-- 1. Orders: drop owner's access completely
DROP POLICY IF EXISTS "owner_all_updates" ON orders;

-- 2. Tables: manager only
DROP POLICY IF EXISTS "tables_write" ON restaurant_tables;
CREATE POLICY "tables_write_staff" ON restaurant_tables FOR ALL
USING (has_role(ARRAY['manager']))
WITH CHECK (has_role(ARRAY['manager']));

-- 3. Waitlist: manager only
DROP POLICY IF EXISTS "waitlist_update" ON waitlist;
CREATE POLICY "waitlist_update_staff" ON waitlist FOR UPDATE
USING (has_role(ARRAY['manager']));
```

Rollback SQL (if post-deploy issues) is in the same file — restores original `owner_all_updates`, `tables_write` (waiter+manager+owner), `waitlist_update` (waiter+manager+owner).

## What Ayush will bring to Chat 14
Result of running this SQL (success/failure), then push + Vercel deploy test:
- Owner should NOT see: Orders status buttons, Billing page (blocked message), Tables Reserve/Clear, Waitlist Seat/Cancel
- Owner SHOULD still see: Bulk Emergency Stop
- Manager should retain full access to all of the above
- Waiter dashboard (Mark Served flow) should be unaffected

## Next task once Fix B is confirmed working
Write Fix C instruction — Menu page (`app/dashboard/menu/page.tsx`, shared component like Tables was):
- Frontend: hide `is_available` toggle for **Manager**, keep for **Owner** (reverse of Fix A/B pattern)
- RLS: drop `manager` from `menu_write` policy, keep `owner`
- Same safety pattern: forward SQL + rollback SQL, no push until Ayush reviews

## Standing workflow rules (unchanged)
- Claude = architecture/diagnosis/instructions only. Antigravity = code execution. Ayush = manual browser testing.
- Investigation and fix always separate prompts.
- No GitHub push without Ayush's explicit go-ahead, given immediately after each verified checkpoint.
- All DB migrations run manually via Supabase SQL Editor (CLI unavailable on this machine).
- Testing method: **deploy-based via Vercel** (localhost `npm run dev` currently fails — root cause found in Chat 13: Antigravity was running it from the Drive bridge folder `G:\My Drive\TableFlow_Staff_Role_System` instead of the actual project code folder, so `package.json` wasn't found — not yet re-verified from the correct directory, but deploy-based testing works and is the agreed default for now since no real users are on production).
- Every RLS/DB fix instruction must include rollback SQL.
- Manual DB changes logged in `04_Logs/` immediately after execution.
- File naming: `Chat{N}_Node{M}_{Type}_{ShortDescription}.ext`
- Instruction files → `02_Instructions/` only (Claude can create there without asking). All other files need Ayush's permission first, every time.

## Key Drive folder IDs
- `02_Instructions/` = `13NcntSWMoqGG105X-wp8Mfu8KpUlASd8`
- `01_Master_Prompts/Claude_Side/` = `1iWwBytbROljqLIE0ZkTEZD0L9mzXmTFQ`
- `04_Logs/` = `13ishjnY1mDlm_xK1yuFxlDnVoIMVf4Nj`
