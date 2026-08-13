# Instruction — Node 11 Batch 2: Realtime Auto-Update (Reservation Status, Staff Management, Menu Management)

**Type:** Fix. Implement code changes as specified below.

## Context

Coverage audit confirmed these 3 pages currently require manual refresh to see new/updated data. Follow the same working pattern from Batch 1 (`app/dashboard/manager/page.tsx`, `app/dashboard/cook/page.tsx`, `app/dashboard/waiter/page.tsx`) — verified working, including correctly surviving tab-switch/background scenarios.

## Scope — 3 files only

1. `app/order/reservation/page.tsx` (Customer — Reservation Status)
2. `app/dashboard/staff/page.tsx` (Staff Management)
3. `app/dashboard/menu/page.tsx` (Owner Menu Management)

## What to do

For each file, add a Supabase Realtime subscription using `postgres_changes`, following the exact pattern used in Batch 1 (`useEffect` with channel setup, `.subscribe()`, cleanup via `supabase.removeChannel(channel)` on unmount).

**Per-file specifics:**

1. **`app/order/reservation/page.tsx`** — Subscribe to `reservation_requests` table, filtered or matched to the logged-in customer's own reservation (this page already fetches via `customer_id = user.id`, per existing investigation — keep that scoping, just make it realtime). On change, re-fetch so the customer sees approval/rejection and their `unique_code` appear without refresh.

2. **`app/dashboard/staff/page.tsx`** — Subscribe to whichever table backs staff records (likely `profiles` or `staff`, confirm exact table name from the existing fetch query in this file before subscribing). On INSERT/UPDATE/DELETE, refresh the staff list.

3. **`app/dashboard/menu/page.tsx`** — Subscribe to `menu_items` table (same table already used by the existing `menu_realtime` channel in `app/order/page.tsx` — reuse the same table, just a new channel name for this page).

- Give each channel a distinct name to avoid collisions with existing channels, e.g. `reservation_status_realtime`, `staff_management_realtime`, `owner_menu_realtime`.
- Keep existing manual refresh (if any) in place as fallback — only add auto-update on top.
- Do not change business logic, role permissions, or UI layout beyond what's needed to wire up the subscription.

## Constraints (per project rules)

- No DB/schema changes — frontend-only.
- No RLS changes.
- Do not touch any other files outside the 3 listed above.
- Do not touch Node 10's locked role-permission logic.
- After implementation: report files modified, confirm build passes, confirm no unexpected files touched.

## Testing note

Ayush will manually verify in-browser (including tab-switch scenario, since that's now part of the verification bar per Batch 1). Do not claim UI verification.

## Output

Report back with: files modified, build result, exact channel names used, and exact table name subscribed to for Staff Management (since it needs confirming from existing code first).
