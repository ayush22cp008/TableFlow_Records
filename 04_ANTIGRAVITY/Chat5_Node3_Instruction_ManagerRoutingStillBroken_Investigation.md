# TableFlow — Chat #5 / Node 3 — Instruction (Investigation): Manager Routing Still Inconsistent After Fix

**To:** Antigravity
**From:** Claude
**Type:** Investigation ONLY — read the actual code, do not guess, do not fix yet

---

## Bug Report

The fix from `Chat5_Node3_Instruction_ManagerRoutingBug_Fix.md` (commit `8457721`, confirmed pushed and live on production) has NOT resolved the issue. Ayush tested repeatedly (~30 min, fresh incognito windows, multiple browsers — Chrome/Brave/Firefox) logging in as the `manager`-role account (`ayushhalpati008@gmail.com`) and observed **inconsistent results across attempts**:

1. Sometimes lands on **Owner Dashboard**
2. Sometimes lands on **Customer/Menu page** (`/order`)
3. Once saw the **Manager Dashboard** loading state (heading + spinner) — outcome after that unconfirmed (Ayush isn't certain if it fully loaded or stalled)

Ayush was not tracking every exact combination of login method (Google Sign-In vs manual) per attempt, so treat this as **inconsistent/non-deterministic behavior**, not a single reproducible path.

## What This Pattern Suggests

Getting three different destinations for the same account across attempts points to a **timing/race condition**, not a plain wrong-redirect logic bug (which would be consistent every time). Read the code — do not assume. Specifically check:

1. **`app/auth/callback/route.ts`** (Google Sign-In path): How is `role` fetched before redirect? Is it awaited from a fresh DB query, or could it read from a session/JWT claim that might not be populated yet on first login?
2. **`app/auth/select-role/page.tsx`** (manual signup path): Same question — is the role value guaranteed fresh at the moment `window.location.href` fires, or is there any async gap where a stale/default value could be used?
3. **`middleware.ts`** (the safeguard added in the last fix): How does it determine role — cookie, JWT, or DB call? If it reads from a JWT/session claim that was minted before the `profiles.role` update (e.g., right after invite-code role assignment), it could redirect based on stale data on the first request or two after signup, explaining why repeated attempts (with fresh sessions each time) give different results.
4. Check whether there's any caching (client-side or edge) of the role value that could serve a stale response intermittently.

## Do NOT fix yet

Report back with:
- The exact code path(s) for both Google Sign-In and manual signup, quoting the relevant logic
- Where role is read from at each decision point (fresh DB call vs session/JWT/cache)
- Your diagnosis of why the same account could get 3 different outcomes across attempts
- Proposed fix (description only — do not implement)

This is a repeat investigation after a fix already landed and failed to resolve it, so be thorough — trace the actual execution path in the code rather than re-stating the previous (already-applied) fix.
