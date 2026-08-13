# Instruction — Node 11 Bug Investigation: Reservation Reject Not Updating + Refresh Resets to Form

**Type:** Investigation only. Do NOT write/modify any code. Report findings only.

## Bug report (from Ayush, manual testing)

Realtime replication is now confirmed ON for `reservation_requests` (verified via Supabase Dashboard screenshot). Testing results:

1. **Approve works** — when Manager approves a reservation, the customer side (`app/order/reservation/page.tsx`) updates automatically without refresh, showing the `unique_code`.
2. **Reject does NOT work** — when Manager rejects a reservation, the customer side does not show any "rejected/cancelled" state. No update is visible.
3. **New bug found separately**: If the customer refreshes the page, they are taken back to the reservation request FORM (as if they have no active reservation), allowing them to submit a duplicate/re-request — even though their original request still exists in the database.

## What to investigate and report

### 1. Reject-specific code path
- Find the `rejectRequest` function in `app/dashboard/tables/page.tsx` (or wherever reject logic lives). Compare it directly against the `approveRequest` function used for approval.
- Report: does reject actually update the `status` column in `reservation_requests` to something like `'rejected'`? Confirm the exact status value written on reject vs approve.
- Confirm the reject action fires an UPDATE on the same row the customer's realtime filter (`customer_id=eq.${user.id}`) is watching — i.e. does reject touch the same row/columns, or does it do something different (e.g. delete the row instead of updating status)?

### 2. Customer page's handling of different statuses
- In `app/order/reservation/page.tsx`, find the `checkReservation` function (the realtime callback). Report exactly what statuses it currently handles/displays (e.g. does it only have UI logic for `'approved'` and not for `'rejected'`? Is there a fallback/default case, or does an unhandled status silently render nothing?).

### 3. Refresh-resets-to-form bug
- Find the initial data-fetch logic in `app/order/reservation/page.tsx` (what runs on page load, not the realtime subscription — the initial `useEffect`/fetch that determines whether to show the form or the existing reservation status).
- Report the exact query used to determine "does this customer have an active reservation". Compare this query's status filter against what values reject/approve/pending actually use. Hypothesis to check: the initial fetch may only look for `status = 'pending'` or `status = 'approved'`, and a `'rejected'` (or whatever value reject actually writes) falls outside that filter, causing the page to treat the customer as having no active reservation and show the form again.

## Explicitly out of scope
- No fixes. Report only what exists today and exact evidence (code snippets, exact status values, exact queries).

## Output

Save findings to: `03_Investigation_and_Errors/Chat15_Node11_Investigation_ReservationRejectAndRefreshBug.md`
