Chat #3 | Node 2b | Instruction — Fix 3 Bugs (independent, safe to fix together)

Based on investigation (Chat3_Node2b_Investigation_SignupAndBulkStopBugs.md). All three are independent — no shared logic, safe to fix in one pass.

## Fix 1: Signup page missing role selection
File: components/AuthForm.tsx (lines ~245-263), app/auth/select-role/page.tsx
- The staff role panel already exists at app/auth/select-role/page.tsx but is disconnected.
- Either: (a) link/route the main signup flow to this page when a non-customer/owner path is chosen, or (b) merge the staff role selector (role dropdown + invite code field) directly into AuthForm.tsx's existing role array, replacing the hardcoded ['customer', 'owner'].
- Decide whichever requires the smaller, cleaner diff given the current AuthForm structure. Report which approach was taken and why.

## Fix 2: Bulk cancel silently fails
File: app/dashboard/orders/page.tsx (submitBulkCancel, lines ~108-124)
- Wrap the RPC call in try/catch (or destructure { error } from the response).
- On error: show a visible error message to the user (toast/inline), do NOT close the modal or refetch orders.
- On success: proceed with existing close + refetch behavior.

## Fix 3: Checkbox selection blocked by modal overlay
File: app/dashboard/orders/page.tsx (bulk modal backdrop, `fixed inset-0 z-50 ... bg-black/80`)
- Checkboxes on order cards already exist (lines ~180-187) but overlay blocks pointer events.
- Redesign so "Select Specific Orders" mode doesn't use a full-screen blocking backdrop — options: modal becomes a non-blocking side panel/drawer while cards remain clickable, or move the modal to only appear AFTER selection is made (select on the board first, then confirm in a modal). Pick whichever fits the existing UI pattern with the smallest change. Report which approach was taken.

## Evidence required before merge
- Build passes.
- Manual confirmation not required from Antigravity (no browser testing) — but report exact code diffs for all three so user can verify by testing manually.

No unrelated changes. Do not touch cancellation RLS, invite code validation logic, or anything outside these three fixes.
