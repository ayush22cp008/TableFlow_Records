# TableFlow — Staff Role System — Master Prompt (Claude-Side)

**Chat #2 → Chat #3 Handoff**

## Node-Map

- ✅ **LOCKED — Node 1: Permission Matrix** (full role/resource access matrix, order status lifecycle, customer-side table board design). See `Chat1_Node1_MasterPrompt_Claude_PermissionMatrix.md`.
- ✅ **LOCKED — Node 2a: Staff Onboarding Design** (invite code system — this file's main content below)
- 🔄 **ACTIVE — Node 2b: Schema + RLS Implementation** — not yet started, next chat begins here
- ⬜ **NOT STARTED — Node 3: Bug fixes** — depends on Node 2b shipping first; existing unknown bugs deliberately deferred until after RLS implementation, per evidence rule (no speculative fixing)

## Node 2a — Locked Design: Staff Invite Code System

**Signup flow:**
1. Existing signup role panel extended with 3 new options: Manager, Waiter, Cook (alongside existing Customer/Owner)
2. Selecting a staff role reveals an invite code field
3. User enters code + sets their own email + password (self-service, no temp password, no forced-change flow — rejected earlier as unnecessary)

**Three-layer validation, ALL must pass to redeem a code:**
1. **Code validity** — exists, unused, not expired
2. **Role match** — role embedded in code (WT/CO/MN tag) = role selected on panel
3. **Identity match** — signup email = `staff_email` Owner registered against that code

Any single failure → reject, no account created, code stays unused (Owner can correct and retry).

**Invite code format:**
- 7 characters total
- Contains 2-letter role tag (`WT`=Waiter, `CO`=Cook, `MN`=Manager) at a **random position** (not fixed prefix) — remaining 5 chars random alphanumeric
- Redemption logic must scan full string for the role tag, not assume fixed position

**`invite_codes` table (new):**
- `code` (7-char, unique)
- `role` (waiter/cook/manager)
- `staff_name` — Owner enters at generation, **editable while code is unused** (emergency correction, e.g. typo)
- `staff_email` — Owner enters at generation, **editable while code is unused**
- `used` (boolean)
- `created_by`, `created_at`
- **30-min expiry, auto-delete** — reuse the SAME existing mechanism already used for reservation-request expiry (Antigravity to confirm exact current implementation during investigation — do not build a new/parallel expiry mechanism)

**Owner controls:**
- Generate invite code (select role + enter name + email)
- Edit name/email on any still-unused, unexpired code
- Multiple active codes per role can coexist simultaneously (e.g. hiring 2 waiters same week)
- Owner can regenerate their own personal code anytime (separate from staff codes)

## Node 2b — Schema/RLS Scope (next chat starts here)

1. Extend `profiles.role` CHECK constraint: add `manager`, `waiter`, `cook` (currently only `customer`, `owner`)
2. Create `invite_codes` table per spec above
3. Build invite code generation logic (Owner-only)
4. Build invite code redemption logic (3-layer validation on signup)
5. Extend existing role panel UI + conditional invite code field
6. **RLS migration — single all-at-once file**, replacing current `is_owner()`-only gate with full role-based policies matching `Chat1_Node1_MasterPrompt_Claude_PermissionMatrix.md`. Tables affected: orders, order_items, restaurant_tables, waitlist, menu_items, and any others currently gated by `is_owner()`.

## Key facts from Node 2 investigation (already done — do not re-investigate)

- Table: `profiles`, column: `role` (text, CHECK constraint), currently `('customer','owner')` only
- Central helper `is_owner()` used almost everywhere in RLS — needs equivalent per-role helpers or a general `has_role()` function
- 0 rows currently in `profiles` — no migration/backfill risk
- Full investigation report: `03_Investigation_and_Errors/Chat2_Investigation_StaffRoleSchema_Result.md`

## Rejected alternatives (do not re-propose)

- Separate `staff` table instead of extending `profiles` — rejected, single-restaurant deployment doesn't need it
- System-generated temp password + forced password-change on first login — rejected, replaced by self-service invite code signup
- Physical-presence-based onboarding — rejected, Owner and staff are often remote; WhatsApp-delivered invite code is the actual workflow

## Next Action for Chat #3

Start Node 2b. First step: Antigravity investigation of the existing reservation-code expiry mechanism (exact table/trigger/cleanup logic) so the invite code expiry reuses it correctly — investigation only, no fix, per engineering discipline rule.
