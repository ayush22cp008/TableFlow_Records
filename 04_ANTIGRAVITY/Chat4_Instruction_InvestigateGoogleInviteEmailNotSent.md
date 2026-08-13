# TableFlow — Chat #4 — Instruction (Investigation Only): Invite Email Not Sent via Google Auth Path

**To:** Antigravity
**From:** Claude
**Type:** Investigation — NO code changes, NO fixes in this prompt

**SUPERSEDES:** `Chat4_Instruction_InvestigateResendInviteEmail.md` (that file was too generic — this replaces it with the precise mechanism confirmed by Ayush. Treat this file as the correct one; the older file can be ignored.)

---

## Confirmed Mechanism (from Ayush, verified)

1. Owner creates an invite code tied to a **specific staff email address** (Owner enters the email when generating the invite).
2. When that exact email later goes through signup, the system should **auto-detect** the pending invite match and **automatically email the invite code** to that address.
3. **This works correctly via `/signup`** (non-Google, "Staff Email" field path) — confirmed working, invite code email arrives.
4. **This does NOT work via `/auth/select-role`** (Google auth path) — even when the Google account's email matches a pending invite, no email is sent.

## Why This Is Likely the Same Root Cause as the Earlier Google Role-Selection Bug

Per the earlier investigation (`Chat4_Investigation_GoogleStaffSignup.md`), `/auth/select-role` has broken invite-code handling because:
- Google auth auto-creates a `profiles` row with role `customer` BEFORE this screen loads
- The invite-code validation/consumption step then fails because `customer` role lacks permission (RLS requires `owner` role to mark invite codes as used)

**Hypothesis to verify:** the email-send trigger is likely wired into the SAME invite-code-matching/validation code path that's already confirmed broken on `/auth/select-role`. If that validation step never successfully completes on this path, the downstream "send email" step never fires either — same root cause, second visible symptom.

## What to Investigate

1. Find the exact code path where "email matches a pending invite → send invite code email" logic lives. Confirm: is this logic shared between `/signup` and `/auth/select-role`, or duplicated/diverged between the two?
2. Trace whether this email-trigger code is ever reached at all on the `/auth/select-role` path — i.e., does execution even get to the point where Resend would be called, or does it fail/return earlier (e.g., blocked by the same RLS issue already found)?
3. Confirm in Resend's dashboard logs: are there ANY attempted sends correlating with Google-auth signups, or zero attempts at all? (Zero attempts = code never reaches the send step, confirming the shared-root-cause hypothesis. Attempted-but-failed = a different, separate issue.)
4. If it turns out this is NOT connected to the already-found bug (i.e., Resend shows failed/bounced attempts, not zero attempts), then investigate as originally scoped: Resend domain/DKIM status, `RESEND_API_KEY` in production, and recipient address correctness.

## Do NOT

- Do not modify any code, RLS policies, or Resend configuration yet
- Do not fix the already-known `/auth/select-role` role-assignment bug in this prompt — that's a separate, already-scoped fix (queued next)
- Do not resend test invites repeatedly — one or two test sends is enough for diagnosis

## Output

Save findings to: `03_Investigation_and_Errors/Chat4_Investigation_GoogleInviteEmailNotSent.md`

Include: whether this is confirmed to be the same root cause as the select-role role-assignment bug (zero send attempts) or a separate issue (attempted-but-failed sends), plus supporting evidence (code trace + Resend logs).

Report back to Claude with the file path once done.
