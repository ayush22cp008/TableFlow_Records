# TableFlow — Chat #8 — Instruction (Investigation Only)

**To:** Antigravity
**From:** Claude
**Type:** Investigation — NO code changes, NO fixes in this prompt

---

## Task

Manager Dashboard's Intake Queue and Billing Queue currently display raw order IDs (e.g. "Order #c7720b", "Order #131aba") instead of the clean sequential `W#`/`R#` labels already used elsewhere in the app (Owner, Cook, Waiter dashboards — confirmed working, e.g. `R3`, `R4`, `W11`, `W12`).

Investigate how the existing `W#`/`R#` label system works, and why Manager Dashboard doesn't use it.

## What to Investigate

1. **Find the label generation source.** Search the codebase for where `W#`/`R#` labels are generated/derived for Owner/Cook/Waiter views (e.g. `app/dashboard/orders/page.tsx`, `app/dashboard/cook/page.tsx`, `app/dashboard/waiter/page.tsx`, or a shared util/helper). Report exact file + function/logic.

2. **Confirm whether it's computed or stored.** Is `W#`/`R#` a value stored in the DB (a column), or computed on the fly client-side (e.g. counting/indexing orders by `is_priority` and creation order)? This matters for whether Manager Dashboard can just reuse the same logic or needs a shared source.

3. **Daily reset behavior — check specifically.** Ayush has observed that `R1`/`W1` numbering resets to start fresh each day. Confirm:
   - Is this reset intentional/by design, or a side effect of how the numbering is computed (e.g. filtered by "today's orders only")?
   - What exact logic drives the reset (date filter, `created_at` comparison, etc.)?
   - Report exact file + line reference.

4. **Confirm why Manager Dashboard doesn't have this.** Check `app/dashboard/manager/page.tsx` — does it fetch/display raw order IDs directly instead of calling the shared label logic? Or is the label logic entirely duplicated per-dashboard (not shared), meaning Manager's queries never had it added?

## What to Report Back

- Exact file/logic where `W#`/`R#` is generated (Owner/Cook/Waiter side)
- Whether it's DB-stored or computed client-side, and the exact reset logic (daily reset confirmed or not, and how)
- Exact reason Manager Dashboard doesn't show it (missing logic call, different query shape, etc.)
- Recommended reuse approach for retrofit (calling shared logic vs duplicating it) — but do NOT write the actual fix code yet

Do **not** apply any fix in this pass — investigation and recommendation only. The fix instruction will be a separate prompt once this is reviewed.

## Output

Save findings to: `03_Investigation_and_Errors/Chat8_Node_Investigation_ManagerWR_LabelRetrofit.md`

Report back to Claude with the file path once done.
