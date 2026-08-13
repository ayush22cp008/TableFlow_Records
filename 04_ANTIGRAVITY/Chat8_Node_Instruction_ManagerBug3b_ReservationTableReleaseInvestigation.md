# TableFlow — Chat #8 — Instruction (Investigation Only)

**To:** Antigravity
**From:** Claude
**Type:** Investigation — NO code changes, NO fixes in this prompt

---

## Task

Manual testing after the `mark_order_paid` RPC fix (Bug 3) shows a split result:

- **Walk-in (W#) orders:** Manager Mark Paid works correctly — table flips to Available (green) once fully paid.
- **Reservation (R#) orders:** Manager Mark Paid does NOT release the table — table stays purple (reserved-looking state), even after payment.
- **Owner side comparison:** On the Owner dashboard, using "Generate Bill" + "Mark as Billed" on the SAME kind of reservation order correctly flips the table to Available (green) immediately.

So there's a working reference (Owner's Generate Bill/Mark as Billed flow) that handles reservations correctly, but the new `mark_order_paid` RPC (used by Manager) does not.

## What to Investigate

1. **Find Owner's working flow.** Locate the Owner dashboard's "Generate Bill" and "Mark as Billed" handlers (likely in `app/dashboard/orders/page.tsx` or a billing-specific component). Report exact file + function + the exact DB/RPC calls they make.

2. **Compare table-release logic for reservations specifically.** Does Owner's flow clear `reserved_from` (or any other reservation-specific column) in addition to decrementing `occupied_seats`? Report the exact SQL/logic.

3. **Check the new `mark_order_paid` RPC** (`supabase/migrations/20260809000001_mark_order_paid_rpc.sql`). Confirm: does it currently touch `reserved_from` at all? (Per the original fix instruction, it was explicitly told NOT to touch `reserved_from` and to flag this for review — confirm that's exactly what happened.)

4. **Confirm root cause.** Is the purple/reserved visual state driven by `reserved_from` being non-null (regardless of `occupied_seats`)? If so, that would explain why Walk-in orders (no `reserved_from` set) release fine, but Reservation orders (with `reserved_from` set) don't — because `mark_order_paid` never clears it.

5. **Check whether Owner's flow only clears `reserved_from` unconditionally, or also accounts for multiple orders per table** (same multi-order concern as the original Bug 3 investigation) — we need to know if blindly copying Owner's approach into `mark_order_paid` would reintroduce the multi-order bug for reservation tables.

## What to Report Back

- Exact Owner-side working code (file + lines) for Generate Bill / Mark as Billed
- Exact confirmation of what `reserved_from` handling Owner's flow does
- Exact confirmation of what `mark_order_paid` currently does (or doesn't do) with `reserved_from`
- Root cause conclusion
- Recommended fix approach — but do NOT write the actual fix code yet, and flag if multi-order-per-table needs special handling for the reservation case

Do **not** apply any fix in this pass — investigation and recommendation only.

## Output

Save findings to: `03_Investigation_and_Errors/Chat8_Node_Investigation_ManagerBug3b_ReservationTableRelease.md`

Report back to Claude with the file path once done.
