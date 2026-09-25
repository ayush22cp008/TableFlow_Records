# Chat 20.6 — TableFlow Cart UTF-8 Mojibake Regression Fix Instructions

Date: 2026-09-25
Source repository: https://github.com/ayush22cp008/TableFlow
Records repository: https://github.com/ayush22cp008/TableFlow_Records
Related fix: Chat 20.5 — Party Size Mobile Input Fix
Primary source file: app/order/cart/page.tsx
Scope: Restore correct UTF-8 text encoding in Cart page without changing Party Size or business logic

## 1. Objective

Fix the new text-encoding regression visible on the deployed TableFlow Cart page after the Chat 20.5 Party Size fix.

Observed mojibake examples include:
- â‚¹ instead of ₹
- âœ“ instead of ✓
- Ã— instead of ×
- â€” instead of —
- â³ instead of ⏳
- ðŸŽ‰ instead of 🎉
- ðŸ›’ instead of 🛒
- âš ï¸ instead of ⚠️

The Party Size fix itself is working and must be preserved.

## 2. Confirmed Regression Source

The current main branch contains the Party Size fix in app/order/cart/page.tsx.

Commit:
263dd88b163fb5d38470d523136282c81dd7a63c

Commit message:
fix(cart): allow intermediate input states for Party Size on mobile

Compared with the previous commit, this commit modified only:
app/order/cart/page.tsx

The current Cart source contains corrupted Unicode sequences such as:

â‚¹
Ã—
âœ“
â€”
â³
ðŸŽ‰
ðŸ›’
âš ï¸

This indicates a text-encoding/mojibake regression in the source file.

## 3. Required Fix

Restore the intended Unicode characters in app/order/cart/page.tsx.

Examples:
- â‚¹ -> ₹
- Ã— -> ×
- âœ“ -> ✓
- â€” -> —
- â³ -> ⏳
- ðŸŽ‰ -> 🎉
- ðŸ›’ -> 🛒
- âš ï¸ -> ⚠️

Do NOT rewrite unrelated text.

The source file must be saved as valid UTF-8.

## 4. Preserve Chat 20.5 Party Size Fix

Do not revert or alter the working Party Size implementation.

Preserve:
const [partySizeInput, setPartySizeInput] = useState('1')

Preserve derived numeric behavior:
const partySize = Math.max(1, parseInt(partySizeInput) || 1)

Preserve:
- value={partySizeInput}
- onChange={(e) => setPartySizeInput(e.target.value)}
- onBlur normalization
- min={1}
- max={maxCapacity}
- disabled={!!verifiedReservation}

Party Size must continue to accept 2, 3, 4, 5, etc. on mobile.

## 5. Texts That Must Remain Correct

Verify every user-facing Unicode symbol in app/order/cart/page.tsx, including:

- currency symbol ₹
- multiplication symbol ×
- checkmark ✓
- em dash —
- hourglass ⏳
- celebration emoji 🎉
- cart emoji 🛒
- warning emoji ⚠️

Also inspect any other non-ASCII characters in the file for mojibake.

Do not replace intended Unicode characters with unrelated ASCII approximations unless the existing product design explicitly uses ASCII.

## 6. Do Not Change Business Logic

Do NOT modify:
- cart quantity logic
- subtotal calculation
- reservation-code verification
- reservation completion
- table allocation
- waitlist creation
- order creation
- order RPC
- Party Size validation semantics
- Navbar
- NotificationBell
- authentication
- database schema
- Supabase RLS
- Node 9
- Node 12
- Node 13

This is an encoding/text-only regression fix.

## 7. Source Scope

Primary and expected only file:
app/order/cart/page.tsx

Do not modify additional files unless a build/tooling check proves the encoding issue is generated elsewhere. If another file must change, document why.

## 8. UTF-8 Verification

After fixing the file:

1. Confirm the file is saved as UTF-8 without an unintended encoding conversion.
2. Search the file for known mojibake patterns such as â, Ã, and ðŸ.
3. Verify those corrupted sequences are absent where they represented intended Unicode characters.
4. Verify the correct Unicode characters are present.
5. Inspect the rendered page in the browser.

Do not solve the issue by adding arbitrary encoding declarations to individual strings.

## 9. Browser Verification

Test the deployed/local Cart page on:
- mobile browser
- desktop browser

Verify the screen renders:

Your Cart
₹30.00
₹40.00
₹70.00
× 1
✓ Place Order

and any applicable status messages/emojis render correctly.

## 10. Party Size Regression Test

While testing the encoding fix, verify the Chat 20.5 behavior remains intact:

- enter 2
- enter 3
- enter 4
- enter 5
- clear and retype
- select all and replace

The Party Size field must not regress to the old snap-back-to-1 behavior.

## 11. Reservation Regression

Verify that the reservation code UI still displays correctly and that the existing reservation flow is unchanged.

Verify:
- Verify button text is readable
- Clear button text is readable after verification
- reservation value displays correctly
- Party Size locks correctly after reservation verification

## 12. Order/Waitlist Regression

Run a basic smoke test to verify:
- Place Order text renders correctly
- existing loading text renders correctly
- success text renders correctly
- waitlist status text renders correctly
- no business logic changed

## 13. Build Checks

Run:
npx tsc --noEmit
npm run build

Run the configured lint command if available and usable.

## 14. Diff Review

Before commit, inspect the exact diff.

The expected change should be limited to restoration of Unicode/text encoding in app/order/cart/page.tsx.

Do not allow accidental changes to:
- Party Size state logic
- Party Size handlers
- order functions
- reservation functions
- waitlist functions
- unrelated formatting/refactoring

## 15. Git Workflow

Before editing:
1. Sync latest main.
2. Confirm commit 263dd88 is present.
3. Inspect current app/order/cart/page.tsx.

After editing:
1. Review diff.
2. Confirm only intended encoding changes are present.
3. Run TypeScript/build/lint.
4. Run mobile and desktop Cart smoke tests.
5. Verify Party Size still works.
6. Commit with a clear message, for example:
fix(cart): restore UTF-8 text encoding
7. Push to main.
8. Record commit SHA.

## 16. Required Antigravity Result Report

Create:
04_ANTIGRAVITY/Chat20.6_TableFlow_Cart_UTF8_Mojibake_Regression_Fix_Result.md

Include:
- regression cause
- exact file changed
- corrupted sequences identified
- Unicode characters restored
- confirmation Party Size fix was preserved
- mobile verification
- desktop verification
- order/reservation/waitlist smoke tests
- TypeScript result
- build result
- lint result if run
- diff scope
- commit message
- commit SHA
- push status

## 17. Guardrails

Do NOT:
- revert Chat 20.5
- rewrite Party Size logic
- change database/business logic
- change Navbar
- change NotificationBell
- replace Unicode with different product text
- add unrelated refactors
- modify Node 9/12/13

Target:

Corrupted source text
      -> restore intended UTF-8 Unicode
      -> preserve Chat 20.5 Party Size fix
      -> preserve all Cart business workflows
      -> verify mobile + desktop
      -> commit and push

## 18. Final Objective

The Cart page must display the original intended Unicode text correctly on both mobile and desktop while preserving the successfully fixed Party Size editing behavior and all existing order, reservation, table-allocation, and waitlist workflows.

This is a narrow encoding-regression fix only.