Chat #3 | Node 2b | Instruction — Implement Invite Code Email-Delivery Redesign

Per locked decision: Chat3_Node2b_Decision_InviteCodeEmailDeliveryRedesign.md

## Scope
Redesign the Waiter/Cook/Manager staff signup flow in components/AuthForm.tsx. This does NOT affect Owner/Customer signup (their existing OTP flow stays untouched).

## New staff signup sequence
1. Staff selects role (Waiter/Cook/Manager) — Step 1, unchanged.
2. Staff enters email — Step 2 (NEW: no code input yet, no OTP).
   - Query invite_codes for a row matching this email + role, where status = 'unused' and not expired.
   - No match -> show error, stop here (no account created, nothing sent).
   - Match found -> send an email to that address containing the invite code, using Resend (reuse existing Resend setup for tableflow.systems domain — check how it's configured for the reservation/OTP-adjacent emails already in the codebase and match that pattern).
3. Staff enters the code they received via email + sets a password — Step 3.
4. On submit, run the full 3-layer validation (code valid/unused/not expired, role match, email match) as it already exists — do not change this validation logic, only where it sits in the sequence.
5. On success: create account (profiles row with correct role), mark invite_codes row as 'used'.

## What to remove
- The current OTP step (supabase.auth.verifyOtp) for this staff path only. Owner/Customer OTP flow must remain fully intact — do not touch that path.
- Any remaining reference to manual code-sharing assumptions in comments/UI copy for this flow.

## Investigate first (report before building)
- How is Resend currently invoked in the codebase (API route, library call, template pattern)? Reuse that exact mechanism rather than building a new one.
- Confirm the invite_codes schema supports this lookup (email + role + status columns, as already built in Node 2b).

## Evidence required
Build must pass. Report exact files changed, and the new step sequence implemented, before this gets merged. Do not touch Owner/Customer signup path, RLS policies, or the cancellation system — those are out of scope for this task.
