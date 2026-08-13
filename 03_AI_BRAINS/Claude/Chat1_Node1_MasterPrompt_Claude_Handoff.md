# TableFlow — Staff Role System — Master Prompt (Claude Side)

Chat #1 | Handoff Point — session ending due to context limit

## Project Context

Post-hackathon upgrade track for TableFlow (VibeAthon 6.0, live at table-flow-nu.vercel.app). Testing convention: no local testing — always commit+push to GitHub → Vercel auto-deploys → Ayush tests live.

## What Actually Happened This Session (deviation from original plan)

Original Node 1 plan was to design the Staff Role System (Waiter/Cook/Manager roles). Before that could start, Ayush wanted existing known bugs fixed first. This entire session went into that bug-fix phase — the Staff Role System itself has NOT been started yet.

## Node Map

- ✅ **LOCKED — Reservation Code Flow Fix (Track A):** Removed "Confirm Arrival" step entirely. Reservation code now validates directly against `status='approved'` + `requested_time` + 30-min grace window (no `arrived` intermediate state). Fixes the original "reservation label stuck" bug and the orphaned-reservation bug. Merged to `main`, deployed, verified live (grace-window-valid case + 40-min-backdated-expiry case both tested and passing).
- ✅ **LOCKED — Auto-Hide Reservation Requests:** Approved reservation requests auto-hide from `/dashboard/tables` panel 5 min after bill generation (if order completed), or at grace-window expiry (if no-show, no order placed). Merged to `main`, deployed, verified live (both cases tested and passing).
- ✅ **LOCKED — Track B: Reservation Priority + Order Numbering:** Orders placed via reservation code get `is_priority=true`, tagged "PRIORITY" badge, sequential number `R1, R2...`. Walk-in orders get separate sequence `W1, W2...`. Both reset daily. Priority orders always sort above walk-in orders in every column of `/dashboard/orders` (Order Placed/Preparing/Ready/Served) — verified live with screenshots showing R1 consistently above W1 across multiple columns/stages. New `daily_order_counters` table handles atomic sequence generation (race-condition safe). Merged to `main`, deployed.
- ⬜ **NOT STARTED — Staff Role System (original Node 1 goal):** Add Waiter/Cook/Manager roles alongside existing Owner/Customer. Access model already decided pre-session: granular RLS enforcement, owner manually creates/manages staff (no self-signup/invite-code for staff). **Permission matrix still NOT decided** — do not assume or infer permissions; must be discussed explicitly before any schema/RLS work. Dependency: none blocking — can start anytime.

## Known Minor Cleanup (not urgent, noted for later)

- Some untracked test/investigation scripts got incidentally included in the Track B merge commit (`3224882`). Harmless (don't affect deployment) but could be cleaned up later per file-creation-discipline.

## Next Action

Resume at the **Staff Role System** feature (original Node 1 intent) — start by deciding the full permission matrix (per-role read/write access across orders, tables, bills, analytics) before any code or schema work.

## Reference Files in This Session (for context if needed)

All in `03_Investigation_and_Errors/`: investigation reports, fix specs, and merge instructions for Track A, Auto-Hide fix, and Track B — kept as historical record, not needed to re-read unless debugging a regression in this already-locked work.
