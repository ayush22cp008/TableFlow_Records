# TableFlow — Chat #7 — Waiter Dashboard: LOCKED

## Node-Map
- Node 1 (Permission Matrix) — ✅ LOCKED
- Node 2b (Schema, RLS, invite codes, cancellation, email) — ✅ LOCKED
- Routing architecture — ✅ LOCKED
- Cook Dashboard (KDS) — ✅ LOCKED
- Manager Dashboard — ✅ LOCKED (Chat 6)
- **Waiter Dashboard — ✅ LOCKED (as of Chat 7)**
- Manager Dashboard — 3 bugs (priority sort x2, Mark Paid table-release) — ⬜ NOT STARTED (deferred, see `Chat6_Node_Manager_Bugs_PriorityDeferred.md`)
- Manager/Customer W#/R# label retrofit — ⬜ NOT STARTED (deferred)
- Node 4 (notifications) — ⬜ NOT STARTED

## Waiter Dashboard — Final Scope (overrides Node 1 matrix for Waiter only)

Per `Chat7_Node_Waiter_ScopeOverride_Node1Matrix.md` — deliberate scope reduction, not a bug:

- **Orders queue only.** Ready → Served transition (Waiter's sole action).
- Order cards show `W#`/`R#` + Table number (e.g. `W3 · Table 2`), reusing existing `daily_order_counters`.
- Priority sort: R (reservation) orders always above W (walk-in), same logic as Owner/Cook.
- **No Tables access, no Reservations access, no Menu access** — all removed from original Node 1 scope. Table occupancy is handled automatically elsewhere (order cancel/close); Reservations handled by Manager.
- Navbar: only logo/title + Sign Out. No nav links (waiter dashboard is the only page, reached directly post-login).

## Build & Bug History This Node (chronological)

1. Initial build (`Chat7_Node_Waiter_Implementation_Plan_v2.md`): routing (`AuthForm.tsx`), Waiter Dashboard page, order labeling + priority sort (reused Cook's logic), Mark Served button, Navbar links (Overview/Menu/Tables — later reduced).
2. **Live tested:** order queue labeling, priority sort, Mark Served transition, real-time sync to Owner's Live Orders — all confirmed working via screenshots.
3. **Bug found:** Google OAuth signup with waiter invite code landed on Customer `/order` page instead of `/dashboard/waiter` (manual signup was fine). Same shape as Manager's Chat 5 bug — `select-role/page.tsx` and `callback/route.ts` both missing `waiter` branch.
4. **Fix applied** to both files (added `waiter` branch matching existing `manager`/`cook` pattern). Build passed, **live tested and confirmed by Ayush** — both manual and Google OAuth paths now correctly route to `/dashboard/waiter`.
5. **Scope decision (Chat 7):** Ayush deliberately reduced Waiter scope from Node 1's original matrix (Tables R/W, Reservations R/W, Menu R) down to orders-queue-only, reasoning that table occupancy is automatic and reservations are Manager's job. Documented in `Chat7_Node_Waiter_ScopeOverride_Node1Matrix.md`.
6. **Navbar fix applied** (v2 instruction — removed Overview/Menu/Tables entirely, not just Menu/Tables). Build passed, **live tested and confirmed by Ayush** — Navbar now shows only logo/title + Sign Out.

**Pushed to GitHub:** Yes (routing fix and Navbar fix both pushed and live-tested on Vercel).

## Next Up

Pick up Manager Dashboard's 3 deferred bugs (priority sort x2, Mark Paid table-release) as the next node — see `Chat6_Node_Manager_Bugs_PriorityDeferred.md` for full evidence and proposed investigation approach.
