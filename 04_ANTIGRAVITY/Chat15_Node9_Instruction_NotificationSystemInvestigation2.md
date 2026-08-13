# Instruction — Node 9: Notification System Investigation (Round 2)

**Type:** Investigation only. Do NOT write/modify any code. Report findings only.

## Context

Follow-up to Chat15_Node9_Investigation_NotificationSystemDiscovery.md. Two design gaps found in that round need more ground-truth before Node 9 fix instruction can be written.

## What to investigate and report

### 1. `unique_code` on reservations — reuse for customer notification lookup?

- `reservation_requests` table has a `unique_code` field (already used for table verification). Report its exact schema: column type, how it's generated, is it unique per reservation or per customer, how long it persists.
- Confirm: after a reservation is approved/rejected, does the customer have any existing UI flow where they re-enter or already hold this `unique_code` to check status? Report where/how (file/component).
- Confirm: is there a lookup mechanism today (RPC, query) that takes a `unique_code` and returns reservation status? If yes, report it.

### 2. Does "Cancel ALL Active Orders" (Bulk Emergency Stop) include `Served` status orders?

- In `app/dashboard/orders/page.tsx`, find the exact query/filter that determines which orders are "active" for the "Cancel ALL Active Orders" bulk mode.
- Report the exact status list/filter used (e.g. does it query `status != 'served' AND status != 'cancelled'`, or something else).
- Confirm explicitly: can an order with status `Served` ever be included in a bulk cancel? Answer yes/no with the exact code/query as evidence.

### 3. Bulk Emergency Stop — confirm Owner-only

- Confirm in code that the Bulk Emergency Stop button/modal only renders/is accessible on the Owner dashboard route, and does not exist on the Manager dashboard route.
- Report the exact file(s) where this button exists and confirm no equivalent exists in `app/dashboard/manager/page.tsx`.

## Explicitly out of scope
- No schema design, no fix proposals. Report only what exists today.
- Do not touch previously investigated areas (order status flow, realtime channels, etc.) unless directly relevant to the 3 points above.

## Output

Save findings to: `03_Investigation_and_Errors/Chat15_Node9_Investigation_Round2_UniqueCodeAndServedStatus.md`
