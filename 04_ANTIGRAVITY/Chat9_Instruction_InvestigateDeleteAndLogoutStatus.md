# Instruction — Investigate: Delete Confirmation + Manual Logout Status (Follow-up)

**Chat #9 — Investigation only. Do NOT write any fix.**

## Context
Staff Management updates (navbar, Staff Details, force logout, delete) were implemented and are live. Ayush tested two things:

## A. Delete staff — needs re-verification
Ayush deleted `ajza0089@gmail.com` (waiter, "patel") via the Owner UI. He initially believed the DB didn't update. However, a DB screenshot Ayush provided shows `ajza0089@gmail.com` with `is_active = FALSE` in the `profiles` table — which suggests the delete DID work, and the row is correctly excluded from the "Active Staff" UI table (which filters `is_active = true`).

**Investigate:**
1. Confirm current DB state for `ajza0089@gmail.com` — is `is_active` actually `FALSE` right now?
2. Confirm the ban API (`/api/staff/deactivate`) was successfully called for this user — check for any error logs/response issues even if the DB update itself succeeded (i.e. did the `profiles.is_active` update succeed but the `auth.users` ban call silently fail, or did both succeed?).
3. Report clearly: is there an actual bug here, or did the delete work correctly and Ayush was looking at stale/cached data?

## B. Manual logout doesn't flip status to Inactive — likely real bug
Ayush manually logged out a Manager ("AD", `ayushhalpati008@gmail.com`) from that account's own session (not via the End Day button). The Owner's Staff Details table still shows this manager as "Active".

**Investigate:**
1. Report the exact current logic for the "Active"/"Inactive" badge in `app/dashboard/staff/page.tsx` — confirm it's based on `last_login` within 48 hours (per the original design), not on current session/login state.
2. Confirm: does a manual logout (client calling `supabase.auth.signOut()`) update `last_login` or `is_active` in `profiles` at all? (Expected answer: no — the `sync_last_sign_in_at` trigger only fires on sign-IN, not sign-out, per the original migration design.)
3. Clarify the actual current behavior: "Active" badge reflects "logged in within last 48 hours" (a login-recency signal), NOT "currently has an active session right now." Confirm this matches Ayush's expectation or if this is a mismatch between what was built and what he now wants (real-time session status vs. login-recency).

## Output required
Pure findings for both A and B. For A: confirm or reject whether this is a real bug. For B: confirm the current logic and clarify the gap between "last login within 48h" vs "currently logged in" — don't propose a fix yet, just lay out the distinction clearly so Ayush can decide if the badge logic needs to change to reflect real-time session state instead.
