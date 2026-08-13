Chat #3 | Node 2b | Instruction — Investigate Missing Staff UI + Unexpected File Change (INVESTIGATION ONLY, no fix)

Post-deploy manual check on live URL found issues. Investigate each separately, report findings — do not fix yet.

## Issue 1: Signup page shows only Customer/Owner, no staff role option
Screenshot: `table-flow-nu.vercel.app/signup` → "Who are you joining as?" → only Customer and Owner cards shown. No Waiter/Cook/Manager option, no email-based staff flow visible at all.

Investigate:
- Read the current `components/AuthForm.tsx` (post-push version) — does it actually contain the new staff role → email → send-invite → code+password flow described in the implementation plan?
- Is the staff option missing from the UI's role array/selector, or is it present in code but not rendering?
- Cross-check against `app/auth/select-role/page.tsx` if it still exists — is this separate page still disconnected from the main signup flow (same issue as the earlier-fixed Bug 1)?
- Report exact current state with file path + line references — implemented-but-hidden vs. never wired up vs. regressed.

## Issue 2: No "Staff Management" panel on Owner Dashboard
Screenshot: Owner Dashboard shows Live Orders, Menu Management, Tables & Waitlist, Billing, Analytics, AI Insights. No Staff Management card/link anywhere.

Investigate:
- Was a Staff Management UI (for Owner to generate invite codes) ever built as part of Node 2a/2b, or only speced?
- If built, why isn't it showing on the dashboard — missing route, missing nav card, or something else?
- Report exact file path and current state.

## Issue 3: Unexpected change to app/dashboard/orders/page.tsx
The commit (`be97449`) modified this file, but it was explicitly out of scope per the instruction ("Do not touch Owner/Customer signup path, RLS policies, or the cancellation system").

Investigate:
- Show the exact diff for this file in the commit.
- Was this an intentional necessary change (e.g. shared component/type used by both AuthForm and orders page) or an accidental/unrelated change?
- Confirm whether this affects the cancellation system (single-cancel, bulk emergency stop) functionality at all.

## Output
Report to: `03_Investigation_and_Errors/Chat3_Node2b_Investigation_MissingStaffUIAndOrdersPageDiff.md`

Include exact root cause per issue, file paths, relevant code snippets, and the orders page diff.

No fixes in this pass — investigation and evidence only.
