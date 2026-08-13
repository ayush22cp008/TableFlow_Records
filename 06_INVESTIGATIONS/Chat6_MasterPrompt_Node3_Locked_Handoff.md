# TableFlow — Chat #6 — Master Prompt: Node 3 LOCKED

## Node-Map
- Node 1 (Permission Matrix) — ✅ LOCKED
- Node 2b (Schema, RLS, invite codes, cancellation, email) — ✅ LOCKED
- Routing architecture — ✅ LOCKED
- Cook Dashboard (KDS) — ✅ LOCKED
- Waiter Dashboard — ✅ LOCKED (waiter branch fix in AuthForm.tsx still pending, queued next)
- **Node 3 (Manager Dashboard) — ✅ LOCKED (as of Chat 6)**
- Node 4 (notifications) — ⬜ NOT STARTED

## Node 3 — Final Summary

**Bugs fixed this chat cycle:**
1. Bug A — `AuthForm.tsx` missing `manager` redirect branch (3 locations) — fixed, confirmed via manual signup test
2. Bug B — Infinite spinner loop in `manager/page.tsx` (`useCallback` deps issue) — fixed, confirmed via live load test
3. Bug C — "Mark Paid" 400 error — root cause: stale `orders.status` CHECK constraint on live DB (never migrated to include `'billed'`), plus missing `payment_method` column (migration never applied to live DB). Both fixed via manual SQL Editor execution on Supabase Dashboard. Confirmed via live test: order successfully clears from Billing Queue, Owner's Live Orders view confirms `served` count drops to 0.

**Functionality confirmed working end-to-end:**
- Intake Queue (Placed) → Accept → Preparing
- Real-time sync across Manager / Owner / Cook / Customer views
- Billing Queue (Served) → itemized bill display, multiple orders stack correctly
- Payment method dropdown (Cash/Card/UPI)
- Mark Paid → order clears from queue, status → `billed`
- Print PDF → generates valid PDF via `window.print()`

**Known gap (not a bug, deferred):** Manager Dashboard has no Service Charge / extra-charge option. Owner's separate billing page (`/dashboard/billing/[id]`) has an optional 10% Service Charge toggle; Manager's billing card does not. Decision on whether to add this to Manager Dashboard is pending — see next chat.

**Pushed to GitHub:** Yes, confirmed by Ayush this chat.

## Next Up
1. Decision: add 10% Service Charge toggle to Manager Dashboard (parity with Owner) or skip for now
2. Waiter branch fix in `AuthForm.tsx` (same pattern as Bug A, deferred until Manager was fully tested — now unblocked)
