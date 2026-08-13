# Investigation: Force Logout vs Deactivate Overlap Check

**Type:** Investigation only — no code changes.

## Context
Owner Side Staff Management includes a "force logout" feature (already verified, not yet pushed). A separate pending fix (`Chat9_Implementation_FixRemoveBanProperRoleGate.md`) will set `is_logged_in: false` inside the staff **deactivate** route. Before approving that plan, we need to know if these two overlap.

## Investigate
1. Find the force logout implementation — which file/route handles it.
2. Show exactly how it sets/clears `is_logged_in` (or any related session field).
3. Check if it shares any function/logic with the deactivate route (`app/api/staff/deactivate/route.ts`), or if the two are fully independent code paths.

## Output
Report findings only — file paths, relevant code snippets, and a clear answer: is there duplicate logic between force logout and deactivate, or are they separate concerns? No fix, no refactor yet.
