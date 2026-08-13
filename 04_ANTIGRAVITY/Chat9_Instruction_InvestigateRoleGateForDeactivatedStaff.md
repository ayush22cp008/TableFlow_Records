# Instruction — Investigate: Role-Gate Enforcement for Deactivated Staff (Pre-fix Check)

**Chat #9 — Investigation only. Do NOT write any fix.**

## Context
Current delete/deactivate flow permanently bans the user's email via Supabase Admin API (`ban_duration: '876000h'`). Ayush flagged this is too broad — a banned staff member can never log in again even as a customer, and the Owner can never re-recruit them as staff later. Decision: remove the permanent ban, rely on `is_active = false` (role-based block) instead.

## Investigate (before removing the ban)

1. **Route protection**: For each staff dashboard (`/dashboard/waiter`, `/dashboard/cook`, `/dashboard/manager`, and any others), report whether there's currently a check that verifies `profiles.is_active = true` (or equivalent) before granting access — or does it only check `profiles.role` without checking `is_active`?

2. **Behavior if ban is removed today, unchanged otherwise**: if a deactivated staff member (`is_active = false`) logs in right now with their existing role still set (e.g. `role = 'waiter'`), would they currently be able to reach `/dashboard/waiter` and use it normally? Report the exact current behavior.

3. **Re-recruitment flow**: if the Owner wants to re-invite a previously deactivated staff member, report what currently happens when a NEW invite code is generated for an email that already exists in `profiles` with `is_active = false` — does the invite/signup flow update the existing profile (`is_active = true`, role reassigned), or would it error/conflict on a duplicate email?

4. **Customer access**: if the ban is removed, confirm a deactivated staff email can sign up/log in as a customer without any residual staff-role artifacts blocking them (e.g. does anything check `profiles.role` in a way that would incorrectly treat them as still 'waiter'/'cook'/'manager' just because the row exists with `is_active=false`?).

## Output required
Pure findings — confirm whether route-level `is_active` checks already exist or are missing, and report the re-invite/re-signup behavior for an existing deactivated profile. No fix. Report back before any fix is proposed.
