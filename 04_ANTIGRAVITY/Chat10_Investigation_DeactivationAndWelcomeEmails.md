# Investigation: Deactivation Email + Welcome Email on Staff Join

**Type:** Investigation only — no code changes.

## Context
Two new email notifications are needed:
1. When a staff member is deactivated (full hard delete), send them an email informing them their staff access has been removed. (Note: NOT triggered by "End Day — Log Out All Staff," which is a routine session logout, not an account removal.)
2. When a staff member successfully completes signup (joins via invite code), send them a simple welcome/congratulations email. Generic message only — no restaurant-specific custom guidelines (explicitly out of scope for now).

## Investigate

1. **Existing email infrastructure:**
   - Confirm how the invite-code email is currently sent (which service — Resend, per memory — which file/route triggers it, what the email template pattern looks like).
   - Confirm this same service/pattern can be reused for both new emails without new setup.

2. **Deactivation email — where to hook in:**
   - Look at `app/api/staff/deactivate/route.ts` (the hard-delete flow). Confirm the staff member's email is available at the point of deletion (needs to be captured BEFORE the delete, since after `deleteUser()` the record won't exist to look up the email from).
   - Confirm order of operations: email should be captured/sent before or immediately as part of the delete, not after — since after deletion there's no record left to pull the email from.

3. **Welcome email — where to hook in:**
   - Look at `app/api/auth/staff-signup/route.ts`. Confirm the exact point where signup is confirmed successful (after profile creation, after auto-login succeeds) — this is where the welcome email trigger should go.
   - Confirm what data is available at that point for the email (staff name, role, restaurant name if applicable).

4. **Failure handling:**
   - For both cases, confirm what should happen if the email fails to send — should it block/fail the main action (deactivation or signup), or should the main action succeed regardless and the email failure just get logged? (Recommend: main action should never be blocked by an email failure — deactivation/signup succeeding is more important than the email.)

## Output
Report findings only — reusable email pattern, exact hook-in points for both emails, and the recommended failure-handling approach. No fix/plan yet.
