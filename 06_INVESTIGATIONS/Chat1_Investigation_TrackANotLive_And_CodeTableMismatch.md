# Investigation — Track A Fix Not Reflected on Live Site + Reservation Code/Table Mismatch

Chat #1 | Node: Bug Investigation (post Track A push)

## Task

Investigate only. No fixes, no code changes.

## Observed Evidence (screenshots, live site table-flow-nu.vercel.app)

1. "Confirm Arrival" button still appears on every reservation request card on `/dashboard/tables`, despite Track A fix (removing Confirm Arrival) having been committed and pushed earlier in this session.
2. Reservation "rajesh" (Party of 2, no table selected/assigned in the request panel at approval time) was given code `258665`. Owner entered `258665` in the "Enter code" field on rajesh's card and clicked "Confirm Arrival".
3. Customer side (`/order/cart`): same code `258665` entered under "Have a reservation code?", Party Size set to `1` (not 2) → result: "Invalid or unused code, or missing table assignment."
4. Separately, Table 2 (which had no reservation approved for it in any screenshot) shows "Partially Occupied, 2/4 seated" — occupancy appeared with no visible reservation link.

## Investigate

### A. Why Track A fix isn't live
1. Check the actual latest commit on GitHub (main branch) for `app/dashboard/tables/page.tsx` and `app/order/cart/page.tsx` — does the committed code actually have Confirm Arrival removed, or was the wrong version committed/pushed?
2. Check Vercel deployment history/logs — what was the last deployment, which commit hash did it build from, did it succeed, and does it match the latest GitHub commit.
3. Report: is this a "fix never actually got committed correctly" issue, or a "deployment didn't pick up the new commit" issue, or something else.

### B. Reservation code / table assignment mismatch
4. Find rajesh's reservation_requests row: current status, assigned table_id (if any), code value, party_size, requested_time.
5. Explain why entering code `258665` with party_size `1` in cart returned "Invalid or unused code, or missing table assignment" — is party_size used to look up the table (per the cart UI's own label "used to find your table"), and does mismatched party_size (1 vs actual 2) cause this specific error? Or is there truly no table_id assigned to this reservation?
6. Find how/why Table 2 shows "2/4 seated, Partially Occupied" — trace what created this occupancy (which order, which table assignment logic, timestamp) and whether it's linked to any reservation record at all.
7. Report exact file paths, line numbers, and current field values for rajesh's reservation row and Table 2's occupancy source.

## Do Not
- Do not modify any files.
- Do not write a fix. Investigation only.
- Do not assume the cause — report what code, git history, Vercel logs, and DB rows actually show.
