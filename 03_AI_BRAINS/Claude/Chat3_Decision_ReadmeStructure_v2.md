# Decision: README Structure (LOCKED — for end-of-project execution)

## Timing
NOT to be done now. README rebuild/update happens at project END, once all Staff Role System nodes are complete and tested — noted here now so it isn't forgotten.

## Structure
README must have two clearly-labeled, separated sections:

1. **VibeAthon 6.0 Submission** — describes exactly what existed at the July 30 hackathon deadline (real-time menu, digital ordering, table/waitlist management with seat-level capacity, AI menu insights, sales analytics, transparent billing, Google OAuth, OTP verification, customer reservation portal, custom domain). This section should reflect the submitted state as-is, not the current state.

2. **Post-Hackathon Updates** — everything built after submission for portfolio/job purposes. Starts with the Staff Role System (invite codes, role-based RLS across orders/tables/menu, order cancellation system with single + bulk emergency stop) and will include whatever comes after.

## Verification Method (how to determine which section a feature belongs to)
Use **git commit history**, not memory or manual comparison against the existing README (which was itself flagged incomplete in Round 1 judge feedback).

- **Cutoff:** VibeAthon submission deadline = 30 July 2026, 11:59 PM IST.
- Run `git log` and split all commits at that exact timestamp:
  - Commits **before/at** cutoff -> VibeAthon Submission section.
  - Commits **after** cutoff -> Post-Hackathon Updates section.
- Cross-check the git-derived list against the Drive bridge folder's locked decision docs (this folder) for context/reasoning on why each post-hackathon feature was added — Drive records supplement the git ground-truth, they don't replace it.

## Why
- Fair/accurate representation of what judges evaluated vs. what exists now.
- Shows continued development to recruiters/portfolio viewers — proof of ongoing work past the hackathon deadline.
- Git log is objective/timestamped, avoiding reliance on a README that was already flagged as incomplete at submission time.

## Next
Revisit this at project end (after all planned Staff Role System nodes are locked and tested). Run `git log` with the cutoff, cross-reference Drive decision docs, then draft the README content.
