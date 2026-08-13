# TableFlow — Master Prompt (Claude Side) — Chat 16 Handoff (v3 — supersedes v2)

**This file supersedes Chat16_Node9_MasterPrompt_Claude_Handoff_v2.md. v2 was missing: Node 9 investigation results, role-cardinality confirmation, the new Node 13 (Cook/Waiter Claiming System), and the revised node execution order. Use this v3 file for the next chat.**

## Project
TableFlow — restaurant management system, role-based dashboards (Owner, Manager, Waiter, Cook, Customer). Next.js 14, Supabase, Vercel, Resend.

## Role cardinality (confirmed, foundational — applies to all future notification/assignment design)
- **Owner** — single person
- **Manager** — single person
- **Cook** — many (multiple cooks per restaurant)
- **Waiter** — many (multiple waiters per restaurant)
- **Customer** — many

This is why Cook/Waiter need a claiming/assignment mechanism (Node 13) while Owner/Manager notifications can always target "the one person" directly.

## Node Map

- ✅ LOCKED + PUSHED — Node 1–8: Cook/Waiter/Manager Dashboards, staff onboarding, deactivation/welcome emails, Customer Dashboard Revamp
- ✅ LOCKED + PUSHED — Node 10: Owner/Staff Role Overlap Cleanup (Manager = sole operational authority; Owner = read-only + Menu Management + Bulk Emergency Stop, Owner-only)
- ✅ LOCKED + PUSHED — Node 11: Realtime Coverage (Chat 15)
- 🔄 ACTIVE — **Node 9: Notifications System (In-App, role-based)** — investigation complete, ready for schema/implementation
- ⬜ NOT STARTED — **Node 13: Cook/Waiter Order Claiming System** (depends on Node 9)
- ⬜ NOT STARTED — **Node 12: Push Notifications** (depends on Node 13 — order revised, see below)

### Node execution order — revised: 9 → 13 → 12
Originally Node 12 was planned right after Node 9. Revised because Node 12 (push) reuses Node 9's event-trigger logic — if Node 12 were built before Node 13, push logic would need rework once Node 13 changes notifications from role-broadcast to specific-person targeting. Doing Node 13 before Node 12 means push notifications get built once, against final/stable trigger logic.

## Node 9 — Investigation Complete (this chat)

Two investigation passes done (both read-only, no code changes). Full reports in `03_Investigation_and_Errors/`:
- `Chat16_Node9_Investigation_NotificationInsertionPoints.md`
- `Chat16_Node9_FollowUpInvestigation.md`

### Confirmed insertion points (VERIFIED, from code)
| Event | File | Function |
|---|---|---|
| Order Placed→Preparing | `app/dashboard/manager/page.tsx` | `markPreparing` |
| Order Preparing→Ready | `app/dashboard/cook/page.tsx` | `markReady` |
| Order Ready→Served | `app/dashboard/waiter/page.tsx` | `markServed` |
| Order Cancelled (Bulk Emergency Stop) | `app/dashboard/orders/page.tsx` | `submitBulkCancel` |
| Reservation Approve/Reject | `app/dashboard/tables/page.tsx` | `approveRequest` / `rejectRequest` |
| New Reservation Request | `app/order/reservation/page.tsx` | `submitRequest` |
| New Order Placed | `app/order/cart/page.tsx` | checkout insert (`.from('orders').insert(...)`) |

Shared header for bell icon placement: `components/Navbar.tsx` (confirmed as the single shared nav across all roles).

### Known gap — Bulk Cancel status-snapshot (VERIFIED missing, needs handling during Node 9 implementation)
`submitBulkCancel` currently does NOT snapshot each order's status before calling `cancel_active_orders` RPC:
- **"Select Specific" mode:** status IS available in client's `orders` state array already — just needs to be read and used, no new fetch required.
- **"Cancel ALL" mode:** client doesn't know which exact orders will be affected — needs a new pre-RPC fetch (e.g. `SELECT id, status FROM orders WHERE status IN ('placed','preparing','ready')`) before calling the RPC, to support locked targeting design (Placed→Manager, Preparing→Cook, Ready→Waiter, Customer→always).

### Known inconsistency — Auth pattern (noted, decision made)
- `useAuth()` hook used in: `app/order/cart/page.tsx`, `app/order/reservation/page.tsx`, `components/Navbar.tsx`
- Direct `supabase.auth.getUser()` used in: `app/dashboard/orders/page.tsx`
- **Decision: no standardization.** New Node 9 notification-trigger code follows whichever pattern already exists in each file. Not worth a cleanup pass — out of scope for Node 9.

### Confirmed complete realtime channel list (VERIFIED — corrects earlier partial list)
`orders_board`, `manager_orders_realtime`, `cook_orders_realtime`, `waiter_orders_realtime`, `owner_analytics_realtime`, `staff_management_realtime`, `owner_menu_realtime`, `tables_realtime`, `my_orders_realtime`, `menu_realtime`, `reservation_status_realtime`. No discrepancy vs Node 11 doc — earlier "mismatch" was just an incomplete first-pass scan.

### Node 9 — Locked Scope (role-based only, no claiming)

