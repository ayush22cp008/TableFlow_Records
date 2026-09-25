# Chat 20.5 — TableFlow Party Size Mobile Input Fix Instructions

Date: 2026-09-25
Source repository: https://github.com/ayush22cp008/TableFlow
Records repository: https://github.com/ayush22cp008/TableFlow_Records
Related investigation: 04_ANTIGRAVITY/chat20.4_party_size_investigation.md
Primary source file: app/order/cart/page.tsx
Scope: Targeted Party Size input fix only

## 1. Objective

Fix the confirmed Party Size input bug on the customer Cart page.

Current problem:
- Party Size initially shows 1.
- On mobile, replacing 1 with 2, 3, 4, 5, etc. is not reliable.
- Clearing the controlled number input temporarily produces an empty string, but the current handler immediately turns it back into 1.

Root cause is confirmed as a React controlled-input issue caused by:
onChange={(e) => setPartySize(Math.max(1, parseInt(e.target.value) || 1))}

Do not change unrelated TableFlow functionality.

## 2. Confirmed Root Cause

When the user edits the input on mobile, the browser can emit an intermediate empty value.

The current handler effectively does:
empty string -> parseInt -> NaN -> fallback 1 -> setPartySize(1)

Because the input is controlled by value={partySize}, React immediately renders 1 again.

This blocks normal replacement of the initial value.

The investigation also confirmed:
- verifiedReservation is not the cause on a fresh cart.
- maxCapacity is not the cause.
- no other effect is resetting partySize.
- the bug is pre-existing and was not introduced by the Navbar/mobile navigation work.
- Reservation page Party Size is separate and must not be changed.

## 3. Required Fix

Implement a controlled-input strategy that allows intermediate editing states while still providing a valid numeric Party Size to existing business logic.

Preferred approach from the investigation:

1. Keep a string-backed input state initialized to '1'.
2. Use the string state as the input value.
3. In onChange, store the raw input string without forcing an empty value back to 1.
4. Normalize the string to a valid numeric value on blur and/or immediately before order placement.
5. Keep the numeric value used by table allocation, waitlist, and RPC logic.

The existing investigation also lists a simpler alternative: allow an empty string during editing and only update the numeric state for non-empty valid numbers. Antigravity may choose that alternative if it is safer for the existing component, but it must satisfy all acceptance criteria below.

## 4. Required User Behavior

On a fresh cart with no reservation code, the customer must be able to:
- replace 1 with 2
- replace 1 with 3
- replace 1 with 4
- replace 1 with 5
- backspace and type a new value
- select all and type a new value
- paste a valid number
- edit naturally with the mobile soft keyboard
- edit normally on desktop

During editing, an empty intermediate state must not force the visible value back to 1 immediately.

After editing is complete, the value used by business logic must be a valid number.

## 5. Preserve Validation Semantics

Preserve the existing logical constraints:
- minimum Party Size = 1
- maximum Party Size = maxCapacity

Do not silently convert valid values such as 2, 3, 4, or 5 into 1.

Preserve the existing maxCapacity error message and button-disabled behavior when the final Party Size exceeds maxCapacity.

Do not change how maxCapacity is calculated.

## 6. Preserve Reservation Behavior

Keep the existing reservation lock behavior:
disabled={!!verifiedReservation}

When a valid reservation code is applied:
- reservation verification must still work
- reservation party_size remains authoritative
- Party Size displays the reservation value
- manual Party Size editing remains disabled

Do not modify reservation-code verification or reservation completion logic.

## 7. Preserve Business Workflows

Party Size is used for:
- best-fit table allocation
- available-seat comparison
- waitlist party_size
- order RPC p_party_size

After the fix, these workflows must receive the corrected numeric value.

Do not change table allocation rules, waitlist rules, or order creation architecture.

## 8. Source Scope

Primary file:
app/order/cart/page.tsx

Normally only this file should change.

Do not modify:
- components/Navbar.tsx
- components/NotificationBell.tsx
- app/order/reservation/page.tsx
- database schema
- migrations
- Supabase RLS
- authentication
- Node 9
- Node 12
- Node 13
- unrelated pages/components

