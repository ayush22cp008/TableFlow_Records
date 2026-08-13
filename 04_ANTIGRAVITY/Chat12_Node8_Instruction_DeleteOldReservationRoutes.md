# Instruction: Delete Orphaned Reservation Routes

**Node:** 8 — Customer Dashboard Revamp
**Type:** FIX — Cleanup. Small scope, single decision already made by Ayush.

## Context
`/order/reservation` (new, authenticated, customer_id-linked) has replaced the old anonymous reservation flow. Navbar no longer links to the old routes — they are orphaned and allow anonymous submission again if directly accessed, which defeats the customer_id linking work just done.

## Change required
Delete:
- `app/reserve/page.tsx`
- `app/reserve/status/page.tsx`

Also remove any now-unused helper functions/components that were exclusively used by these two pages (check for orphaned imports). Do not touch anything still referenced by `/order/reservation` or elsewhere.

## Output
Confirm both files deleted, confirm build still passes, confirm no other route/component references the deleted files. No push — wait for explicit go-ahead per project rule.
