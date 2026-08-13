# TableFlow — Chat #4 — Master Prompt (Claude Side): Handoff to Chat #5

**Status:** Chat #4 closing, handing off to Chat #5
**Next chat focus:** Manager Dashboard (Node 3, third of three)

---

## Node-Map

| Node | Status | Notes |
|---|---|---|
| Node 1 — Core Staff Role System | ✅ LOCKED | Permission Matrix, RBAC, lifecycle — established Chat #1 |
| Node 2b Part 1 — Schema/RLS/Cancellation | ✅ LOCKED | Migrations applied + verified live, Chat #3 |
| Node 2b Part 2 — Invite Email (Resend, non-Google) | ✅ LOCKED | Verified end-to-end, Chat #3/#4 |
| Node 3 — Routing architecture decision | ✅ LOCKED | Role-specific routes confirmed via investigation (`Chat4_Node3_Investigation_RoutingArchitecture.md`) |
| Node 3 — Cook Dashboard | ✅ LOCKED | Built, tested live, verified working (routing, KDS view, Mark Ready, cancellation) |
| Node 3 — Google Auth Staff Signup + Invite Email Bug | ✅ LOCKED | Root cause: `/auth/select-role` was an incomplete page (no email trigger, no server-side invite validation). Fixed via new `verify-invite` API route + wired-up email trigger. Verified live with real Gmail test — email received, code marked `used`. |
| Node 3 — Waiter Dashboard | ⬜ NOT STARTED | Depends on: routing decision (done). Same pattern as Cook — reuse existing Kanban/query pattern, scope to Waiter's permission row (Ready→Served transition, tables/reservations access). |
| Node 3 — Manager Dashboard | 🔄 ACTIVE (Chat #5 focus) | Depends on: routing decision (done). Manager has the widest permission scope — intake (Placed→Preparing), billing, full R/W on orders/tables/menu. Needs its own spec session before building, same as Cook did. |
| Node 4 — Notification System (polling) | ⬜ NOT STARTED | Deferred. Auto-refresh will be added uniformly across all dashboards (Cook/Waiter/Manager) once this node starts — deliberately excluded from Cook to keep concerns separate. |

## Parked Issue (deliberately deferred — not forgotten)

**`ayushhalpati09@gmail.com` — stuck as Owner, cannot be deleted.**

- This is the original test account that surfaced the Google-auth bug. It still exists with Owner role and Ayush has been unable to delete it via Supabase Dashboard.
- Separately, a real schema bug was found and FIXED: `profiles.id → auth.users(id)` was missing `ON DELETE CASCADE` (live DB drifted from migration file), and `invite_codes.created_by → profiles(id)` had no cascade rule at all. A fix instruction (`Chat4_Instruction_FixUserDeleteCascade.md`) was written and given to Antigravity — **status of whether this specific fix was applied/verified is NOT confirmed in this chat's evidence trail. Confirm in Chat #5 whether this migration was actually applied live before assuming the delete will now work.**
- Ayush has explicitly chosen to fix this account later, not now. Do not block Manager Dashboard work on this.

## Decisions Locked This Chat (#4)

1. Dashboard routing: role-specific routes (`/dashboard/cook`, `/dashboard/waiter`, `/dashboard/manager`), not a shared dashboard with conditional rendering — confirmed via code investigation, not assumption.
2. Cook dashboard ships with manual refresh only; auto-refresh deferred to Node 4 uniformly.
3. Order status transitions stay whole-order (no per-item partial-ready), matching existing lifecycle precedent.
4. `invite_codes.created_by` uses `ON DELETE SET NULL` (not CASCADE) — preserves invite history if the creator account is later deleted.
5. Google-auth signup fix scope was strictly limited to staff roles (Cook/Waiter/Manager) — Customer and Owner Google flows were explicitly left untouched and confirmed still correct as a regression check.

## Evidence (this chat)

- Cook dashboard: 7 screenshots showing full loop — Owner board → Cook KDS → Mark Ready → Owner board reflects Ready status. Cancellation (single + bulk) also manually confirmed by Ayush.
- Google auth fix: Staff Management table showing invite code `JRBC07D` status changed `unused → used`, plus live Gmail screenshot showing the invite email correctly received with role `cook` stated.

## Next Task (Chat #5 opening move)

Start Manager Dashboard spec session — same process as Cook: pull Manager's exact Permission Matrix row first, ask clarifying questions on any granularity decisions (e.g. billing flow specifics, bulk actions scope), THEN write Claude-side master prompt + Antigravity-side instruction as separate approved files.
