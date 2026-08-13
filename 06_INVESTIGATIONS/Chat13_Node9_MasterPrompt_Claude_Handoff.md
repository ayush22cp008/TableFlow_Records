# TableFlow — Claude-Side Master Prompt (Chat 13 Handoff)

## Node Map
- ✅ LOCKED: Node 1, 2b, Routing, Cook Dashboard, Manager Dashboard, Waiter Dashboard, Node 4 (double-booking), Reservation Requests panel cleanup
- ✅ LOCKED: Owner Side Staff Management — pushed
- ✅ LOCKED: Deactivate → Full Hard Delete — pushed
- ✅ LOCKED: Invite Codes Cleanup — pushed, verified
- ✅ LOCKED: Deactivation Email + Welcome Email — pushed, verified
- ✅ LOCKED: Invite Code Verification Bugs — pushed, verified
- ✅ **LOCKED: Node 8 — Customer Dashboard Revamp — pushed, fully manually verified** (reservation nav move, customer_id-linked reservation status page, my-orders table number + O#/W# label, old anonymous `/reserve` + `/reserve/status` routes deleted). End-to-end lifecycle tested: pending → approved → entry code → seated, owner-side sync confirmed working.
- 🔄 ACTIVE: **Node 9 — Notifications System** (this chat — scoping phase only, no investigation/fix started yet)
- ⬜ NOT STARTED: Cancellation email (folded into Node 9 scope, not a separate node)

## Node 9 — Notifications — Scope (as discussed, NOT yet fully confirmed)

### Confirmed requirements
1. **Status change notifications** — customer notified when their order moves through: Order Placed → Preparing → Ready → Served (and Billed).
2. **Cancellation notifications** — customer notified if their order is cancelled.
3. **Cancellation permissions (already implemented via RLS, confirmed by Ayush):**
   - Only **Owner** can cancel an order — no staff role (waiter/cook/manager) has cancel permission.
   - Cancel allowed at any status **except "Served"** (Order Placed / Preparing / Ready are cancellable; Served/Billed are not).
   - Owner can cancel **multiple tables/orders at once** (bulk, e.g. "Bulk Emergency Stop" button seen in Orders dashboard) or a **specific single order**.
4. **Notification detail page** — clicking a notification opens a separate page showing: status at time of cancellation, **cancellation reason** (text, written by owner), and confirmation it was cancelled by the owner.
5. **New suggested additions (Ayush approved, add to scope):**
   - **Reservation approved/rejected notification** — currently customer must manually revisit `/order/reservation` to check; should notify proactively when owner approves/rejects.
   - **"Order Ready" alert** — highest-value moment for customer (food is coming), should be a real-time ping, not just a status label change on a page they have to check.

### Open questions — Ayush will answer at start of next chat (do NOT proceed with investigation/fix until answered)
1. **Reason capture UI** — Current "Cancel" button (Orders dashboard) has no reason field. Does a modal/popup need to be added for the owner to type a mandatory/optional reason when cancelling?
2. **Notification channel** — In-app only (e.g. bell icon + list in customer navbar), email only, or both?
3. **Bulk cancel notification behavior** — If owner uses "Bulk Emergency Stop" and multiple orders cancel at once, does each customer get an individual notification, or is there a group/batch behavior to consider?
4. **Audience scope** — Is this customer-facing only, or should staff also get notifications (e.g. waiter notified when an order becomes "Ready" so they know to serve it)?

## Standing Reminders Still Active
- Investigation and fix always separate prompts.
- All manual DB/migration changes logged in `04_Logs/` immediately.
- No GitHub push without Ayush's explicit go-ahead.
- Supabase CLI unavailable — all migrations are manual SQL Editor runs.
- Antigravity handles execution/build-check only; Ayush does all manual browser UI verification with screenshots.
- Node 9 is a "major node" per earlier note (Notifications) — expect full scoping investigation before any fix work starts, given the open questions above.

## Next chat starts with
Ayush answering the 4 open questions above. Only after that: create Node 9 investigation instruction(s) in `02_Instructions/` (schema needs — likely a new `notifications` table, RLS policies, trigger points across existing status-change code paths) before any fix work.
