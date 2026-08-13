# Instruction — Investigate Reservation Requests Panel Mechanism (New, Post-Node-4)

**Chat #9 — Investigation only. Do NOT write any fix. This is a new topic, unrelated to Node 4's overlap/lifecycle fix (that is locked and pushed).**

## Context
The "Reservation Requests" panel on the Tables dashboard currently shows 30 entries, including many that are clearly old/completed test data (e.g. "mahesh", "ajh", "Za", "Zarna", "A0", "Ui", "T2", "T3" — short/test-like names, all already showing `Seated` status). Ayush wants to understand what mechanism/logic currently governs this panel — not fix or change it yet.

## Investigate

1. **Locate the query/fetch logic** that populates the Reservation Requests panel (likely in `app/dashboard/tables/page.tsx`, wherever `reservationRequests` state is fetched from `reservation_requests`). Report the exact query — does it filter by status at all, or fetch all rows unconditionally?

2. **Report current status distribution**: how many rows exist in `reservation_requests` total, and break down by status (`pending`, `approved`, `completed`, `cancelled` — per the Node 4 lifecycle fix, `cancelled` should now also exist). Just report counts, don't modify anything.

3. **Explain what "Seated" badge means in the UI** — trace which status value(s) map to the "Seated" label shown next to names like mahesh/ajh/Za. Confirm whether "Seated" = `completed` status, or something else.

4. **Report whether there's any existing sort/limit/pagination** on this panel (e.g. ordered by `created_at`, limited to N rows, or truly showing everything unfiltered).

5. **Report whether old/completed/seated entries serve any current purpose in the UI** (e.g. are they referenced elsewhere, used for history/audit, or just visually cluttering the panel with no functional use).

## Output required
Pure findings — exact code/query, status counts, and how "Seated" is derived. No opinions on what to change, no fix, no refactor suggestions unless Ayush asks for them separately.
