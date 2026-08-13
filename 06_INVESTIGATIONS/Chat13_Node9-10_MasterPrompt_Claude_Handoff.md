# TableFlow — Claude-Side Master Prompt — Chat 13 Handoff

## Node Map

- Node 1 → 8 (Reservation double-booking, Owner Staff Mgmt, Waiter/Manager/Cook Dashboards, Deactivate→Hard Delete, Invite Cleanup, Deactivation/Welcome Email, Invite Verification Bugs, Customer Dashboard Revamp): ✅ LOCKED — do not re-touch
- **Node 9 — Notifications System**: 🔄 PAUSED (blocked on Node 10 — targeting logic depends on finalized role permissions)
- **Node 10 — Owner/Staff Role Overlap Cleanup**: 🔄 ACTIVE — investigation sent to Antigravity, result pending (to be pasted in next chat)

## Why Node 10 exists (discovered mid-Node-9-scoping)
While scoping Node 9 notification targeting, found that Owner's dashboard duplicates operational actions already owned by Manager/Cook/Waiter on their dedicated dashboards. This creates access overlap and race-condition risk (e.g. Owner and Manager could both act on the same reservation simultaneously). Decision: Owner becomes a **read-only monitoring role** for day-to-day ops, keeping only true emergency override (Bulk Emergency Stop). Node 10 must complete before Node 9 resumes, since notification targeting (who gets notified for what) depends on finalized permissions.

## Node 10 — Locked Scope (Owner/Staff Role Overlap Cleanup)

| Area | Current Owner access | Decision |
|---|---|---|
| Live Orders status buttons | `→ preparing`, `→ ready`, `→ served`, `Bill`, plain `Cancel` | Remove all — only **Bulk Emergency Stop** remains (reason-mandatory, untouched) |
| Bill page/route | Full access — itemized bill, "Generate Bill & Mark as Billed" | Remove/restrict — this is Manager's job (Manager Dashboard already has "Mark Paid") |
| Reservation approve/reject | Full access, same panel as Manager | **Manager = final authority.** Owner becomes **read-only** |
| Tables page | Full access — Reserve/Clear/toggle buttons | **Read-only** for Owner — status visible, but Reserve/Clear buttons removed |

**Investigation instruction already sent** (`02_Instructions/Chat13_Node10_Instruction_OwnerPermissionCleanupInvestigation.md`), asking Antigravity to report for all 4 areas above:
- Exact file/component paths and RPC/mutation names
- Whether Owner/Manager share the same component or separate ones
- Current RLS policy status — verdict needed per area: **"UI-only restriction sufficient"** vs **"DB/RLS-level restriction also required"**
- Any other Owner-side duplicate action not yet listed (flag only, don't act)

**Next chat starts with:** paste Antigravity's investigation result (`03_Investigation_and_Errors/Chat13_Node10_Investigation_OwnerPermissionCleanupInvestigation.md`) → Claude reviews findings → writes a **separate fix instruction** (investigation and fix stay separate prompts, per project rules) → user reviews → Antigravity executes → user manually tests in browser → confirm before GitHub push.

## Node 9 — Notification Targeting (locked once Node 10 completes; do not re-derive)

Full-system investigation already done (`03_Investigation_and_Errors/Chat13_Node9_Investigation_RoleWiseChangeInventory.md`) — confirmed almost the entire system is silent (only 2 email cases exist: Invite Generated, Staff Deactivated). 4 realtime channels exist (`orders_board`, `my_orders_realtime`, `menu_realtime`, `waitlist_realtime`) but these only sync UI state, not true notifications.

### Phase 1 Scope (Top Priority — LOCKED)
1. **Order Status changes** — Preparing → Ready → Served (Ready = highest priority, real-time ping)
2. **Order Cancelled** — via Bulk Emergency Stop only (plain Cancel button being removed in Node 10)
3. **Reservation Approved/Rejected** → Customer
4. **New Reservation Request** → **Manager only** (not Owner — Owner is read-only per Node 10)
5. **New Order Placed** → Manager (existing forward mechanism to Cook/Waiter — needs investigation, do not assume; Manager Dashboard "Accept (Send to Kitchen)" already exists, confirm this is the same flow)

### Deferred to Phase 2
Order Billed/Paid, Waitlist Seated/Cancelled, Staff Role Changed/Force Logout

### Not in scope
Table toggle, new table, menu changes, invite used

### Answered design questions (locked)
1. **Cancel reason**: Mandatory — already implemented in Bulk Emergency Stop modal (Category dropdown + Additional Details text field)
2. **Channel**: In-app only (bell icon + list) — no email, no push, for Phase 1
3. **Bulk cancel notification scope**:
   - "Cancel ALL Active Orders" mode → all active-order customers get the same reason
   - "Select Specific Orders" mode → only selected customers get the same reason
   - (Scoping/selection logic already exists in the Bulk Emergency Stop feature — Node 9 only needs to hook notification-send onto the existing confirm action, not build new selection logic)
4. **Audience**: Customer + Waiter (Waiter notified specifically on Order Ready, since Waiter serves it)

### Order Cancelled — status-based staff targeting (locked)
| Order status at time of cancellation | Staff notified |
|---|---|
| Order Placed | Manager |
| Preparing | Cook |
| Ready | Waiter |
| (any status) | Customer — always |

## General reminders (from general-project-setup skill — apply throughout)
- Investigation and fix are always separate prompts — never mix.
- No GitHub push without explicit user go-ahead.
- All Supabase migrations run manually via SQL Editor (CLI unavailable on Windows) — log manual DB changes in `04_Logs/`.
- Instruction files for Antigravity go directly in `02_Instructions/` (no permission needed for that folder only). All other files (master prompts, specs) require permission each time.
- User does all manual browser UI testing — Claude/Antigravity do not claim UI verification.
