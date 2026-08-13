Chat #3 | Node 2b | Instruction — Investigate 3 Bugs Found in Manual Testing (INVESTIGATION ONLY, no fix)

User tested the deployed build and found 3 issues. Investigate each separately, report findings — do not fix yet.

## Bug 1: Signup page missing role selection (Waiter/Cook/Manager)
Screenshot shows "Create Account" -> "Who are you joining as?" with only Customer and Owner options. No Waiter/Cook/Manager option, and no invite code input field visible.

Investigate:
- Find the signup page component (app/signup or similar).
- Confirm whether the 3-role staff signup panel (role selector + invite code field + email) was ever actually built into this page, or only designed/speced but never implemented.
- Report exact file path and current state of the signup flow.

## Bug 2: "Cancel ALL Active Orders" button does nothing
Screenshot shows Bulk Emergency Stop modal, "Cancel ALL Active Orders" mode selected, category + optional details filled in, button clicked — no visible result (no success, no error, modal doesn't close, orders don't change).

Investigate:
- Find the button's onClick handler / form submit logic.
- Check browser console for JS errors when this is clicked (ask user for a screenshot of DevTools Console if not already captured, or check code for whether errors are being silently swallowed e.g. empty catch block).
- Confirm the RPC (cancel_active_orders) is actually being called — check network tab evidence or add/check logging.
- Report exact root cause: is it a frontend wiring issue (button not calling the RPC), an RPC error being swallowed, or an RLS/permission rejection?

## Bug 3: "Select Specific Orders" mode has no way to actually select orders
Screenshot shows modal saying "Please select the orders on the board behind this modal" but the Live Orders board (screenshot provided) has no checkboxes or selection UI on the order cards.

Investigate:
- Confirm whether checkbox/selection UI was ever implemented on the order cards for this mode, or only the modal-side messaging was built.
- Report exact file path of the order card component and current state.

## Output
Report to: 03_Investigation_and_Errors/Chat3_Node2b_Investigation_SignupAndBulkStopBugs.md
Include: exact root cause per bug (implemented-but-broken vs. never-implemented), file paths, and any console/network evidence found.

No fixes in this pass — investigation and evidence only.
