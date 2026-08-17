# TableFlow — Chat 16 Master Prompt Handoff (Claude Side)

**Project:** TableFlow Staff Role System
**Chat #16 → Chat #17 handoff**
**Date:** 2026-08-14

---

## Node-Map Status

- ✅ LOCKED: Nodes 1–8, 10, 11
- 🔄 ACTIVE: **Node 9 — In-App Notifications**
- ⬜ NOT STARTED: Node 13 (Order Claiming) — depends on Node 9
- ⬜ NOT STARTED: Node 12 (Push Notifications) — depends on Node 13

## Node 9 — Step Progress

1. ✅ **LOCKED** — Schema design (`notifications` + `notification_reads` tables, RLS). Migration `20260814000001_node9_schema.sql` executed successfully in Supabase SQL Editor (screenshot-verified).
2. 🔄 **IN PROGRESS, BLOCKED ON BUILD ERROR** — Bell Icon UI (`components/NotificationBell.tsx` + Navbar integration). Component built per spec, but production build failed on `@typescript-eslint/no-unused-vars` (intentional placeholders for Step 3 wiring). Fix instruction issued (`Chat16_Node9_Fix_BuildFailure.md`) — **a NEW/different error surfaced after that fix was applied; not yet diagnosed in this chat.** This is the first task for Chat 17.
3. ⬜ NOT STARTED — Triggers at 8 confirmed `type` events (mapping finalized, see below)
4. ⬜ NOT STARTED — Bulk Cancel pre-RPC status-snapshot fix
5. ⬜ NOT STARTED — Enable Supabase Realtime replication on `notifications` table

## Key Locked Decisions (Node 9)

- **Per-user read tracking** via separate `notification_reads` table — not a shared `is_read` boolean.
- **Owner excluded** from all notifications (matches Node 10: Owner = read-only monitor).
- **Targeting model:** `recipient_role` (broadcast) XOR `recipient_id` (specific person) — enforced by `one_target_only` CHECK constraint.
- **`type` values (8 total):** `order_placed`, `order_preparing`, `order_ready`, `order_served`, `order_cancelled`, `reservation_requested`, `reservation_approved`, `reservation_rejected`.
- **`order_cancelled` is status-dependent** (not a fixed role set) — see mapping below. Always includes Manager and the customer; the staff role notified depends on order status at time of cancellation. This is intentionally role-broadcast for now (not targeted to the specific Cook/Waiter who was handling it) because no per-order staff assignment exists yet — **will need revisiting once Node 13 (Claiming) exists.**

| Order status when cancelled | Rows inserted |
|---|---|
| `placed` | `manager`, customer |
| `preparing` | `cook`, `manager`, customer |
| `ready` | `waiter`, `manager`, customer |

- **Bell Icon UI:** dropdown panel (not modal/page), number badge for unread count, click-outside-to-close, `lucide-react` added as new dependency, placed left of Sign Out button in Navbar, visible only to `waiter`/`cook`/`manager` roles.

## Authoritative Spec Files (Google Drive — still the source of truth for this chat's artifacts)

- `01_Master_Prompts/Claude_Side/Chat16_Node9_ClaudeSpec_SchemaDesign.md`
- `01_Master_Prompts/Claude_Side/Chat16_Node9_ClaudeSpec_GapResolution_v2.md` (supersedes v1 on `order_cancelled` mapping only)
- `01_Master_Prompts/Claude_Side/Chat16_Node9_ClaudeSpec_BellIconUI.md`
- `02_Instructions/Chat16_Node9_Fix_BuildFailure.md`
- `04_Logs/Chat16_Node9_Step1_Locked.md`

## Known Incident (for awareness, resolved)

Antigravity generated a conflicting duplicate schema spec early in this chat (wrong column names, wrong Owner-access logic) in a different Drive folder. It was trashed. Actual `.sql` migration that got executed was verified correct against the authoritative spec. No repeat of this needed — flagging only so Chat 17 doesn't need to re-verify the migration again (it's already confirmed correct and locked).

## Immediate Next Task for Chat 17

A **new build/runtime error appeared after applying the ESLint-suppression fix** for `NotificationBell.tsx`. Not yet diagnosed — Ayush will paste the new error at the start of Chat 17. Follow the investigation pipeline (OBSERVATION → INVESTIGATION → EVIDENCE → ROOT CAUSE → DECISION → FIX → BUILD/TEST → AYUSH VERIFICATION) per standing rules — don't skip straight to a fix.

## Process Note (this chat)

GitHub Integration connector is visible in Claude.ai settings but its MCP tools were not available to this chat session (scoped to Chat-attach/Projects-sync/Claude-Code use, not general repo write). This master prompt is saved to Google Drive as usual; Ayush will manually instruct Antigravity to push it to `github.com/ayush22cp008/TableFlow_Records` under a `handoff/` folder. Skill file (`general-project-setup`) has not yet been updated to reflect a GitHub-based master-prompt-handoff rule — Ayush said to update it later, not this session.
