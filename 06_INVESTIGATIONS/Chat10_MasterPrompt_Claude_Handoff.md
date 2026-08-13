# TableFlow Staff Role System — Chat 10 Master Prompt (Claude-Side Handoff)

**Handoff from Chat #9. Continue from here.**

## Node Map

| Node | Status |
|---|---|
| 1 — Permission Matrix | ✅ LOCKED |
| 2b — Schema/RLS/invites/cancellation/email | ✅ LOCKED |
| Routing | ✅ LOCKED |
| Cook Dashboard | ✅ LOCKED |
| Manager Dashboard | ✅ LOCKED |
| Waiter Dashboard | ✅ LOCKED |
| **Node 4 — Reservation double-booking prevention** | ✅ LOCKED — verified + pushed (commit `22d848d`) |
| Reservation Requests panel clutter fix | ✅ DONE — verified + pushed (same push as Node 4 area) |
| **Owner Side: Staff Management (navbar, Staff Details, force logout, delete)** | ✅ VERIFIED by Ayush (full manual test passed) — **NOT YET PUSHED** |
| **Owner Side: Remove permanent ban, proper role-gate on deactivation** | 🔄 ACTIVE — instruction written, not yet sent to Antigravity |
| Notifications | ⬜ NOT STARTED |

## Immediate next task (start of Chat #10)

Send this instruction to Antigravity (already written, sitting in `02_Instructions/`, not yet executed):

```
G:\My Drive\TableFlow_Staff_Role_System\02_Instructions\Chat9_Instruction_FixRemoveBanProperRoleGate.md
```

**What it does:** Removes the permanent Auth ban (`ban_duration: '876000h'`) on staff deactivation — Ayush correctly identified this was too broad (blocks the person from ever being a customer or being re-recruited as staff). Replaces it with:
1. `is_active = false` + `role` downgraded to `customer` on deactivation
2. `middleware.ts` + DB `has_role()` also check `is_active`, not just `role`
3. Re-invite flow fixed so a previously-deactivated email can be re-invited to any role without crashing at signup

Investigation already confirmed all 3 gaps are real (see `Chat9_Investigation_RoleGateForDeactivatedStaff_Result.md` in this same folder). No further investigation needed — go straight to sending the fix instruction to Antigravity.

## Pending items after that fix is verified

1. **GitHub push** — TWO things are sitting un-pushed right now:
   - Owner Side Staff Management (navbar + Staff Details + force logout + delete) — fully verified by Ayush, ready to push
   - The upcoming ban-removal/role-gate fix (once Antigravity implements + Ayush verifies)
   - Recommend pushing both together once the ban fix is also verified, to avoid two separate deploy cycles — confirm with Ayush.

2. **Notifications node** — not started yet, next major node after Staff Management wraps up.

## Standing context (unchanged from Chat #9)
- Drive bridge folder: `TableFlow_Staff_Role_System`, subfolders `01_Master_Prompts/`, `02_Instructions/`, `03_Investigation_and_Errors/`.
- `02_Instructions` folder ID: `13NcntSWMoqGG105X-wp8Mfu8KpUlASd8`
- `Claude_Side` master prompts folder ID: `1iWwBytbROljqLIE0ZkTEZD0L9mzXmTFQ`
- Supabase CLI unavailable on Ayush's machine — all migrations applied manually via SQL Editor, always report this to Ayush as a pending manual step after any fix involving a migration.
- Investigation and fix are always separate instructions/prompts.
- No GitHub push without Ayush's explicit go-ahead and manual verification.