**Channel:** In-app only (bell icon + list). No email, no OS push (that's Node 12).

**Phase 1 events (role-based broadcast — explicitly NOT per-person assignment):**
1. Order Status changes: Preparing → **all Cooks**; Ready → **all Waiters** (Ready = highest priority, real-time ping)
2. Order Cancelled (Bulk Emergency Stop) → staff notified by status-based targeting table below + Customer always
3. Reservation Approved/Rejected → Customer (logged-in only)
4. New Reservation Request → Manager only (not Owner)
5. New Order Placed → Manager

**Order Cancelled — status-based staff targeting (locked):**
| Order status at cancellation | Staff notified |
|---|---|
| Placed | Manager |
| Preparing | Cook (all — role-based, no claiming yet) |
| Ready | Waiter (all — role-based, no claiming yet) |
| (any status) | Customer — always |

**Explicitly OUT of Node 9 scope (deferred to Node 13):**
- Claiming/assignment of orders to a specific Cook or Waiter
- "Who accepted this order" tracking or Manager-facing visibility of it
- Any notification targeted at a single specific Cook/Waiter rather than the whole role

**Deferred to Phase 2 (unrelated to Node 13, still just not-yet-scoped):** Order Billed/Paid, Waitlist Seated/Cancelled, Staff Role Changed/Force Logout

**Not in scope:** Table toggle, new table, menu changes, invite used

**Design decisions (locked, from Chat 15/16):**
1. Bell icon lives in shared layout (`components/Navbar.tsx`), not a single dashboard page.
2. On login, unread notifications must load immediately (fetch-on-mount of unread count, not just realtime-going-forward) — covers notifications generated while logged out.
3. Notification click → navigate → fresh data guaranteed (existing fetch-on-mount pattern on all pages already handles this, no extra work).
4. No login-wall bypass risk — notifications only exist inside already-auth-protected routes.

### What Node 9 needs next (not yet started)
1. Design `notifications` table schema (columns, RLS policies).
2. Build bell icon UI component in `components/Navbar.tsx`.
3. Implement triggers at the 7 confirmed insertion points above.
4. Implement Bulk Cancel pre-RPC status-snapshot fix (see Known Gap above) as part of this node — needed for correct targeting.
5. **Remember:** new `notifications` table needs Realtime replication toggle checked/enabled manually in Supabase Dashboard (per Node 11 lesson) before assuming realtime will work.

## Node 13 — Cook/Waiter Order Claiming System (NOT STARTED, comes after Node 9)

**Scope (locked, discussed Chat 16):**
- When an order is Placed, all Cooks get notified (role-based, from Node 9). First Cook to click "claim/accept" gets that order assigned to them exclusively — other Cooks can no longer claim it.
- **Strict one-order-at-a-time rule:** a Cook cannot claim a new order until their current claimed order is marked Ready (i.e., their active order is done).
- Same claiming logic applies to Waiters (claiming a Ready order to serve).
- **Manager Dashboard — 2 new features:**
  1. Visibility into which Cook has claimed which order (Cook name + email).
  2. Same visibility for Waiter/order-serving assignment.
- When an order is claimed/accepted by a Cook or Waiter, a notification is sent **to the Manager** ("Order #X accepted by [name]") — this notification itself is part of Node 13, not Node 9, since the claim action doesn't exist until Node 13 is built.

**Not yet discussed (to be scoped when Node 13 starts):**
- Exact schema changes needed (likely `assigned_cook_id` / `assigned_waiter_id` columns on `orders`, or a separate assignment table).
- Race-condition handling if two Cooks click claim at the same moment (needs DB-level locking/constraint, not just frontend check).
- What happens if a claimed order needs to be reassigned (Cook goes off-shift mid-order, etc.) — not discussed yet.

## Node 12 — Push Notifications (NOT STARTED, now comes after Node 13)

- **Scope:** True OS-level push notifications (phone notification tray, works even when app/tab isn't open) for the same event types locked for Node 9 — but by the time this is built, targeting will be Node 13's per-person claiming logic, not Node 9's role-broadcast.
- **Why after Node 13 (revised):** avoids building push once against role-based triggers and then reworking it once Node 13 changes targeting to per-person.
- **Known requirements (not yet scoped in detail):** PWA setup/manifest, service worker, push subscription per-device (store push tokens), a push service (Web Push API or Firebase Cloud Messaging), browser/OS permission handling.
- **Not yet discussed:** exact push payload format, permission-denied fallback, multi-device handling.

## Standing rules (all still active)
- Investigation and fix always in separate prompts.
- No GitHub push without Ayush's explicit approval — per-checkpoint, not batched.
- All DB migrations run manually via Supabase SQL Editor (CLI unavailable).
- Antigravity: code execution + build/compile check only, no browser UI testing.
- Ayush: manual browser verification, screenshot evidence (or verbal "maine check kiya hai" when screenshot isn't feasible).
- Instruction files → `02_Instructions/` only (no permission needed). Master prompts/specs → ask permission before creating every time.
- File naming: `Chat{N}_Node{M}_{Type}_{ShortDescription}.ext`.
- Full local path always output in copy-paste code block.
- Always check Supabase Realtime replication toggle status before assuming a table will support realtime — don't assume, verify (lesson from Node 11).

## Drive folder IDs (confirmed)
- `02_Instructions/` = `13NcntSWMoqGG105X-wp8Mfu8KpUlASd8`
- `01_Master_Prompts/Claude_Side/` = `1iWwBytbROljqLIE0ZkTEZD0L9mzXmTFQ`
