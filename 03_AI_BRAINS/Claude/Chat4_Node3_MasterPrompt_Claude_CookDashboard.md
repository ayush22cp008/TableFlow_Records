# TableFlow — Chat #4 / Node 3 — Master Prompt (Claude Side): Cook Dashboard

**Status:** Spec locked, implementation NOT started
**Depends on:** Node 3 routing investigation (LOCKED — role-specific routes confirmed via `Chat4_Node3_Investigation_RoutingArchitecture.md`)

---

## Scope

Build the Cook-specific dashboard at a new role-specific route. First of three dashboards (Manager, Cook, Waiter) — Cook chosen first as narrowest scope (single status transition, no money/table access).

**Explicitly OUT of scope for this node:** auto-refresh/polling. Ships with manual refresh only. Auto-refresh will be added uniformly across all three dashboards when Node 4 (Notification System) is built — bundling it now adds complexity without a way to isolate which layer broke if something fails.

## Grounding — Permission Matrix (Cook row, locked in Node 1)

| Resource | Cook access |
|---|---|
| Order items (kitchen ticket: item+qty only, no table) | R |
| Order status: Preparing → Ready | R/W (only transition Cook owns) |
| Order status (all other states) | R (view only) |
| Tables, Bills, Menu, Analytics, Staff | No access |

## Decisions Locked This Session

1. **Route pattern:** role-specific route (e.g. `/dashboard/cook`), confirmed by Antigravity investigation — matches existing `owner`/`customer` redirect pattern in `app/auth/callback/route.ts` and `middleware.ts`, and `UserRole` type already includes `'cook'`.
2. **Order-level granularity:** whole order marked Ready at once — no per-item partial-ready tracking. Matches existing precedent (all other status transitions in the locked lifecycle are whole-order).
3. **Refresh:** manual only for this node. Auto-refresh deferred to Node 4.

## Functional Spec

- **Query:** orders where `status = 'preparing'`, reusing the existing join pattern already proven in `app/dashboard/orders/page.tsx`: `.select('*, order_items(quantity, menu_items(name))')`
- **Ticket card shows:** order short ID, item list as `{quantity}x {name}` — **no price, no table number, no customer info** (per Permission Matrix: kitchen ticket is item+qty only)
- **Action:** single "Mark Ready" button per card, calls the existing locked RLS-backed transition (`cook_prep_to_ready` policy: `USING (status='preparing') WITH CHECK (status='ready')`) — update `orders.status = 'ready'`
- **Manual refresh:** simple refresh button/reload, no polling interval

## Reuse, Don't Rebuild

The Kanban card + status-advance button pattern already exists and works in `app/dashboard/orders/page.tsx` (`NEXT_STATUS[status]` → `advanceStatus(order)`). Cook's dashboard is a **filtered, stripped-down variant** of this existing pattern — reuse the query/join/button mechanics, just scope the query to `preparing` only and strip fields not permitted for Cook (price, table, customer).

## Routing Change Required

- `app/auth/callback/route.ts` and `middleware.ts`: add `cook` branch → redirect to `/dashboard/cook` (currently falls through to `/order` per investigation findings)

## Out of Scope (do not build in this node)

- Auto-refresh/polling (→ Node 4)
- Manager dashboard, Waiter dashboard (separate nodes, spec'd one at a time)
- Cancellation UI for Cook (already exists via `cook_cancel` RLS policy but that flow lives in the existing shared orders board — not being duplicated here unless explicitly requested later)

## Next Action

Write Antigravity-side build instruction (separate prompt per engineering discipline rule — investigation/fix are never mixed with build). After Cook dashboard is built and pushed, Ayush manually tests live per evidence rule before Node 3 continues to Waiter or Manager.
