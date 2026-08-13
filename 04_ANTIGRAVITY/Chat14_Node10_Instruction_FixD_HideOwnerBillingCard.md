# Instruction — Chat 14, Node 10, Fix D — Hide Billing card/nav for Owner

## Problem
Fix B correctly blocked Owner from Billing via RLS + route guard (`dashboard/billing/[orderId]` shows "Access Restricted" for Owner — confirmed working). But the Owner Dashboard still shows a "Billing" card ("Generate itemized bills with optional service charge") that leads nowhere useful for Owner — clicking it or navigating there just hits Access Restricted. Same class of issue as the Manager/Menu case in Fix C.

## Decision
Since Billing is now Manager-exclusive, remove the Billing card from Owner Dashboard's card grid entirely. Do NOT touch the route guard (already correct) or the RLS policy (already correct from Fix B). This is a dashboard-card visibility fix only, same scope discipline as Fix C's nav fix.

## Frontend change
File: likely `app/dashboard/page.tsx` (Owner Dashboard card grid — the one showing Live Orders, Menu Management, Tables & Waitlist, Staff Management, Billing, Analytics, AI Insights cards).

- Find the Billing card entry in that grid.
- Remove it (or wrap with `{role !== 'owner' && ...}` if the same component is reused for Manager's dashboard — confirm first whether Owner Dashboard and Manager Dashboard are separate components/pages or share one with conditional cards).
- Do not remove or alter any other card (Live Orders, Menu Management, Tables & Waitlist, Staff Management, Analytics, AI Insights all stay for Owner).

## Scope boundary
- Only touch the Owner Dashboard card grid. Do NOT touch:
  - The billing route guard (`dashboard/billing/[orderId]`) — already correct, leave as-is.
  - The RLS policy on billing — already correct from Fix B, leave as-is.
  - Manager Dashboard's Billing Queue section (Image 1 reference — Manager's actual billing UI with Mark Paid/Print PDF) — that must keep working exactly as-is, this fix does not touch Manager's dashboard at all.

## Verification checklist (Ayush, manual browser test — screenshot each)
- [ ] Owner Dashboard: Billing card no longer appears in the grid
- [ ] Owner Dashboard: all other cards (Live Orders, Menu Management, Tables & Waitlist, Staff Management, Analytics, AI Insights) still present and unchanged
- [ ] Manager Dashboard: Billing Queue section still fully functional (Mark Paid, Print PDF, receipt view) — confirm no regression
- [ ] Owner: direct URL to a billing route still shows Access Restricted (already working, just confirm no regression)

## Standing rules (unchanged)
- No GitHub push without Ayush's explicit go-ahead after verification.
- Antigravity: code execution + build/compile check only. No browser testing.
