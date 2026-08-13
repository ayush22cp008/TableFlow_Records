# Chat #3 | Node 2b — Decision: Invite Code Delivery Redesign (LOCKED, SUPERSEDES prior manual-share design)

## Change summary
Original Node 2a design assumed Owner manually shares the invite code with staff (WhatsApp/verbally) after generating it, and staff then enters that code alongside an OTP-verified email during signup. This is now REPLACED with an email-triggered delivery flow — no manual sharing, no separate OTP step for staff signup.

## New Flow

1. **Owner — Staff Management panel:** Owner enters staff name + email + role, generates invite code. Code is saved to DB (invite_codes table). **No email is sent at this point.**

2. **Staff signup — Step 1:** Staff selects their role (Waiter / Cook / Manager).

3. **Staff signup — Step 2:** Staff enters their own email.
   - System checks DB for a pending (unused, not expired) invite_codes row matching this email + role.
   - **Match found** -> system sends the invite code to that email address.
   - **No match** -> signup rejected with an error (no pending invite for this email/role combination).

4. **Staff signup — Step 3:** Staff checks their email, enters the received code into the signup form (along with setting a password).

5. **Final validation — same 3-layer check as originally locked in Node 2a, unchanged:**
   - **Code valid** — exists, unused, not expired.
   - **Role match** — role selected at signup = role embedded in the invite_codes row.
   - **Email match** — signup email = invite_codes row's registered email (this is naturally satisfied since the code was only emailed after this match was found in Step 2, but the check remains as a safety net / defense in depth).

## What's removed
- Owner manually sharing the code via WhatsApp/verbally — no longer part of the flow.
- The separate Supabase OTP step for staff signup — the invite code delivered via email now serves as the email-ownership proof, replacing OTP's role for this signup path. (Owner/Customer signup OTP flow is UNCHANGED — this only affects Waiter/Cook/Manager signup.)

## Implementation implications
- Need an email-sending mechanism for invite code delivery (Resend, already used for tableflow.systems domain, is the natural choice — to be confirmed with Antigravity during implementation).
- Signup flow's step ordering changes: role -> email (match lookup + send) -> code entry, rather than the previous role -> email+code+OTP combined step.
- The 3-layer validation logic itself (code/role/email matching) does NOT change — only the point at which the code is revealed to the user changes (email delivery, not manual sharing).

## Next
Antigravity instruction to follow: redesign the staff signup step sequence in AuthForm.tsx (and related invite lookup logic) to match this flow, remove OTP step for staff path only, wire up email delivery.
