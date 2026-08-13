# TableFlow — Chat #6 → Chat #7 — Master Prompt: Claude-Side Handoff

## Node-Map
- Node 1 (Permission Matrix) — ✅ LOCKED
- Node 2b (Schema, RLS, invite codes, cancellation, email) — ✅ LOCKED
- Routing architecture — ✅ LOCKED
- Cook Dashboard (KDS) — ✅ LOCKED
- **Node 3 (Manager Dashboard) — ✅ LOCKED (as of Chat 6)**
- 🔄 **Waiter Dashboard + `waiter` branch in `AuthForm.tsx` — ACTIVE (next task, NOT locked — correction from earlier draft)**
- Manager Service Charge parity — ⬜ NOT STARTED (deferred, low priority, revisit later)
- Node 4 (notifications) — ⬜ NOT STARTED

**Correction note:** An earlier draft of this handoff incorrectly marked "Waiter Dashboard core" as ✅ LOCKED. This was wrong — the `waiter` redirect branch in `AuthForm.tsx` was never built (same gap as Bug A was for `manager`, but for waiter it's still unfixed), so Waiter Dashboard cannot be considered locked or even reachable via normal signup yet. Corrected here to 🔄 ACTIVE.

## Chat 6 Summary — Node 3 closed out

Three bugs found and fixed this chat, each with confirmed evidence:

1. **Bug A** — `AuthForm.tsx` missing `manager` redirect branch (3 locations: `handleStaffSignup`, `handleVerifyOtp`, `handleLogin`). Fixed. Confirmed via manual signup landing on `/dashboard/manager`. The equivalent `waiter` branch was intentionally left untouched at the time — still missing, now the next task.
2. **Bug B** — Infinite spinner loop in `manager/page.tsx` (`fetchOrders` `useCallback` had `paymentMethods` in its own deps, causing self-retrigger). Fixed. Confirmed via live dashboard load, no spinner hang.
3. **Bug C** — "Mark Paid" returning 400. Root cause: live database's `orders.status` CHECK constraint was stale (schema file said `'billed'` was allowed, but `CREATE TABLE IF NOT EXISTS` skipped re-applying it on the already-existing live table). Secondary cause: `payment_method` column migration also never applied to live DB. Both fixed manually via Supabase Dashboard SQL Editor (CLI `npx supabase db push` failed on Windows — known win32-x64 binary issue, worked around via direct SQL Editor execution). Confirmed via live test: order clears from Billing Queue, Owner's Live Orders "Served" count drops to 0.

**Full Manager Dashboard functionality confirmed working end-to-end:** Intake Queue → Accept → Preparing; real-time sync across Manager/Owner/Cook/Customer views; Billing Queue display; payment method dropdown; Mark Paid; Print PDF (valid PDF confirmed via Producer metadata).

**Pushed to GitHub:** Yes, confirmed.

**Deferred decision:** Manager Dashboard has no Service Charge toggle (Owner's separate billing page does, at 10%, optional). Decision made this chat: skip for now, revisit later as its own small task — not urgent, not a bug.

## Next Task (Chat 7 starting point)

Add the missing `waiter` redirect branch in `AuthForm.tsx` (3 locations, same file, same pattern as Bug A's `manager` fix). This is the actual current gap — Waiter Dashboard itself has not been confirmed built/tested/locked in any prior chat; only the auth routing gap is confirmed as a known, well-understood issue at this point.

**Scope reminder:** the fix pattern is already known and proven (identical to Bug A, just for `waiter`) — but per the "investigation and fix separate" rule, confirm with Ayush whether to skip straight to a fix instruction or do a quick confirm-only pass first. Also confirm with Ayush what the actual current state of Waiter Dashboard is (built? tested? never touched?) before assuming anything about it — don't inherit the incorrect "locked" assumption from the earlier draft.

## Standing notes for Chat 7

- Supabase CLI (`npx supabase db push`) does not work on this Windows machine (win32-x64 binary not found). Manual SQL Editor execution via Supabase Dashboard is the working alternative — use this if any future schema change needs live-DB application.
- Antigravity's local `.env.local` does not have live DB push credentials — any live DB changes need Ayush's manual action via Dashboard.
