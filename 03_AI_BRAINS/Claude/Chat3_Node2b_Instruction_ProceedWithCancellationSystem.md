Chat #3 | Node 2b | Instruction — Proceed with Cancellation System Implementation

Approved: proceed with the full plan in Chat3_Node2b_CancellationSystem_Implementation_Plan.md as written.

Both review questions approved:
1. RPC approach for Bulk Emergency Stop (cancel_active_orders) — approved.
2. Paired RLS policies (separate CREATE POLICY per transition, 7 total) — approved.

Proceed with:
- Schema migration (cancellation_reason, cancellation_category columns)
- Drop old orders_update_* policies, create the explicit paired transition policies as scoped
- cancel_active_orders RPC (owner-only, validates role, atomic, zeroes occupied_seats on affected tables)
- Frontend: single-cancel modal (mandatory reason, all 4 roles) + Owner-only Bulk Emergency Stop UI (select-specific + cancel-all modes, mandatory category dropdown, optional detail text)

Report back with build/compile evidence and the exact final RLS policy SQL for verification before this gets merged, per evidence rule.
