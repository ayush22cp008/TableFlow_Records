Chat #3 | Node 2b | Instruction — Fix Manager Orders UPDATE Policy

Issue found in evidence review (Chat3_Node2b_Evidence_Implementation.md):

Current policy:
CREATE POLICY "orders_update_manager" ON orders FOR UPDATE
USING ( has_role(ARRAY['manager']) AND status IN ('placed', 'served') )
WITH CHECK ( has_role(ARRAY['manager']) AND status IN ('preparing', 'billed', 'cancelled') );

Problem: USING/WITH CHECK with IN (...) on both sides allows any cross-product transition (e.g. placed -> cancelled, served -> preparing), not just the two intended transitions from the Permission Matrix: placed -> preparing, and served/completed -> billed (Completed/Paid).

Fix required:
1. Split into separate paired policies (or use explicit OLD/NEW status pairing) so only these exact transitions are allowed:
   - placed -> preparing
   - served -> billed (or whatever the confirmed enum value for "Completed/Paid" is — see below)
   - If cancellation by Manager is intended at all, confirm which specific source states it's valid from (per Permission Matrix, this wasn't explicitly scoped) rather than allowing it from any state.

2. Confirm exact status enum values currently in the DB/schema (investigation, report first): Is it 'billed', 'completed', 'paid', or something else representing "Completed/Paid" from the Permission Matrix? Report back with the actual enum/CHECK constraint values before finalizing the fix, per evidence rule — do not assume 'billed' is correct.

Investigation and fix as separate steps per engineering discipline rule: first report back the actual status values in use, then apply the corrected policy in a follow-up.
