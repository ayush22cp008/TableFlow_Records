# Fix Plan: Deactivation Email + Welcome Email

**Type:** Fix (implementation) — based on `Chat11_Investigation_DeactivationAndWelcomeEmails_Result.md`.

## Part 1 — Deactivation Email

### [MODIFY] `app/api/staff/deactivate/route.ts`
- Right after owner-verification (Step 2) and **strictly before** the profile/auth delete (Step 3):
  - Fetch the staff member's email via `await supabaseAdmin.auth.admin.getUserById(userId)`.
  - Send email via existing Resend pattern (same `from` address and client setup as `app/api/send-invite/route.ts`).
  - Subject: something like "Your TableFlow staff access has been removed."
  - Body: simple, neutral notice — access removed, contact the owner if this is unexpected. No blame/tone issues, just factual.
- Wrap the email send in `try/catch`; on failure, `console.error` and continue — do not block or fail the deactivation.
- Proceed with the existing delete logic (profiles + auth.users) exactly as before, regardless of email outcome.

## Part 2 — Welcome Email

### [MODIFY] `app/api/auth/staff-signup/route.ts`
- At the end of the successful signup block — after the account is created/reactivated, before (or alongside) marking the invite code as used, and before returning `{ success: true, userId }`:
  - Send a welcome email via Resend using the `email` and `role` already available from the request body.
  - Subject: something like "Welcome to TableFlow!"
  - Body: generic congratulations message — restaurant name (if available) + their assigned role. No custom owner-editable guidelines field (explicitly out of scope).
- Wrap in `try/catch`; on failure, `console.error` and continue — signup must succeed regardless of email outcome.

## Verification Plan
### Automated
- `npm run build` to confirm no type/syntax errors.

### Manual (Ayush)
- Deactivate a staff member, confirm they receive the deactivation email at their registered address, and confirm deactivation itself still works even if you temporarily test with a bad/invalid email to see it fail gracefully (optional stress test).
- Invite and complete signup for a new staff member, confirm they receive the welcome email with correct role/restaurant info.
