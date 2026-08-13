# Instruction — Node 11 Batch 3: Realtime Auto-Update (Owner Analytics, Insights, Main Dashboard)

**Type:** Fix. Implement code changes as specified below.

## Context

Final batch of Node 11. Follow the same verified working pattern from Batch 1 and Batch 2 (`useEffect` with channel setup, `.on('postgres_changes', ...)`, `.subscribe()`, cleanup via `supabase.removeChannel(channel)`).

**Lesson learned from Batch 2 (apply here too):** Before subscribing to any table, confirm via Supabase Dashboard → Database → Replication that the table's Realtime toggle is ON. If any table used by these 3 pages is not yet enabled, flag it explicitly in your report rather than assuming — Ayush will need to toggle it manually, same as `profiles` and `reservation_requests` were handled.

## Scope — 3 files only

1. `app/dashboard/analytics/page.tsx` (Owner Analytics)
2. `app/dashboard/insights/page.tsx` (Owner Insights — AI Insights)
3. `app/dashboard/page.tsx` (Owner Main Dashboard)

## What to do

For each file:
- First identify which table(s) each page actually queries for its data (likely `orders`, possibly `order_items`, `menu_items` — confirm from the existing fetch logic in each file, don't assume).
- Add a Supabase Realtime subscription on the relevant table(s), following the Batch 1/2 pattern.
- On relevant events, re-fetch/recompute so the displayed numbers/charts update without a manual refresh.
- Give each channel a distinct name, e.g. `owner_analytics_realtime`, `owner_insights_realtime`, `owner_dashboard_realtime`.
- Keep any existing manual refresh in place as fallback.
- Do not change business logic, calculations, or UI layout beyond what's needed to wire up the subscription.

## Constraints (per project rules)

- No DB/schema changes beyond confirming/flagging replication toggle status (the toggle itself is a manual Dashboard action for Ayush, not something Antigravity does).
- No RLS changes.
- Do not touch any other files outside the 3 listed above.
- Do not touch Node 10's locked role-permission logic.
- After implementation: report files modified, confirm build passes, confirm no unexpected files touched.

## Testing note

Ayush will manually verify in-browser, including checking Realtime toggle status per table before testing (per the lesson from Batch 2).

## Output

Report back with: files modified, build result, exact channel names used, exact table(s) subscribed per page, and explicit confirmation/flag of each table's current Realtime toggle status if known (or note that Ayush needs to check).
