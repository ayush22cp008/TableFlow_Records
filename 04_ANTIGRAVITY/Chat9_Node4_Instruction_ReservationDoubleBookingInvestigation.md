# Instruction — Investigate Reservation Double-Booking (Node 4)

**Chat #9 — Node 4 — Investigation only. Do NOT write any fix.**

## Bug observed
Two Reservation Requests were approved for the same table with overlapping windows:
- Table 1 (Seats 2): "Ak" party approved 3:55 PM, "Ar" party approved 3:45 PM — both approved/seated simultaneously.
- Side effect: table's seated count showed 4/2, exceeding capacity.

## Investigate

1. Locate the reservation **Approve** handler — likely on Owner or Manager Tables page, wherever the "Reservation Requests" list with Approve/Reject buttons lives. Report exact file + function name.
2. Check whether the Approve handler performs ANY validation before approving:
   - Overlap check against other active/approved reservations on the same table
   - Current occupancy check (`occupied_seats` vs `seats` capacity)
3. Report the relevant schema fields involved at approval time: `occupied_seats`, `reserved_from`, `seats` (capacity), and how the Approve action currently mutates them.
4. Report whether approving a second reservation on an already-reserved/occupied table is currently possible with no guard at all, or if some partial check exists but failed in this case.

## Output required
Terminal/code findings only — file paths, function names, relevant code snippets, and a clear yes/no on whether an overlap/capacity check exists. No fix. Report back to Claude side before any fix is proposed.
