# Instruction — Investigate Why Specific Old Requests Still Not Hidden (Regression on Panel Fix)

**Chat #9 — Investigation only. Do NOT write any fix.**

## Context
Chat9_Instruction_FixReservationRequestsPanel_OptionC.md was applied and pushed (query date window + 5-min-after-seated client filter). Ayush confirms it partially works: a NEW request he tested does correctly disappear from the panel ~5 min after being seated.

## Bug observed
However, 5 specific old entries are STILL visible in the panel despite the fix: "mahesh" (Party of 4), "ajh" (Party of 5), "ayushhalpati" (Party of 3), "Za" (Party of 2), "Zarna" (Party of 2) — all showing "Seated" badge. These are the same ones from before the fix. Total count still shows "Reservation Requests (30)".

## Investigate

1. For each of these 5 request rows (mahesh, ajh, ayushhalpati, Za, Zarna): query `reservation_requests` and report their exact `requested_time`, `status`, and `table_id`.
2. Check why the Part 1 query-level date filter (`requested_time >= startOfToday`) did NOT exclude them — report whether their `requested_time` is actually within today's window (if so, the filter is working as designed and these are correctly "today's" data, just old within today) or whether the filter itself has a bug (timezone mismatch, wrong date boundary calculation, etc).
3. Check the Part 2 client-side filter for these 5 specific rows: does each have a `linkedOrder`? If yes, report the linked order's `created_at`/`status` and why the "5 min after seated" hide logic isn't triggering for them. If no `linkedOrder` exists for these rows, report that explicitly — this may be the actual cause (the hide logic likely depends on finding a `linkedOrder`, and if these old test rows have no matching order, the hide condition never fires).
4. Report total current count breakdown: how many of the 30 visible rows are "old stuck" like these 5, vs normal/expected pending or recently-seated entries.

## Output required
Pure findings — exact data per row, confirm/reject whether missing `linkedOrder` is the cause, and clarify whether Part 1 (date filter) or Part 2 (client hide logic) is the failure point, or both. No fix. Report back before any fix is proposed.
