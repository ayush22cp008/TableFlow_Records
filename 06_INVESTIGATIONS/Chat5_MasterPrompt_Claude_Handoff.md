# TableFlow — Chat #5 → Chat #6 Handoff (Claude-Side)

**Project:** TableFlow Staff Role System
**Drive bridge root:** `TableFlow_Staff_Role_System` (ID `16Xhn59dqZg2_XVMnqulmLeywO3CfhzyL`)
**GitHub:** ayush22cp008/TableFlow · **Live:** table-flow-nu.vercel.app

---

## Node-Map (current status)

| Node | Status |
|---|---|
| Node 1 — Permission Matrix | ✅ LOCKED |
| Node 2b Part 1 — Schema/RLS/Cancellation | ✅ LOCKED |
| Node 2b Part 2 — Invite Email (Resend) | ✅ LOCKED |
| Node 3 — Routing architecture | ✅ LOCKED |
| Node 3 — Cook Dashboard | ✅ LOCKED |
| Node 3 — Waiter Dashboard | ✅ LOCKED (per Chat 5 handoff carried over — verify if uncertain) |
| **Node 3 — Manager Dashboard** | 🔄 **ACTIVE — blocked on 2 open bugs, see below** |
| Node 4 — Notification System (polling) | ⬜ NOT STARTED |

## Manager Dashboard — Locked Scope (Chat #5)

- Order intake: Placed → Preparing (R/W)
- Billing closure: Placed status uses `billed` (NOT `completed` — matches live locked RLS policy `manager_served_to_billed`, confirmed do not rename)
- Payment method: simple dropdown (Cash/Card/UPI), record-only field, no gateway integration
- Bill detail: itemized on-screen (items, qty, price, line total, grand total)
- Bill export: PDF via `window.print()` + `@media print` CSS (no new dependency)
- Bulk actions: none — per-order only (matches Cook/Waiter pattern)
- Tables/Reservations/Menu management: full R/W — reuses existing Owner UI (Navbar exposes links to Manager role, no duplicate pages built)
- No access: Sales Analytics, AI Insights, Staff Management, Restaurant Settings

## Build History This Chat (chronological)

1. Manager Dashboard built per spec (`Chat5_Node3_Instruction_ManagerDashboard.md`) — schema (`payment_method` column), routing, Intake/Billing queue UI, PDF export. Build passed.
2. Status-terminology correction: instruction originally said `completed`, corrected to `billed` to match live RLS (`Chat5_Node3_Instruction_ManagerDashboard_StatusCorrection.md`) — no new migration needed, UI matched to existing schema.
3. **Bug found (live test):** Manager login routed to Owner Dashboard instead of `/dashboard/manager`.
   - Root cause: hardcoded bug in `app/auth/select-role/page.tsx` — `manager` ternary branch pointed to `/dashboard` instead of `/dashboard/manager`. Predates this session; nobody had wired the manager branch since only Cook/Waiter existed before.
   - Fix applied: corrected the redirect + added `middleware.ts` safeguard (any staff role hitting bare `/dashboard` gets bounced to their own dashboard). Confirmed pushed to GitHub (commit `8457721`) and live on Production.
   - Waiter gap noted: same ternary still has no `waiter` branch (falls through to `/order`) — correctly NOT touched since Waiter Dashboard scope wasn't part of this fix; flag if Waiter routing needs revisiting.
4. **Bug persisted after fix 3** — inconsistent results across repeated fresh-incognito logins (~30 min testing, multiple browsers): sometimes Owner Dashboard, sometimes Customer `/order` page, once a correctly-routed but unconfirmed Manager Dashboard load.
   - Root cause (two separate issues found via code investigation, not guessing):
     - **Bug 1 (Owner result):** `app/auth/callback/route.ts` called `getUser()` after `exchangeCodeForSession()`, which read the *stale incoming request cookie* instead of the freshly established session — served the old Owner profile if a prior Owner session cookie existed.
     - **Bug 2 (Customer result):** `middleware.ts` queried `profiles.role` via a Supabase client whose underlying `fetch` was cached by Next.js Edge runtime by default — stale `customer` role kept getting served even after DB was updated to `manager`.
   - Fix applied: (a) callback.ts now uses the `user` object already returned by `exchangeCodeForSession()` instead of a redundant stale `getUser()` call; (b) `cache: 'no-store'` forced on the Supabase server client's fetch in `middleware.ts` and `lib/supabase-server.ts` (and any other server-client construction sites, per scope-check instruction). Build passed, reported pushed.
5. **Bug persisted differently after fix 4** — Ayush ran a *controlled* test this time (separated by signup method):
   - **Bug A:** Manual "Create Account" signup still routes to `/order` (Customer Menu), not `/dashboard/manager`.
   - **Bug B:** Google Sign-In now correctly reaches `/dashboard/manager` (heading renders correctly), but the queue content area never loads — permanent spinner + stuck "Refreshing..." state. Routing is fixed for this path; this is now a data-fetching bug inside the Manager Dashboard page itself, not a routing bug.
   - Investigation instruction sent (`Chat5_Node3_Instruction_ManagerTwoRemainingBugs_Investigation.md`) — **treats these as two separate root causes, not one shared cause.** Antigravity told explicitly: read the code, don't guess; check manual-signup's own redirect path separately from the OAuth path already fixed; for Bug B check for silent RLS blocks, malformed `.select()` joins, missing `await`, or whether the dashboard page's own Supabase client also needs the `cache: 'no-store'` fix.

## Standing Correction / Process Note

Earlier in this chat there was a moment of confusion about an unplanned GitHub push — turned out Ayush had directly instructed Antigravity to use a reusable commit-and-push instruction file himself, so it was NOT a violation of the "push only with explicit permission" rule. No process gap exists; noting this only so Chat #6 doesn't need to re-litigate it if it comes up.

## STATUS AT HANDOFF — result pending

**Investigation result for Bug A + Bug B has NOT been reviewed yet.** Ayush will paste Antigravity's result into Chat #6 as the first message. Chat #6 should:
1. Read the investigation result fresh (do not assume the hypotheses above are confirmed — they were the investigation *ask*, not the *answer*).
2. Confirm root cause(s) with Ayush before approving a fix, same as this chat's pattern (investigation and fix always separate prompts).
3. Once both bugs are fixed and Ayush confirms via live browser test (screenshot or verbal "maine check kiya hai"), mark Node 3 — Manager Dashboard ✅ LOCKED and update the Node-Map.
4. After Manager Dashboard locks, Node 4 (notification system) is the next unstarted node — not yet scoped in any chat.

## Reminders carried forward

- File creation needs permission except `02_Instructions/` files.
- Investigation and fix are always separate prompts — never combined.
- Antigravity-reported build success is not evidence; Ayush's live browser test (screenshot or verbal confirmation) is the gold standard.
- GitHub push is manually triggered by Ayush.
- All instruction-type files live exclusively in `02_Instructions/`.
