# TableFlow — Chat #4 / Node 3 — Instruction (Investigation Only)

**To:** Antigravity
**From:** Claude
**Type:** Investigation — NO code changes, NO fixes in this prompt

---

## Task

Investigate the existing routing/dashboard architecture and give evidence-backed confirmation on which pattern Node 3 (Manager/Cook/Waiter dashboards) should follow: **role-specific routes** (e.g. `/dashboard/cook`) vs **one shared `/dashboard` with conditional rendering**.

Claude's working hypothesis (from locked Node 1 docs): role-specific routes, because the original Node Map already specifies "Waiter/Cook/Manager dashboards (3 separate views)" and the existing OAuth callback already does role-based redirect logic for Owner/Customer. This needs verification against the actual current code before Node 3 spec is finalized.

## Files to Investigate (read-only — do not edit)

1. `app/auth/callback/route.ts` — confirm current role-based redirect logic (what happens for `owner` vs `customer`, and what happens for `waiter`/`cook`/`manager` today — likely falls through to `/order`)
2. `app/dashboard/*` — list existing folder structure under `app/dashboard/` to confirm the convention (e.g. `app/dashboard/tables`, `app/dashboard/orders`, `app/dashboard/menu`, `app/dashboard/analytics`)
3. `middleware.ts` (if it exists) — check if there's any role-gating middleware already, or if all gating happens client-side/in the callback route
4. `types/index.ts` — confirm the current `UserRole` / role type definition includes `waiter`, `cook`, `manager` already, or only `owner`/`customer`

## What to Report Back

For each of the two options (role-specific routes vs shared dashboard), give:
- Whether the current codebase already leans toward one pattern (cite exact file/line evidence)
- Effort estimate to extend vs retrofit, based on what's actually there today
- Any risk you see in either direction given the current RLS/auth setup

Do **not** write or suggest actual route code yet — this is investigation and recommendation only. The fix/build instruction will be a separate prompt once this is reviewed.

## Output

Save findings to: `03_Investigation_and_Errors/Chat4_Node3_Investigation_RoutingArchitecture.md`

Report back to Claude with the file path once done.