## 9. Order Placement Regression

Verify normal no-reservation order placement using Party Size values:
- 2
- 3
- 4

Confirm the available-table calculation receives the selected numeric Party Size.

Confirm the RPC continues to receive numeric p_party_size.

Verify the waitlist path stores the selected numeric Party Size when no suitable table is available.

## 10. Mobile Test Matrix

Use a real phone when possible.

Test viewport widths:
- 360 x 800
- 375 x 812
- 390 x 844
- 412 x 915

Fresh-cart test sequence:
1. Log in as Customer.
2. Open /order.
3. Add at least one item.
4. Open /order/cart.
5. Do not apply a reservation code.
6. Focus Party Size.
7. Replace 1 with 2.
8. Repeat with 3, 4, and 5.
9. Clear the field and type a new number.
10. Select all and type a new number.
11. Paste a valid number.
12. Enter a value above maxCapacity.

Record whether the value remains stable after each edit.

## 11. Desktop Regression

Run the same Party Size editing tests on desktop.

Desktop behavior must remain functional after the fix.

## 12. Edge Cases

Test:
- empty value while editing
- 0
- negative value if browser allows it
- value above maxCapacity
- multi-digit value
- rapid replacement
- backspace followed by typing

The component must preserve the existing logical minimum and maximum semantics without breaking normal editing.

## 13. Reservation Regression

Use a valid reservation code in a controlled test.

Verify:
- reservation verification succeeds
- reservation party_size loads correctly
- Party Size reflects reservation value
- Party Size remains disabled
- order placement continues to use the reservation party size

Do not redesign this flow.

## 14. Build Checks

Run:
npx tsc --noEmit
npm run build

Run the configured lint command when available and usable.

A successful build is necessary but not sufficient; perform runtime tests.

## 15. Regression Boundaries

Explicitly verify that the fix does not affect:
- cart item quantities
- subtotal
- reservation-code verification
- table allocation
- waitlist creation
- order creation
- order RPC behavior
- Navbar
- NotificationBell
- authentication
- existing Node functionality

## 16. Git Workflow

Before editing:
1. Sync latest main.
2. Confirm the current Party Size implementation.
3. Read the Chat 20.4 investigation result.
4. Review the exact diff scope.

After implementation:
1. Review the diff.
2. Confirm the fix is isolated to Party Size logic/UI.
3. Run TypeScript/build/lint checks.
4. Run mobile and desktop Party Size tests.
5. Run order-placement regression tests.
6. Run reservation regression tests.
7. Run waitlist regression testing.
8. Commit with a clear message, for example:
fix(order): allow mobile party size editing
9. Push to main.
10. Record the commit SHA.

## 17. Required Antigravity Result Report

Create:
04_ANTIGRAVITY/Chat20.5_TableFlow_Party_Size_Mobile_Input_Fix_Result.md

Include:
- confirmed root cause
- exact implementation chosen
- exact source file(s) changed
- before/after behavior
- mobile test results
- desktop test results
- Party Size values tested
- maxCapacity test
- reservation-code regression
- order-placement regression
- waitlist regression
- TypeScript/build/lint results
- confirmation Navbar/NotificationBell were untouched
- confirmation Reservation page was untouched
- confirmation Node 9/12/13 were untouched
- confirmation database/authentication were untouched
- commit message
- commit SHA
- push status

## 18. Guardrails

Do NOT:
- immediately reset an empty editing state back to 1 on every onChange
- remove maxCapacity handling
- remove the minimum logical Party Size
- bypass reservation locking
- modify the Reservation page Party Size implementation
- modify table allocation rules
- modify order-placement architecture
- modify Navbar or NotificationBell
- make unrelated UI changes

The expected interaction is:
1 -> clear/edit -> 2
1 -> clear/edit -> 3
1 -> clear/edit -> 4
1 -> clear/edit -> 5

without the controlled input snapping back to 1 during normal editing.

## 19. Final Objective

Enable reliable Party Size editing on mobile while preserving the existing numeric value semantics and all existing order, table-allocation, reservation, and waitlist workflows.

This is a targeted functional input fix, not a broader UI or business-logic change.