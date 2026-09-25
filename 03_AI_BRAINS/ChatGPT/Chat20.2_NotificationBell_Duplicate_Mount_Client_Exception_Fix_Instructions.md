# Chat 20.2 — NotificationBell Duplicate-Mount Client Exception Fix

Date: 2026-09-25
Related investigation: Chat 20.1 — Mobile Client-Side Exception Root Cause Investigation
Source repository: https://github.com/ayush22cp008/TableFlow
Primary source file: components/Navbar.tsx
Related component: components/NotificationBell.tsx

## 1. Objective

Fix the confirmed client-side exception affecting Manager, Cook, and Waiter after the Phase 1 mobile Navbar change.

The root cause is duplicate mounting of NotificationBell inside Navbar.tsx. The fix must preserve the existing TableFlow workflow and all notification functionality.

## 2. Confirmed Root Cause

The current responsive Navbar renders NotificationBell in both the desktop navigation area and the mobile right cluster.

CSS classes such as hidden sm:flex and flex sm:hidden control visual display only. They do not prevent React from mounting both component instances.

For staff roles, both instances use the same Supabase realtime channel name: notifications:<userId>.

The second instance attempts to add its postgres_changes callback after the channel has already subscribed, producing:

Error: cannot add postgres_changes callbacks for realtime:notifications:<userId> after subscribe().

This exception crashes the React tree and produces the Next.js client-side exception page.

Confirmed role behavior:

Manager -> NotificationBell -> exception
Cook -> NotificationBell -> exception
Waiter -> NotificationBell -> exception
Customer -> no NotificationBell -> working
Owner -> no NotificationBell -> working

The investigation identified commit 9906e4cb7ef4ee355c3705c2d222f236372a80c1 as the commit that introduced the duplicate NotificationBell rendering.

## 3. Required Fix

There must be exactly ONE mounted NotificationBell instance per Navbar.

Do not keep two NotificationBell components and rely on CSS to hide one.

Target architecture:

Navbar
  └── ONE NotificationBell
       └── responsive positioning/layout

The same mounted instance must work on desktop and mobile.

## 4. Preserve Existing Notification Functionality

Do not remove or change:

- Supabase notification query
- realtime INSERT subscription
- unread count
- notification list
- mark-as-read behavior
- PushSubscriptionButton
- click-outside handling
- notification message display
- timestamps
- role filtering
- recipient filtering

The fix is structural and responsive. Do not redesign the notification system.

## 5. Preserve Existing Navbar and Business Workflow

Customer navigation must remain: Menu, My Orders, Reservation, Sign Out.

Owner navigation must remain: Overview, Menu, Tables, Orders, Analytics, AI Insights, Staff, Sign Out.

Manager navigation must remain: Overview, Tables, NotificationBell, Sign Out.

Cook and Waiter must retain their current Navbar behavior and NotificationBell access.

Do not change authentication, routing, order processing, table allocation, claiming, billing, or any completed Node behavior.

## 6. Preserve Phase 1 Mobile Responsiveness

Do not revert the mobile responsive work.

Mobile must still provide:

- no horizontal Navbar overflow
- fully visible TableFlow branding
- hamburger menu for authenticated users
- working mobile navigation drawer
- NotificationBell for Manager/Cook/Waiter
- accessible Sign Out
- notification dropdown inside the viewport

Keep the existing notification dropdown viewport-safe behavior. The current intended width rule is w-80 with max-w based on the viewport.

## 7. Source Files

Primary file:
components/Navbar.tsx

Only modify components/NotificationBell.tsx if required for responsive positioning of the single instance.

Do not change NotificationBell Supabase/realtime business logic.

## 8. Explicitly Out of Scope

Do not modify:

- app/order/cart/page.tsx
- Party Size logic
- database schema
- migrations
- Supabase RLS
- authentication
- Node 9
- Node 12
- Node 13
- order workflow
- table allocation
- order claiming
- push notification backend
- unrelated pages/components

The Party Size mobile bug is a separate Phase 2 task.

