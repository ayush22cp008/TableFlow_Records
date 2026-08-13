# Feature Spec — Track B: Reservation Priority + Order Numbering

Chat #1 | Node: Track B (Node 2, per original master prompt node map — feature, not bug fix)

## Feature Goal

1. Orders placed via a verified reservation code get tagged as priority and always sort to the top of the kitchen/order queue.
2. Every order (reservation or walk-in) gets a sequential daily number, so kitchen staff know order sequence.

## Decided Spec

### Tagging
- New field on orders: `is_priority` (boolean) — `true` if the order was placed using a verified reservation code, `false` for walk-in orders.

### Numbering — Two Separate Daily Counters
- **Reservation orders:** own sequence, prefixed `R` — R1, R2, R3... resets daily.
- **Walk-in orders:** own sequence, prefixed `W` — W1, W2, W3... resets daily.
- Both counters reset at the start of each day (midnight, consistent with existing day-boundary logic already used elsewhere in the app, e.g. reservation date handling).

### Queue Sorting
- In every column on `/dashboard/orders` (Order Placed, Preparing, Ready, Served): priority (`R`) orders always appear **above** all walk-in (`W`) orders, regardless of placement time.
- Within each group (priority vs walk-in), sort by placement time (existing behavior, unchanged).

### Display
- The R#/W# number is shown on the order card in **all 4 columns** (Order Placed, Preparing, Ready, Served) — not just the initial column.
- Priority (`R`) orders get a distinct visual badge/highlight to make them stand out, in addition to the number.

## Explicitly Out of Scope
- No changes to Track A / Auto-Hide reservation logic — those are locked and verified.
- No changes to billing, table assignment, or approval flow.
- No shared/unified counter — reservation and walk-in numbering are intentionally separate series.

## Engineering Discipline Reminders
- Single source of truth: the daily counter value should be derived/stored in a way that doesn't require dual updates (e.g. a counter table or a computed sequence per day — avoid maintaining two separate fields that must stay in sync).
- Build on an isolated git branch, verify with `npm run build`, then follow existing push protocol (commit+push to main after Ayush's go-ahead, Vercel deploys, live testing after).
- Evidence required before calling this done: place a reservation order and a walk-in order on the same day, confirm R1/W1 (or correct next numbers) appear correctly in all 4 columns, and confirm priority order sorts above walk-in in each column.
