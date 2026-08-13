# TableFlow — Chat #6 / Node 3 — Instruction (Investigation): Manager "Mark Paid" 400 Error

**To:** Antigravity
**From:** Claude
**Type:** Investigation ONLY — do not modify any files, do not apply any fix

---

## Evidence (confirmed via browser DevTools Network tab)

Manual test on Manager Dashboard: clicking "Mark Paid" on a served order (`orders?id=eq.b8c25242-e11a-4c32-a5d1...`) returns:

- **Status: 400 Bad Request**
- The two subsequent `fetchOrders` re-fetch calls both return 200 OK and correctly reflect current DB state
- Confirmed behaviorally: Intake Queue updates correctly on refresh (proves fetch/display pipeline is healthy), but the order stays in Billing Queue after Mark Paid — meaning the actual database update never succeeds, this is not a UI/refresh bug

## What to investigate

1. **Find the exact error message.** Locate the `handleMarkPaid` (or equivalently named) function in `app/dashboard/manager/page.tsx`. Trace exactly what payload/query it sends to Supabase when "Mark Paid" is clicked (which columns, which values, especially the `status` field value it's trying to set).

2. **Check the `orders` table schema** — specifically any CHECK constraint on the `status` column. Find the migration file(s) or schema definition that defines allowed values for `orders.status`. Confirm whether `'billed'` is actually included in that constraint's allowed value list, alongside the other known statuses (`placed`, `preparing`, `ready`, `served`, etc.)

3. **Check for any other 400 causes** if the constraint looks fine: mismatched column name in the update payload, wrong data type, a NOT NULL column not being populated (e.g. `payment_method` required but not sent), or any other reason Postgres/PostgREST would reject this specific write with 400.

## Explicitly out of scope for this instruction

- Do NOT modify any files.
- Do NOT apply any fix, even if the cause seems obvious.
- Do NOT touch other unrelated code paths.

## Output — Result file

Write findings to `03_Investigation_and_Errors/Chat6_Node3_Investigation_ManagerMarkPaid400_Result.md` in the Drive bridge folder, containing:
- The exact code snippet of the update call (`handleMarkPaid` or equivalent)
- The exact current CHECK constraint definition (or full schema block) for `orders.status`
- Your conclusion: single root cause identified, or multiple candidates if genuinely ambiguous
- Do NOT propose or write the fix — root cause identification only
