# TableFlow — Master Prompt (Claude Side) — Chat 16 Handoff

## Project
TableFlow — restaurant management system, role-based dashboards (Owner, Manager, Waiter, Cook, Customer). Next.js 14, Supabase, Vercel, Resend.

## Node Map

- ✅ LOCKED + PUSHED — Node 1–8: Cook/Waiter/Manager Dashboards, staff onboarding, deactivation/welcome emails, Customer Dashboard Revamp
- ✅ LOCKED + PUSHED — Node 10: Owner/Staff Role Overlap Cleanup (Manager = sole operational authority; Owner = read-only + Menu Management + Bulk Emergency Stop, Owner-only)
- ✅ LOCKED + PUSHED — **Node 11: Realtime Coverage** (Chat 15, this handoff) — see full breakdown below
- 🔄 ACTIVE (unblocked, ready to start) — **Node 9: Notifications System**

## Node 11 — Final State (LOCKED, do not re-touch or re-reason)

**Goal:** Convert manual-refresh pages to auto-update via Supabase Realtime, since Node 9 depends on a working realtime foundation.

**Batch 1 (Manager, Cook, Waiter dashboards):** Added `postgres_changes` subscriptions on `orders` table. Channels: `manager_orders_realtime`, `cook_orders_realtime`, `waiter_orders_realtime`. Verified working including tab-switch/background scenarios (one early "Cook not updating" report was a one-off, not a code bug — confirmed via investigation, code was identical across all 3 files).

**Batch 2 (Reservation Status, Staff Management, Menu Management):**
- Files: `app/order/reservation/page.tsx`, `app/dashboard/staff/page.tsx`, `app/dashboard/menu/page.tsx`.
- Channels: `reservation_status_realtime` (table `reservation_requests`, filtered `customer_id=eq.${user.id}`), `staff_management_realtime` (table `profiles`), `owner_menu_realtime` (table `menu_items`).
- **Root cause found:** `profiles` and `reservation_requests` had Realtime replication toggled OFF in Supabase Dashboard (Database → Replication). Frontend code was correct throughout — this was a DB-config issue, not a code bug. Ayush manually toggled both ON.
- **Bug found and fixed:** Customer reservation page (`app/order/reservation/page.tsx`) only handled `['pending', 'approved', 'arrived']` statuses — `'rejected'` was missing, causing (a) reject events to be silently ignored (no UI update) and (b) page refresh to reset to the request form (duplicate-request risk) since `activeReservation` stayed `null`. Fixed: `'rejected'` added to the handled list, plus a new "Rejected" UI state (declined message + "Request Again" button, no auto-form-reveal).

**Batch 3 (Owner Analytics, Insights, Main Dashboard):**
- Only Analytics needed realtime — added `owner_analytics_realtime` channel on `orders` + `order_items` tables in `app/dashboard/analytics/page.tsx`.
- Main Dashboard and Insights were correctly skipped: Main Dashboard is a static nav-link grid with no data query; Insights only fetches on-demand via a "Generate Insights" button (Gemini API cost), so realtime doesn't apply. Confirmed with Ayush — this scoping is correct, not a gap.
- `order_items` replication toggle was checked and confirmed ON by Ayush before final verification.

**All batches verified by Ayush via manual browser testing + confirmed via chat.** All pushed to GitHub.

**Current known Supabase Realtime replication status (`supabase_realtime` publication):**
| Table | Status |
|---|---|
| orders | ON |
| menu_items | ON |
| restaurant_tables | ON |
| waitlist | ON |
| profiles | ON (toggled during Node 11) |
| reservation_requests | ON (toggled during Node 11) |
| order_items | ON (toggled during Node 11) |
| daily_order_counters | OFF (not in scope) |
| feedback | OFF (not in scope) |
| invite_codes | OFF (not in scope) |

## Next: Node 9 — Notifications System

**Full design already locked (originally scoped in Chat 13, refined further in Chat 15 during Node 11 investigation gaps). Do not re-derive — treat as locked spec, go straight to implementation investigation/planning.**

### Channel
In-app only (bell icon + list) — no email, no push. Phase 1.

