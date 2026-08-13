# Instruction — Investigate: Navbar Structure + Staff Data Schema (Owner Side, New Node)

**Chat #9 — New topic (Owner-side updates). Investigation only. Do NOT write any fix.**

## Context
Ayush wants 3 updates on the Owner side:
1. Add "Staff Management" link to the main navbar (currently navbar has: Overview, Menu, Tables, Orders, Analytics, AI Insights — Staff Management is only accessible via a dashboard card, not the navbar).
2. Add the same navbar to the Staff Management page (`/dashboard/staff`) — currently this page has no navbar at all, unlike Menu (`/dashboard/menu`) which has the full navbar at the top.
3. Add a "Staff Details" section/tab within the existing Staff Management page, showing basic info per staff member: name, email, role, join date/status.

## Investigate

1. **Navbar component**: locate the shared navbar component (used on Overview/Menu/Tables/Orders/Analytics/AI Insights). Report the file path and how it's structured (is it a shared layout component, or duplicated per page?). Confirm why `/dashboard/staff` doesn't currently include it — is it missing the layout wrapper entirely, or intentionally excluded?

2. **Staff data schema**: inspect the current staff-related table(s) in Supabase (likely linked to the invite code system shown in `Chat9` — code, role, name, email, status columns visible in the "Invite Codes History" table). Report:
   - Exact table name(s) and full column list
   - Does a `join date` / `created_at` / `accepted_at` field already exist that reflects when a staff member actually joined (vs when the invite code was generated)?
   - Does "status" in the current table mean invite-code status (`used`/`unused`) or staff-account status (`active`/`inactive`)? Report both if separate.
   - Is there a separate `staff`/`users`/`profiles` table (distinct from `invite_codes`) that represents actual staff accounts once they've signed up? If yes, report its schema too.

3. Report whether the data needed for "Staff Details" (name, email, role, join date, status) can be sourced entirely from existing tables, or if anything is missing/needs to be added.

## Output required
Pure findings — file paths, component structure, exact schema/columns, and a clear statement of what data is available vs missing for the Staff Details feature. No fix, no code changes. Report back before any fix is proposed.
