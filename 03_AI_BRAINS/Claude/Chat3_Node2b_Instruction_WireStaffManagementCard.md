Chat #3 | Node 2b | Instruction — Wire Staff Management Card into Owner Dashboard

Per investigation (Chat3_Node2b_Investigation_MissingStaffUIAndOrdersPageDiff.md, Issue 2): `app/dashboard/staff/page.tsx` already exists and is fully built, but `app/dashboard/page.tsx` (Lines 24-31) hardcodes 6 quick-action cards and omits `/dashboard/staff`.

## Fix
Add a 7th card to the Owner Dashboard's quick-action grid linking to `/dashboard/staff`, matching the same visual pattern as the existing 6 cards (icon + title + one-line description).

Suggested copy (adjust to match existing tone):
- Title: "Staff Management"
- Description: "Generate invite codes, manage staff accounts"
- Icon: pick one consistent with the existing set (👥 or similar)

## Scope
Only touch `app/dashboard/page.tsx` — do not modify `app/dashboard/staff/page.tsx` itself unless you find it's broken when you open it (if so, stop and report back, don't fix in this same pass, per evidence rule).

## Evidence required
- Build must pass
- Screenshot or description confirming the card renders and links correctly
- Report to: `03_Investigation_and_Errors/Chat3_Node2b_Evidence_StaffManagementCardWired.md`

## Next
Once this is verified and Ayush confirms via manual testing, this + the earlier deploy fix are ready for a final commit+push (using the standard commit/push reusable prompt).
