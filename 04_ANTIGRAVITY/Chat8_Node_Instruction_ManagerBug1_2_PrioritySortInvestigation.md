# TableFlow — Chat #8 — Instruction (Investigation Only)

**To:** Antigravity
**From:** Claude
**Type:** Investigation — NO code changes, NO fixes in this prompt

---

## Task

Investigate Manager Dashboard Bugs 1 & 2 (priority sort). Do not fix yet — report findings only.

**Bug 1:** Intake Queue not priority-sorted — Walk-in (`W`) orders appearing above reservation/priority (`R`) orders. Should match Owner/Cook's existing correct sort (`R` always above `W`).

**Bug 2:** Billing Queue sorted by amount, not priority. Same fix needed as Bug 1, applied to Billing Queue fetch/sort logic.

## Known-good reference pattern (Owner/Cook — confirmed working, do not touch)

```typescript
.order('is_priority', { ascending: false })
.order('created_at', { ascending: true })
```

## Files to Investigate (read-only — do not edit)

1. Manager Dashboard's Intake Queue fetch/sort code (likely `app/dashboard/manager/page.tsx` or a related query/component) — confirm exact current `.order()` clause(s) used, or absence of one
2. Manager Dashboard's Billing Queue fetch/sort code — same check
3. Compare both against the Owner/Cook working pattern above — confirm whether `is_priority` sort is missing entirely, present but wrong order, or something else
4. Note the exact file path + line numbers for both queue queries

## What to Report Back

- Exact current sort/order clause for Intake Queue (or confirm it's missing)
- Exact current sort/order clause for Billing Queue (or confirm it's missing)
- Confirm whether root cause is the same for both (missing `is_priority` sort) or different
- Any other differences vs the Owner/Cook pattern worth flagging

Do **not** write or apply any fix in this pass — investigation and recommendation only. The fix instruction will be a separate prompt once this is reviewed.

## Output

Save findings to: `03_Investigation_and_Errors/Chat8_Node_Investigation_ManagerBug1_2_PrioritySort.md`

Report back to Claude with the file path once done.