### Phase 1 Scope (locked)
1. **Order Status changes** — Preparing → Ready → Served (Ready = highest priority, real-time ping)
2. **Order Cancelled** — via Bulk Emergency Stop only (Owner-only feature, confirmed in Node 11 investigation — plain per-order Cancel button was removed in Node 10)
3. **Reservation Approved/Rejected** → Customer, **logged-in customers only** (anonymous customers use existing `unique_code` manual-verify flow in `app/order/cart/page.tsx` — no push notification for them, by design)
4. **New Reservation Request** → Manager only (not Owner — Owner is read-only per Node 10)
5. **New Order Placed** → Manager (existing "Accept (Send to Kitchen)" flow confirmed manual, not auto-triggered — `markPreparing` in `app/dashboard/manager/page.tsx`)

### Deferred to Phase 2
Order Billed/Paid, Waitlist Seated/Cancelled, Staff Role Changed/Force Logout

### Not in scope
Table toggle, new table, menu changes, invite used

### Audience
Customer + Waiter (Waiter specifically pinged on Order Ready)

### Order Cancelled — status-based staff targeting (locked)
| Order status at cancellation | Staff notified |
|---|---|
| Placed | Manager |
| Preparing | Cook |
| Ready | Waiter |
| (any status) | Customer — always |

**Technical note:** `cancel_active_orders` RPC overwrites status directly to `'cancelled'` — old status is not preserved in DB. Targeting must be done by having the frontend snapshot the order's current status *before* calling the RPC (no RPC/schema change needed). Confirmed via investigation: bulk cancel filter is `WHERE status IN ('placed', 'preparing', 'ready')` — `served`/`cancelled` orders are already excluded at the SQL level, so the "customer already eating, order gets cancelled" edge case cannot happen through this feature.

### Cancel reason
Already mandatory in Bulk Emergency Stop modal (Category dropdown + Additional Details text field) — Node 9 just hooks notification-send onto the existing confirm action.

### Bulk cancel notification scope
- "Cancel ALL Active Orders" mode → all active-order customers get the same reason
- "Select Specific Orders" mode → only selected customers get the same reason

### Ground truth confirmed (Node 9 investigation, Chat 15)
- No `notifications` table exists — needs to be created from scratch.
- No bell icon/notification UI exists anywhere — needs to be built from scratch.
- 4 realtime channels already exist and work: `orders_board`, `my_orders_realtime`, `menu_realtime`, `tables_realtime` (note: this is the actual name — `waitlist_realtime` from earlier docs doesn't exist as a separate channel, it's folded into `tables_realtime`).
- Auth/role pattern: `supabase.auth.getUser()` or `useAuth()` context; role via `profiles` table join on `user.id`.
- `reservation_requests.unique_code`: 6-digit string, generated on approval, persists indefinitely. Existing lookup: `verifyReservationCode` in `app/order/cart/page.tsx`.

## What Node 9 needs next (not yet started)
1. Design `notifications` table schema (columns, RLS policies) — not yet done.
2. Design bell icon UI/component placement per dashboard.
3. Investigation into exact insertion points for each of the 5 event types (which existing functions to hook into).
4. **Remember:** any new table needs its Realtime replication toggle checked/enabled manually in Supabase Dashboard (per Node 11 lesson) before assuming realtime will work.

## Standing rules (all still active)
- Investigation and fix always in separate prompts.
- No GitHub push without Ayush's explicit approval — approval requested and given per-checkpoint, not batched.
- All DB migrations run manually via Supabase SQL Editor (CLI unavailable).
- Antigravity: code execution + build/compile check only, no browser UI testing.
- Ayush: manual browser verification, screenshot evidence (or verbal "maine check kiya hai" when screenshot isn't feasible).
- Instruction files → `02_Instructions/` only (no permission needed). Master prompts/specs → ask permission before creating every time.
- File naming: `Chat{N}_Node{M}_{Type}_{ShortDescription}.ext`.
- Full local path always output in copy-paste code block.
- New lesson from Node 11: always check Supabase Realtime replication toggle status before assuming a table will support realtime — don't assume, verify.

## Drive folder IDs (confirmed)
- `02_Instructions/` = `13NcntSWMoqGG105X-wp8Mfu8KpUlASd8`
- `01_Master_Prompts/Claude_Side/` = `1iWwBytbROljqLIE0ZkTEZD0L9mzXmTFQ`
