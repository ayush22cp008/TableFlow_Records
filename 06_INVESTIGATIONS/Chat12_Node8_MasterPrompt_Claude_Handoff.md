# TableFlow — Claude-Side Master Prompt (Chat 12 Handoff)

## Node Map
- ✅ LOCKED: Node 1, 2b, Routing, Cook Dashboard, Manager Dashboard, Waiter Dashboard, Node 4 (double-booking), Reservation Requests panel cleanup
- ✅ LOCKED: Owner Side Staff Management (navbar, Staff Details, force logout, delete) — pushed
- ✅ LOCKED: Deactivate → Full Hard Delete — pushed
- ✅ LOCKED: Invite Codes Cleanup — pushed, verified
- ✅ LOCKED: Deactivation Email + Welcome Email — pushed, verified
- ✅ LOCKED: Invite Code Verification Bugs (password re-activation + Google OAuth bypass) — pushed, verified
- 🔄 ACTIVE: **Node 8 — Customer Dashboard Revamp** (this chat, investigation phase — see below)
- ⬜ NOT STARTED: Notifications (major node, scoping not done — deferred until Node 8 closes)

## Node 8 — Customer Dashboard Revamp — Scope (decided this chat)

Three changes, all in one node, customer-side only:

1. **Reservation nav move**: Currently "Reserve a Table" is a small button inside the Menu page. Move to navbar (Menu | My Orders | Reservation | Sign Out), linking to a dedicated new page (e.g. `/order/reservation`).
2. **Reservation status feature**: New Reservation page shows customer's **current/latest reservation only** (no history list) — table, party size, current status. **No cancel/modify button on customer side** — Ayush believes owner-side already has 30-min no-show auto-release + manual clear, but this is unconfirmed and under investigation.
3. **My Orders table/order label**: Add table number + short order label (matching R1/W1-style convention used on staff dashboards) to `/order/my-orders` for consistency. Ayush believes table is already linked to orders at order-placement time — unconfirmed, under investigation.

## Investigation Prompts Sent (awaiting results — no fix yet)

Both created this chat in `02_Instructions/`, investigation-only, no code changes:

1. `Chat12_Node8_Investigation_ReservationStatusLogic.md` — reservation schema/status enum, whether 30-min auto-release + owner manual-clear actually exists in code (or needs building), how to query a customer's own latest reservation.
2. `Chat12_Node8_Investigation_OrderTableLinking.md` — whether `table_id` already exists on orders and is reliably populated, how table is determined at order time, whether `formatOrderNumber()` can be reused for customer-facing labels.

**Next chat starts with Ayush pasting both investigation results.** Do not propose fixes until both are reviewed — investigation and fix stay separate prompts per standing rule.

## Reference Screenshots (this chat)
- Current Menu page: shows "Reserve a Table" as a small pill button above item filters, plus item cards (mug pulav, sev samosa, vadapav, paneer pulav).
- Current My Orders page: shows only short order ID hash, timestamp, status badge, total — no table number, no R#/W#-style label.

## Standing Reminders Still Active
- Investigation and fix always separate prompts.
- All manual DB/migration changes logged in `04_Logs/` immediately.
- No GitHub push without Ayush's explicit go-ahead.
- Supabase CLI unavailable — all migrations are manual SQL Editor runs.
- Antigravity handles execution/build-check only; Ayush does all manual browser UI verification with screenshots.
