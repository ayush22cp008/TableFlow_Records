# TableFlow Staff Role System — Chat #9 — Master Prompt (Claude Side Handoff)

**From:** Chat #8
**To:** Chat #9
**Status at handoff:** Node 3 LOCKED. Node 4 starting fresh.

---

## Node Map

| Node | Description | Status |
|---|---|---|
| 1 | Permission Matrix | ✅ LOCKED |
| 2b | Schema, RLS, invite codes, cancellation, email delivery | ✅ LOCKED |
| Routing | Role-based auth redirects | ✅ LOCKED |
| Cook Dashboard | KDS | ✅ LOCKED |
| Manager Dashboard | Intake/Billing queues, priority sort, W#/R# labels, Mark Paid table release | ✅ LOCKED |
| Waiter Dashboard | — | ✅ LOCKED |
| **4** | **Reservation double-booking prevention** | 🔄 ACTIVE (starting point for Chat #9) |
| Notifications | — | ⬜ NOT STARTED |

## Node 3 — Locked Summary (context only, do not re-touch)

Three bugs fixed and verified in Chat #8, all pushed to GitHub:

1. **Priority sort** — Manager's Intake/Billing queues were missing `is_priority` sort clause. Fixed by matching Owner/Cook's `.order('is_priority', {ascending:false}).order('created_at',{ascending:true})` pattern.
2. **W#/R# label retrofit** — Manager was showing raw UUID fragments instead of clean sequential labels. Fixed via shared `formatOrderNumber()` util wired into Manager's queue rendering.
3. **Mark Paid → table release (two parts)**
   - 3a: New `mark_order_paid` RPC added — decrements `occupied_seats`, flips table to `available` only when fully released (mirrors proven `cancel_active_orders` pattern). Handles multi-order tables.
   - 3b: RPC also now conditionally clears `reserved_from` (only when table fully releases), fixing reservation tables staying visually "Reserved" after payment.
   - **Noted, not fixed (future cleanup, not blocking):** Owner's Generate Bill flow unconditionally clears `reserved_from` regardless of other orders on the table — same latent bug as 3a/3b, just not yet fixed on Owner's side.

## Node 4 — New Issue to Investigate (starting point)

**Reservation double-booking.** Two separate Reservation Requests were approved for the same table with overlapping time windows — e.g. Table 1 (Seats 2): "Ak" party approved for 3:55 PM and "Ar" party approved for 3:45 PM, both on the same table, both showing in Reservation Requests as approved/seated. Side effect observed: table's seated count went to `4/2` (exceeded capacity).

**Root issue (hypothesis, not yet confirmed):** The reservation Approve flow doesn't check for existing active reservations or current occupancy on the target table before allowing approval — no overlap/capacity validation.

**Next action:** Write and run an investigation-only prompt for Antigravity to:
- Locate the reservation Approve handler (likely on the Owner or Manager Tables page, wherever "Reservation Requests" list with Approve/Reject buttons lives)
- Confirm whether any table-availability/overlap check exists before approval
- Confirm table occupancy schema interaction (`occupied_seats`, `reserved_from`, `seats` capacity) at approval time
- Report back before any fix is written (investigation and fix stay separate prompts, per standing rule)

## Standing Workflow Rules (unchanged, see general-project-setup skill for full detail)

- Claude = architecture/diagnosis/instructions. Antigravity = execution only (file edits, terminal/build verification, no manual UI testing). User = manual browser testing + final decisions.
- Investigation and fix are always separate prompts.
- File naming: `Chat9_Node4_{Type}_{ShortDescription}.md`, saved to `02_Instructions/` (instructions) or read from `03_Investigation_and_Errors/` (investigation results).
- GitHub push only on Ayush's explicit go-ahead, after manual verification — pushes can be batched if Ayush requests.
- Master prompts are never pasted in chat — always created as files in the Drive bridge folder.

## Drive Bridge Reference

- Root: `16Xhn59dqZg2_XVMnqulmLeywO3CfhzyL`
- `02_Instructions`: `13NcntSWMoqGG105X-wp8Mfu8KpUlASd8`
- `Claude_Side` master prompts: `1iWwBytbROljqLIE0ZkTEZD0L9mzXmTFQ`
