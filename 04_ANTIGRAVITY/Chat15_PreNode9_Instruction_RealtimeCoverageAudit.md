# Instruction — Realtime Coverage Audit (Pre-Node 9 Check)

**Type:** Investigation only. Do NOT write/modify any code. Report findings only.

## Context

Before building Node 9 (Notification System) on top of existing Supabase Realtime infrastructure, need a clear picture of which dashboards/pages already update automatically (no refresh needed) vs which still require a manual refresh to see new data. This is NOT a fix task — just a coverage report. Any gaps found will be triaged separately as their own task, not fixed here.

## What to investigate and report

For **each** of the following pages/dashboards, report:
- Does it have a Supabase Realtime subscription (`postgres_changes` or `broadcast`) set up? If yes, which channel name and which table(s)/event(s) does it listen to?
- Based on the code (not assumption), will a change made elsewhere (e.g. another user updating an order) reflect on this page automatically, or does the page only fetch data once on load / on manual action (e.g. button click, page refresh)?

### Pages to check
1. Owner Dashboard (`app/dashboard/orders/page.tsx` and any other Owner-specific views)
2. Manager Dashboard (`app/dashboard/manager/page.tsx`)
3. Cook Dashboard (locate the relevant file — likely `app/dashboard/cook/page.tsx` or similar)
4. Waiter Dashboard (locate the relevant file — likely `app/dashboard/waiter/page.tsx` or similar)
5. Tables page (`app/dashboard/tables/page.tsx`)
6. Customer — My Orders (`app/order/my-orders/page.tsx`)
7. Customer — Reservation status (`app/order/reservation/page.tsx`)
8. Customer — Menu/ordering page (`app/order/page.tsx`)
9. Staff Management page (wherever Owner manages staff — locate file)
10. Any other dashboard page not listed above that displays data which can change from another user's action

## Output format

A simple table: Page | Realtime subscription (Y/N + channel name) | Auto-updates or needs manual refresh | File path

## Explicitly out of scope
- No fixes, no code changes, no recommendations on how to fix gaps.
- This is a coverage snapshot only — decisions on what to fix will be made after reviewing this report.

## Output

Save findings to: `03_Investigation_and_Errors/Chat15_PreNode9_Investigation_RealtimeCoverageAudit.md`
