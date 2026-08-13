# TableFlow — Staff Role System — Master Prompt (Claude Side)

Chat #8 | Handoff from Chat #7

**Project:** TableFlow Staff Role System
**Drive bridge root:** `TableFlow_Staff_Role_System` (ID `16Xhn59dqZg2_XVMnqulmLeywO3CfhzyL`)
**GitHub:** ayush22cp008/TableFlow · **Live:** table-flow-nu.vercel.app

## Node-Map

- Node 1 (Permission Matrix) — ✅ LOCKED
- Node 2b (Schema, RLS, invite codes, cancellation, email) — ✅ LOCKED
- Routing architecture — ✅ LOCKED
- Cook Dashboard (KDS) — ✅ LOCKED
- Manager Dashboard (core) — ✅ LOCKED (Chat 6)
- Waiter Dashboard — ✅ LOCKED (Chat 7)
- **Manager Dashboard — 3 deferred bugs — 🔄 ACTIVE (this chat's starting point)**
- Manager/Customer W#/R# label retrofit — ⬜ NOT STARTED (deferred, lower priority than the 3 bugs)
- Node 4 (notifications) — ⬜ NOT STARTED

## Starting Task — Manager Dashboard: 3 Bugs

Full evidence already recorded in `Chat6_Node_Manager_Bugs_PriorityDeferred.md` (03_Investigation_and_Errors or 02_Instructions, search Drive by title). Summary:

1. **Bug 1 — Intake Queue not priority-sorted.** Walk-in (`W`) orders appearing above reservation/priority (`R`) orders. Should match Owner/Cook's existing correct sort (`R` always above `W`).
2. **Bug 2 — Billing Queue sorted by amount, not priority.** Same fix needed as Bug 1, applied to the Billing Queue fetch/sort logic.
3. **Bug 3 — Mark Paid doesn't release table occupancy.** After billing closure, the Tables page still shows the table as occupied/full instead of resetting to Available. Root cause not yet investigated.

**Suggested approach (not yet started, awaiting Ayush's go-ahead):**
- Investigate Bug 1 & 2 together first — likely the same root cause (Intake/Billing queue queries missing the `is_priority` sort clause that Owner/Cook views already have correctly).
- Investigate Bug 3 separately — likely a missing table-release call in the Mark Paid handler.
- Per standing rule: investigation and fix are always separate prompts. Do not mix them.

## Reference — Prior Session Learnings Relevant to This Bug

- Owner/Cook's correct priority-sort pattern (confirmed working, safe to copy):
  ```typescript
  .order('is_priority', { ascending: false })
  .order('created_at', { ascending: true })
  ```
- Manager Dashboard's queue-fetch logic (Intake/Billing) does NOT currently include this — that's the suspected root cause for Bugs 1 & 2.
- Supabase CLI (`npx supabase db push`) does not work on this Windows machine (win32-x64 binary not found). Use manual SQL Editor via Supabase Dashboard for any live-DB schema changes.

## Standing Workflow Reminders

- Claude = architecture/diagnosis/spec-writing. Antigravity = code execution, no manual UI testing. Ayush = manual testing, final decisions.
- All instruction/investigation/result files go in `02_Instructions/` (Claude→Antigravity) — never elsewhere.
- File naming: `Chat8_Node_{Description}.md`. Master prompts saved to Drive, never pasted in chat.
- GitHub push only on Ayush's explicit go-ahead, immediately after each verified checkpoint.
- Evidence rule: screenshots or Ayush's manual confirmation required before any bug is considered fixed.

## Next Action

Begin with investigation-only pass on Bug 1 & 2 (priority sort). Confirm exact fetch/sort code in Manager Dashboard's Intake/Billing queue components, compare against Owner/Cook's working pattern, report findings before proposing a fix.