## 9. Preferred Implementation Direction

Remove the duplicate NotificationBell mount from Navbar.tsx.

Keep one NotificationBell instance and make its surrounding layout/position responsive using CSS/Tailwind.

Do not create two React instances merely to obtain different desktop/mobile positions.

Do not introduce a global Context/provider or major architectural refactor unless absolutely required.

Keep the implementation minimal and targeted.

## 10. Regression Test Matrix

Test Manager:
- login
- dashboard loads
- Navbar loads
- NotificationBell appears
- no client-side exception
- open notification panel
- notifications display
- unread badge works
- mark-as-read works
- realtime notification still arrives

Test Cook with the same checks.

Test Waiter with the same checks.

Smoke-test Customer:
- /order loads
- mobile hamburger works
- no NotificationBell appears
- no client-side exception

Smoke-test Owner:
- owner dashboard loads
- mobile hamburger works
- no NotificationBell appears
- no client-side exception

## 11. Viewport Test Matrix

Test at:
- 360 x 800
- 375 x 812
- 390 x 844
- 412 x 915
- desktop width >= 1280

For Manager, Cook, and Waiter specifically test the NotificationBell at narrow mobile widths.

Verify:
- no horizontal page overflow
- no cropped Navbar
- no cropped notification panel
- no client-side exception

## 12. Runtime Verification

Specifically verify that a staff user has exactly one mounted NotificationBell instance per Navbar.

Verify that the notifications:<userId> realtime subscription is created once for the component.

Verify that the application no longer attempts to add duplicate postgres_changes callbacks to an already subscribed channel as a consequence of desktop/mobile rendering.

Do not simply rely on a successful build. Perform runtime role-based testing.

## 13. Build and Static Checks

Run at minimum:

npx tsc --noEmit
npm run build

Run the repository lint command if it is available and usable.

## 14. Git Workflow

Before editing:
1. Sync the latest main branch.
2. Confirm commit 9906e4c is present.
3. Inspect the current Navbar and NotificationBell code.

After implementation:
1. Review the exact diff.
2. Confirm only intended files changed.
3. Run TypeScript/build/lint checks.
4. Perform the role and viewport tests above.
5. Commit the fix with a clear message such as: fix(ui): prevent duplicate NotificationBell realtime subscription
6. Push to the TableFlow main branch.
7. Record the resulting commit SHA.

## 15. Required Final Result File

Create:
04_ANTIGRAVITY/Chat20.2_NotificationBell_Duplicate_Mount_Client_Exception_Fix_Result.md

The report must contain:

- confirmed root cause
- files changed
- exact fix implemented
- confirmation that NotificationBell is mounted exactly once
- confirmation that existing notification functionality was preserved
- roles tested
- routes tested
- viewport widths tested
- TypeScript/build/lint results
- realtime notification result
- desktop/mobile result
- confirmation that Party Size was untouched
- confirmation that Node 9, Node 12, and Node 13 were untouched
- confirmation that database/authentication/business workflows were untouched
- commit message
- commit SHA
- push status

## 16. Guardrails

Do NOT:

- render NotificationBell twice and hide one with CSS
- remove NotificationBell from mobile
- remove NotificationBell from Manager, Cook, or Waiter
- disable Supabase realtime
- remove notifications
- suppress the exception instead of fixing the duplicate mount
- revert all mobile responsiveness work
- modify Party Size
- make unrelated refactors

Correct target:

Two NotificationBell mounts
  -> ONE NotificationBell mount
  -> responsive positioning
  -> ONE realtime subscription
  -> no client-side exception
  -> existing workflow preserved

## 17. Final Objective

The deployed TableFlow application must allow Manager, Cook, and Waiter to log in, load their dashboards normally, see one NotificationBell, receive realtime notifications, open the notification panel, and use the existing notification controls on both mobile and desktop.

Customer and Owner behavior must remain unchanged.

This is a targeted implementation fix for the confirmed duplicate-mount exception. It is not a new feature and must not alter the existing TableFlow business workflow.