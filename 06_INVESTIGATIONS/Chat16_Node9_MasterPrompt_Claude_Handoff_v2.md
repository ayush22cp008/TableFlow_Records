# TableFlow — Master Prompt (Claude Side) — Chat 16 Handoff (v2 — supersedes v1)

**This file supersedes Chat16_Node9_MasterPrompt_Claude_Handoff.md (v1). v1 was missing the bell-icon/login-refresh design points and the new Node 12 (Push Notifications) scope, added after further discussion. Use this v2 file for Chat 16.**

## Project
TableFlow — restaurant management system, role-based dashboards (Owner, Manager, Waiter, Cook, Customer). Next.js 14, Supabase, Vercel, Resend.

## Node Map

- ✅ LOCKED + PUSHED — Node 1–8: Cook/Waiter/Manager Dashboards, staff onboarding, deactivation/welcome emails, Customer Dashboard Revamp
- ✅ LOCKED + PUSHED — Node 10: Owner/Staff Role Overlap Cleanup (Manager = sole operational authority; Owner = read-only + Menu Management + Bulk Emergency Stop, Owner-only)
- ✅ LOCKED + PUSHED — **Node 11: Realtime Coverage** (Chat 15) — see full breakdown below
- 🔄 ACTIVE (unblocked, ready to start) — **Node 9: Notifications System (In-App)**
- ⬜ NOT STARTED — **Node 12: Push Notifications** (depends on Node 9 — see bottom of this file)

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

## Next: Node 9 — Notifications System (In-App)

**Full design already locked (originally scoped in Chat 13, refined further in Chat 15 during Node 11 investigation gaps, and further extended just before this handoff). Do not re-derive — treat as locked spec, go straight to implementation investigation/planning.**

### Channel
In-app only (bell icon + list) — no email, no OS-level push in Node 9. Phase 1.

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

## Additional Node 9 design decisions (locked, discussed in Chat 15 right before this handoff)

1. **Bell icon must live in a shared layout/header**, not a single dashboard page — so it stays visible and live-updating no matter which page (Menu, Tables, etc.) the logged-in user is currently on.
2. **On login, unread notifications must load immediately** — bell count should reflect any notifications generated while the user was logged out (e.g. overnight, since Owner logs everyone out nightly for attendance-based login). This is a fetch-on-mount of unread count, not just realtime-going-forward.
3. **Notification click → navigate → fresh data guaranteed.** Clicking a notification navigates to the relevant page; since all pages already do fetch-on-mount (`useEffect` pattern, confirmed throughout Node 11), this naturally guarantees fresh data on arrival — no extra work needed beyond normal navigation.
4. **No login-wall bypass risk** — bell/notifications only exist inside authenticated dashboard routes, which are already auth-protected. A logged-out user cannot reach or see notifications by design; no extra security work needed here.
5. **Explicitly OUT of Node 9 scope: true OS-level push notifications** (phone buzzing/alerting while the user is in a different app entirely, e.g. Instagram). This requires PWA conversion + service worker + push subscription infra — genuinely separate scope, not a small addition. Ayush confirmed he wants this eventually because Order Ready is time-sensitive and email is too slow/passive for it — this becomes **Node 12: Push Notifications** (see below), built after Node 9's `notifications` table/trigger logic exists, since Node 12 reuses that same event/trigger foundation and just adds a push delivery channel on top.

## New Node created: Node 12 — Push Notifications (NOT STARTED, comes after Node 9)

- **Scope:** True OS-level push notifications (phone notification tray, works even when the app/tab isn't open) for the same 5 event types already locked for Node 9 (Order Status changes, Order Cancelled, Reservation Approved/Rejected, New Reservation Request, New Order Placed) — Ayush confirmed all 5, not a subset.
- **Why after Node 9:** Node 12 builds on Node 9's `notifications` table and event-trigger logic — it adds a push delivery channel on top of the same triggers, rather than duplicating event-detection logic.
- **Known requirements (not yet scoped in detail):** PWA setup/manifest, service worker, push subscription per-device (store push tokens), a push service (e.g. Web Push API or Firebase Cloud Messaging), browser/OS permission handling.
- **Not yet discussed:** exact push payload format, whether push should duplicate in-app bell content or be a shorter alert, permission-denied fallback behavior, multi-device handling if a staff member logs in from multiple devices.

## What Node 9 needs next (not yet started)
1. Design `notifications` table schema (columns, RLS policies) — not yet done.
2. Design bell icon UI/component placement — now specifically as a shared layout/header component, not page-specific (see additional design decisions above).
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
