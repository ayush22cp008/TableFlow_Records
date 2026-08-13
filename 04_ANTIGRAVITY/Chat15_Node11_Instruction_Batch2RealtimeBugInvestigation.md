# Instruction — Node 11 Batch 2 Bug Investigation: Realtime Not Working At All

**Type:** Investigation only. Do NOT write/modify any code. Report findings only.

## Bug report (from Ayush, manual testing)

None of the 3 Batch 2 pages auto-update — not even while staying on the page (this is different from the Batch 1 tab-switch issue, which was a one-off and resolved itself). Manual refresh is required in all 3 cases, every time, even without switching tabs.

Affected files:
1. `app/order/reservation/page.tsx` (`reservation_status_realtime` on `reservation_requests`)
2. `app/dashboard/staff/page.tsx` (`staff_management_realtime` on `profiles`)
3. `app/dashboard/menu/page.tsx` (`owner_menu_realtime` on `menu_items`)

## What to investigate and report

### 1. Show the exact current subscription code
Paste the exact `useEffect`/channel setup code as it currently exists in all 3 files, in full (not summarized) — channel creation, `.on(...)`, `.subscribe()`, cleanup.

### 2. Compare against the known-working Batch 1 pattern
Show the working `app/dashboard/manager/page.tsx` subscription code again for reference, and do a line-by-line diff against each of the 3 Batch 2 files. Flag ANY difference, however small (typo in table name, wrong event filter, missing `.subscribe()` call, wrong channel/table name, filter syntax error, etc).

### 3. Check Supabase Realtime is enabled for these tables
- Confirm whether `reservation_requests`, `profiles`, and `menu_items` tables have Realtime replication enabled in Supabase (this is a per-table toggle in Supabase — check via `supabase/migrations/` for `ALTER PUBLICATION supabase_realtime ADD TABLE ...` statements, or note if this needs to be checked directly in Supabase dashboard, which Ayush would need to confirm manually).
- Compare against `orders` and `menu_items` used elsewhere — is `menu_items` already enabled for realtime via the existing `menu_realtime` channel in `app/order/page.tsx`? If yes, and `owner_menu_realtime` in `app/dashboard/menu/page.tsx` subscribes to the same table, report why one might work and not the other (e.g. is `owner_menu_realtime` even correctly wired, or is the issue elsewhere on that page).

### 4. Check for the `customer_id=eq.${user.id}` filter specifically on reservation page
- Report the exact filter syntax used in `app/order/reservation/page.tsx`. Supabase Realtime filter syntax is strict — confirm it matches Supabase's documented filter format exactly (e.g. `filter: 'customer_id=eq.' + userId` as a string, correctly placed in the `.on()` config object).
- Confirm `user.id` is actually populated/available at the point the subscription is set up (not `undefined` due to async auth loading timing).

### 5. RLS policies on these 3 tables
- Report current RLS SELECT policies on `reservation_requests`, `profiles`, and `menu_items`. Realtime respects RLS — if the current user's role can't SELECT rows via RLS, the realtime event won't be delivered even if the subscription is technically correct. Report policy definitions for each table.

## Explicitly out of scope
- No fixes. Report only what exists today and exact differences/evidence found.

## Output

Save findings to: `03_Investigation_and_Errors/Chat15_Node11_Investigation_Batch2RealtimeNotWorking.md`
