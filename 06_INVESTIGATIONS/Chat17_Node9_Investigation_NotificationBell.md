# TableFlow — Node 9 Investigation: NotificationBell Build Failure + Silent Notifications

**Project:** TableFlow Staff Role System
**Node:** 9 — In-App Notifications
**Chat:** #17
**Date:** 2026-08-17
**Status:** IN PROGRESS — awaiting Ayush's Vercel deployment-list confirmation

---

## OBSERVATION

Two symptoms reported together, not yet established as one root cause or two:

**1. Build failure (Vercel deploy log, `next build`)**
File: `./components/NotificationBell.tsx`
ESLint rule: `@typescript-eslint/no-unused-vars`

| Line | Symbol | Error |
|---|---|---|
| 5:10 | `supabase` | defined but never used |
| 8:44 | `userId` | defined but never used |
| 8:52 | `role` | defined but never used |
| 10:25 | `setNotifications` | assigned a value but never used |
| 11:23 | `setUnreadCount` | assigned a value but never used |

Result: `Failed to compile.` → `Error: Command "npm run build" exited with 1`

This is a **new/different error** than the one addressed by `Chat16_Node9_Fix_BuildFailure.md` in the prior chat — that fix targeted an earlier ESLint failure; this one surfaced afterward and was never diagnosed.

**2. Runtime — notifications not appearing (screenshots, live/prod site)**
- Manager Dashboard (`/dashboard/manager`): bell icon present, dropdown shows "No notifications" — even after Ayush placed an order that should have triggered `order_placed` (Manager should always be notified per the locked type-mapping).
- Kitchen Display / Cook view (`/dashboard/cook`, mobile): bell icon present, no visible badge.
- Waiter Dashboard (`/dashboard/waiter`, mobile): bell icon present, no visible badge, despite an order sitting in "ready" state.

All three roles show the bell icon rendering (UI shell present) but zero notification content across the board.

## Why these two symptoms are being investigated together

If the component that defines `supabase`, `userId`, `role`, `setNotifications`, `setUnreadCount` — and never uses any of them — is the *same* component rendering on all three dashboards, that is consistent with a component whose data-fetching/subscription logic was never wired up (stubbed placeholders, per Chat 16 handoff notes: "intentional placeholders for Step 3 wiring"). That would mean:
- The bell renders (static shell) ✅ — matches screenshots.
- No data is ever fetched or subscribed to ❌ — matches "No notifications" everywhere.
- ESLint flags the unused variables because the fetch/subscribe code that would consume them doesn't exist yet ❌ — matches build log.

**This is a hypothesis, not a confirmed root cause.** It has NOT been verified against the actual current source of `NotificationBell.tsx`.

## Open question — blocking root cause confirmation

Not yet established: **which deployment is currently live in production.**

- If the failed build (Image 2) never went live, the site is serving an **older, pre-Node-9 build** — meaning the "No notifications" behavior is fully expected (old code has no bell logic at all) and is NOT a separate bug. Fix = resolve the 5 ESLint errors, redeploy, re-verify.
- If some other build did go live with partial Node-9 code, there could be a second, independent runtime bug beyond the build failure.

Ayush confirmed (in-chat) he is not yet sure which case applies and needs to check the Vercel **Deployments** tab (timestamps + which one is marked current/production) to resolve this.

## Evidence on file
- Vercel build log screenshot — Next.js 14.2.35, compiled successfully, then failed at lint stage (see error table above).
- 3 screenshots of live/deployed dashboards (Manager desktop, Cook mobile, Waiter mobile) all showing empty notification state.

## NOT yet done (do not skip ahead to these)
- Source of `components/NotificationBell.tsx` has not been viewed/reviewed in this chat.
- Vercel deployment list (timestamps, which is "Production") not yet confirmed.
- No fix has been proposed or written.

## Next step (Chat 17, immediate)
Ayush to check Vercel → Deployments tab and report:
1. Timestamp + status of the deployment matching this failed build log.
2. Timestamp of whichever deployment is currently marked live/Production.

Once that's confirmed, proceed to view `NotificationBell.tsx` source before writing any fix instruction — per investigation pipeline (OBSERVATION → INVESTIGATION → EVIDENCE → ROOT CAUSE → DECISION → FIX → BUILD/TEST → AYUSH VERIFICATION). Currently at INVESTIGATION stage.
