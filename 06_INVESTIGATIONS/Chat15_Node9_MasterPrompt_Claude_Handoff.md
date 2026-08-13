# TableFlow — Master Prompt (Claude Side) — Chat 15 Handoff

## Project
TableFlow — restaurant management system, role-based dashboards (Owner, Manager, Waiter, Cook, Customer). Next.js 14, Supabase, Vercel, Resend.

## Node Map

- ✅ LOCKED — Node 1–6: Cook Dashboard, Waiter Dashboard, Manager Dashboard (billing, Mark Paid/table release RPC), staff onboarding (invite codes, Resend)
- ✅ LOCKED — Node 7: Deactivation/welcome emails, staff hard-delete via `supabaseAdmin.auth.admin.deleteUser()`
- ✅ LOCKED — Node 8: Customer Dashboard Revamp (`customer_id` on reservations, reservation nav move, order labels)
- ✅ LOCKED — **Node 10: Owner/Staff Role Overlap Cleanup** (Chat 13–14, this handoff) — see full breakdown below
- 🔄 ACTIVE (next) — **Node 9: Notifications System** — was blocked by Node 10, now unblocked. Not yet started.

## Node 10 — Final State (LOCKED, do not re-touch or re-reason)

**Core decision:** Manager is sole operational authority (Orders, Tables, Waitlist, Billing). Owner is read-only across those + retains Menu Management and Bulk Emergency Stop (Orders).

Sub-fixes completed and verified end-to-end (screenshots confirmed each):

- **Fix A/B — Orders/Tables/Waitlist role split:**
  - Orders: Owner has no status-advance buttons; retains Bulk Emergency Stop. Manager unaffected.
  - Tables: Owner has no Reserve/Clear; Manager retains full control.
  - Waitlist: Owner has no Seat/Cancel/Approve/Reject; Manager retains full control.
  - RLS: `tables_write_staff` (manager), `waitlist_update_staff` (manager), `owner_all_updates` on orders dropped.

- **Fix B — Billing block for Owner:**
  - RLS + route guard on `dashboard/billing/[orderId]` — Owner sees "Access Restricted — Billing operations are now exclusively handled by Managers." Manager has full Billing Queue (receipt, Mark Paid, Print PDF) — confirmed working post Fix D.

- **Fix C — Menu reverse case (Owner-only):**
  - RLS: `menu_write_owner` policy — only Owner can write `menu_items`.
  - Manager: "Menu" nav link removed entirely (`components/Navbar.tsx`); direct URL to `dashboard/menu` shows Access Restricted (route guard added in `app/dashboard/menu/page.tsx`).
  - Owner: full Menu Management unaffected (toggle, edit, remove, add item all working).

- **Fix D — Owner Dashboard Billing card removed:**
  - Billing card removed from Owner Dashboard's card grid (was linking to a blocked route). Owner Dashboard now shows: Live Orders, Menu Management, Tables & Waitlist, Staff Management, Analytics, AI Insights only.
  - Manager Dashboard's own Billing Queue section untouched and confirmed still functional.

**All verification checklists across Fix A–D confirmed by Ayush via screenshot evidence.** GitHub push pending — use the reusable commit-and-push instruction once this handoff is read.

## Next: Node 9 — Notifications System

Not yet scoped in detail this session. Starting fresh in Chat 15 (new chat). No prior decisions made on Node 9 beyond it being next in the map — treat as a fresh investigation/spec task, not a continuation of Node 10 context.

## Standing rules (all still active)
- Investigation and fix always in separate prompts.
- No GitHub push without Ayush's explicit approval — approval requested and given per-checkpoint, not batched.
- All DB migrations run manually via Supabase SQL Editor (CLI unavailable).
- Antigravity: code execution + build/compile check only, no browser UI testing.
- Ayush: manual browser verification, screenshot evidence (or verbal "maine check kiya hai" when screenshot isn't feasible).
- Instruction files → `02_Instructions/` only. Master prompts/specs → ask permission before creating (this file is the exception already covered by this handoff itself).
- File naming: `Chat{N}_Node{M}_{Type}_{ShortDescription}.ext`.
- Full local path always output in copy-paste code block.

## Drive folder IDs (confirmed)
- `02_Instructions/` = `13NcntSWMoqGG105X-wp8Mfu8KpUlASd8`
- `01_Master_Prompts/Claude_Side/` = `1iWwBytbROljqLIE0ZkTEZD0L9mzXmTFQ`
