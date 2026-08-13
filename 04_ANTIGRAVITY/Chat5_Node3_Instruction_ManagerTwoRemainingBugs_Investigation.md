# TableFlow — Chat #5 / Node 3 — Instruction (Investigation): Two Remaining Manager Issues

**To:** Antigravity
**From:** Claude
**Type:** Investigation ONLY — read the actual code, do not guess, do not fix yet

---

## Context

The stale-cookie + fetch-cache fixes (commit after `Chat5_Node3_Instruction_ManagerRoutingStaleCache_Fix.md`) improved things but did NOT fully resolve the manager routing problem. Ayush ran a controlled test this time (incognito, both signup paths tested separately) and found **two distinct, reproducible bugs**:

## Bug A — Manual signup ("Create Account") still routes to Customer page

Steps: Fresh incognito → manual email/password signup with invite code → lands on `/order` (Customer Menu page), not `/dashboard/manager`.

Investigate:
- Trace the manual signup code path completely — likely `app/auth/select-role/page.tsx` or wherever manual signup completes and redirects, separate from the Google OAuth `callback/route.ts` path already fixed.
- Confirm whether this path reads `profiles.role` fresh (post our `cache: 'no-store'` fix) or whether manual signup has its own redirect logic that was never touched by the previous fixes.
- Check: does the manual signup flow even correctly set `role = 'manager'` in the DB at the point of redirect, or could there be a timing issue where the redirect fires before the invite-code-to-role assignment write completes?

## Bug B — Google Sign-In reaches `/dashboard/manager` but content never loads (infinite spinner)

Steps: Fresh incognito → Google Sign-In → correctly lands on `/dashboard/manager` (heading renders: "Manager Dashboard — Intake and Billing Queues"), but the queue content area shows a permanent loading spinner and a "Refreshing..." button state that never resolves.

Investigate:
- This means **routing now works** for this path — the bug has moved to `app/dashboard/manager/page.tsx`'s own data-fetching logic.
- Check the client-side data fetch (Supabase query for intake/billing queues) — is it stuck on a pending promise? Check browser console errors if reproducible, or trace the query code for something that could hang (e.g., an RLS policy silently blocking the query without erroring, a malformed `.select()` join, or a fetch that never resolves/rejects).
- Check whether this component also constructs its own Supabase client — if so, confirm whether it needs the same `cache: 'no-store'` treatment, or whether this is a completely separate issue (e.g., RLS silently returning zero rows forever vs. hanging, or a `.single()` call erroring silently, or missing `await`).

## Do NOT fix yet

Report back with:
- Root cause for Bug A (exact code path + why it doesn't route correctly)
- Root cause for Bug B (why the spinner never resolves — check for actual errors, not just assumed hang)
- Proposed fix for each (description only — do not implement)

Treat these as two separate bugs with two separate root causes — do not assume they share a cause just because both are "manager-related."
