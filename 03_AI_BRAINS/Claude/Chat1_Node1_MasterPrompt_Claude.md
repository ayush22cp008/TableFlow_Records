# TableFlow — Staff Role System — Master Prompt (Claude Side)

Chat #1 | Node #1

## Project Context

TableFlow (VibeAthon 6.0 project, live at table-flow-nu.vercel.app) is being upgraded post-hackathon for job/portfolio purposes. This feature — Staff Role System — is one upgrade track, developed as a standalone effort separate from the main TableFlow bridge folder.

## Feature Goal

Add role-based staff accounts to TableFlow: Waiter, Cook, Manager, alongside the existing Owner and Customer roles.

## Roles Decided So Far (high-level, not final permissions)

- Waiter — serves orders, sees table status
- Cook — sees kitchen queue, updates food prep status
- Manager — generates bills, collects customer payments
- Owner — full access — analytics, profit tracking, dish-level sales data, staff management (existing role, unchanged)

## Access Model Decision

- Permissions will be granular, enforced at the RLS (Row-Level Security) level — not just hidden UI elements.
- Owner creates/manages staff accounts manually via an admin panel (no self-signup, no invite-code flow for staff — this is different from the existing owner/customer invite-code pattern).

## ⚠️ PENDING — Not Yet Decided

Permission matrix is NOT decided. Exact read/write access per role (e.g., can Waiter edit an order? Can Cook see pricing? Can Manager void a bill?) has not been discussed or locked. Do not assume or infer permissions. Do not write RLS policies or schema until this matrix is explicitly decided in a future session.

## Sequencing Note

This feature work starts after Phase 1 (existing known-bug fixes on main TableFlow: reservation label bug, order cancellation seat-release, reservation arrival notification) is complete.

## Node Map

- ✅ Node 1 (this node): Role scope defined (Waiter/Cook/Manager/Owner), access model decided (granular RLS, owner-managed staff creation), Drive bridge folder set up (TableFlow_Staff_Role_System, standalone from main TableFlow bridge folder — moved to My Drive root).
- ⬜ Node 2 (NOT STARTED): Decide the full permission matrix (per-role read/write access across orders, tables, bills, analytics). Dependency: Node 1 complete.
- ⬜ Node 3 (NOT STARTED): Schema design — staff table, role enum, RLS policy design based on Node 2 matrix. Dependency: Node 2 complete.
- ⬜ Node 4 (NOT STARTED): Owner admin panel (staff add/remove/edit UI). Dependency: Node 3 complete.
- ⬜ Node 5 (NOT STARTED): Staff login flow (owner-created accounts, no self-signup). Dependency: Node 3 complete.
- ⬜ Node 6 (NOT STARTED): Waiter / Cook / Manager dashboards (3 separate views). Dependency: Node 4, Node 5 complete.
- ⬜ Node 7 (NOT STARTED): Order state machine update to reflect waiter-serving / cook-preparing stages. Dependency: Node 6 complete.

## Risk Note

Existing RLS policies and the place_order_and_occupy_table RPC are stable and tested on main TableFlow. This feature should be built on an isolated git branch and fully tested before merging, to avoid destabilizing the working core flow.

## Next Action

Bug-fix phase first (per user decision): fix existing known bugs on main TableFlow before resuming Staff Role System planning. When ready to resume this feature: start at Node 2 — decide the permission matrix together before any code or schema work begins.
