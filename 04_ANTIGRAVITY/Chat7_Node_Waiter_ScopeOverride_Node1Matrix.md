# Waiter Scope — Node 1 Permission Matrix OVERRIDE (Chat 7)

**This is a deliberate override of the Node 1 locked Permission Matrix, not a bug fix or a correction of an error. Both decisions were valid at the time they were made — Ayush is revising the scope now based on updated judgement.**

## Original Node 1 decision (locked, Chat 1)

Waiter was scoped as: "floor service — manages tables, reservations. Owns Ready→Served transition only. No billing/money access."
Matrix gave Waiter: Tables (R/W), Reservations (R/W), Menu (R), Orders Ready→Served (R/W).

## New decision (Chat 7) — supersedes Node 1 for Waiter only

**Reasoning (Ayush):**
- Table occupancy already updates automatically elsewhere in the system (e.g. when Owner cancels/closes an order) — Waiter doesn't need manual Tables R/W for this.
- Reservations are handled by Manager (Manager accepts/manages reservation requests) — Waiter doesn't need Reservations R/W.
- Waiter's actual job is narrower than originally scoped: just floor service on the order queue.

**New Waiter scope:**

| Resource | Access |
|---|---|
| Order status: Ready → Served | **R/W** (unchanged — only action Waiter owns) |
| Order status (view) | R (unchanged) |
| Tables | **REMOVED** — no access, no Navbar link |
| Reservations | **REMOVED** — no access, no Navbar link |
| Menu | **REMOVED** — no access, no Navbar link (previously R-only) |
| Billing | — (unchanged, no access) |
| Analytics / Insights / Staff / Settings | — (unchanged, Owner-only) |

**Net effect:** Waiter Dashboard = orders queue only. Nothing else.

## Action required (not yet done)

Current live build (`Navbar.tsx`) shows **Overview, Menu, Tables** links for waiter role (per `Chat7_Node_Waiter_Implementation_Result.md`). This needs to change:
- Remove Menu and Tables links from Navbar for `waiter` role — only Overview (waiter dashboard) should remain.
- No RLS changes needed for Tables/Reservations (Waiter never had working R/W there in practice per this new scope — just remove the UI entry points; confirm no waiter-specific RLS grants exist on `tables`/`reservation_requests` that should also be revoked, as a safety check).

## Node-Map impact

Node 1's Permission Matrix document remains historically accurate for what was decided at that time — do not edit it. This file is the authoritative override for Waiter specifically, going forward. Any future chat referencing Waiter's scope should read this file, not the original Node 1 matrix, for the waiter row.
