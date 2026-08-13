# TableFlow — Chat #5 / Node 3 — Instruction (Correction): Manager Dashboard Status Terminology

**To:** Antigravity
**From:** Claude
**Type:** Correction — resolves the RLS conflict flagged in `Chat5_Node3_ManagerDashboard_ImplementationPlan.md`

---

## Resolution

Use **`billed`** as the target status — matches the existing live, locked RLS policy (`manager_served_to_billed` from `20260804000002_node2b_cancellation.sql`). Do **not** write a new migration. Do **not** introduce a `completed` status.

This corrects my original instruction (`Chat5_Node3_Instruction_ManagerDashboard.md`), which incorrectly said `completed` — that was written without checking the actual live schema first. The DB is the source of truth here.

## What to change in your plan

Wherever the original instruction or your implementation plan says `orders.status = 'completed'`, use `orders.status = 'billed'` instead. Specifically:

- Billing queue "Mark Paid" button → `orders.status = 'billed'` (paired atomically with `payment_method` write, same single-update rule as before — no dual-write)
- Any query filtering for the billing queue (`status = 'served'`) is unaffected — that part was already correct
- If the Owner dashboard or any other existing view filters/displays by order status, confirm `billed` already renders correctly there (it should, since this policy is already live) — just a sanity check, not expected to need changes

## Everything else from the original instruction stands unchanged

- `payment_method` schema addition (cash/card/upi) — proceed as planned
- PDF via `window.print()` + `@media print` — approved, good call avoiding a new dependency
- Navbar exposure of Tables/Menu links to `manager` role instead of duplicating pages — approved, good investigation

## Proceed

You're clear to proceed with the full build now — schema change, routing, Manager page (Intake + Billing queues), Navbar update — using `billed` as the terminology throughout.

## Verification Before Reporting Back

- `npm run build` — 0 errors
- Confirm the `served → billed` transition actually succeeds against the live `manager_served_to_billed` policy (not just that the UI compiles)
- Confirm `payment_method` write is paired with the `billed` status update in one call

## Output

Do NOT push to GitHub yet — push is manually triggered by Ayush after review. Report back with files touched + build result. Ayush will test live after Vercel deploy, per evidence rule.
