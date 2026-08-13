# TableFlow — Chat #4 — Instruction (Investigation Only): Resend Invite Email Not Arriving

**To:** Antigravity
**From:** Claude
**Type:** Investigation — NO code changes, NO fixes in this prompt

---

## Bug Report

Staff invite code emails (sent via Resend, custom domain `tableflow.systems`, built in Node 2b Part 2) are not being received by staff. This is a regression or gap in an already-built and previously-verified feature — Node 2b Part 2 was marked locked/verified end-to-end at last session handoff, so something has changed or the earlier verification missed a case.

## Context (from project history)

- Email delivery via Resend SDK, custom domain `tableflow.systems`
- This flow was built to replace manual invite code sharing
- Previously verified "end-to-end live" per Chat #4 handoff notes — needs re-confirmation of what exactly was tested then vs. what's failing now

## What to Investigate

1. **Resend dashboard/logs** — check Resend's own delivery logs (not just app code) for the specific invite emails that should have been sent. Look for: were they sent at all? Did Resend report success, bounce, or failure? Common failure reasons to check: domain verification status (SPF/DKIM/DMARC for `tableflow.systems`), rate limits, sender reputation, or the recipient address being invalid/typo'd.
2. **App-side code path** — find the code that triggers the invite email send (likely in the staff invite creation flow, wherever `invite_codes` rows get created). Confirm:
   - Is the Resend API actually being called? (Check for silent failures — e.g. API call wrapped in try/catch that swallows errors without logging or surfacing them)
   - Is the correct recipient email being passed in? (This matters given the earlier finding that `/auth/select-role` and `/signup` are two different code paths — confirm which path is used for creating invites and whether it's the same one that was verified before)
   - Any recent code changes (check git log/blame) to the email-sending function or its environment variables since the last verified test
3. **Environment variables** — confirm `RESEND_API_KEY` (or equivalent) is still correctly set in the live Vercel environment, not just locally — a key could be valid locally but missing/expired in production.
4. **Spam/deliverability check** — if Resend logs show the email as "delivered," check whether it's landing in spam (ask Ayush to check spam folder for the test recipient) rather than assuming it's a send-side failure.

## Do NOT

- Do not modify any Resend configuration, environment variables, or code yet
- Do not resend test invites repeatedly (Resend may have rate limits — one or two test sends is enough for diagnosis)

## Output

Save findings to: `03_Investigation_and_Errors/Chat4_Investigation_ResendInviteEmailNotArriving.md`

Include: Resend dashboard status for recent send attempts, whether the API call is succeeding or failing app-side, current env var status in production, and a clear determination of whether this is a send-side failure, a deliverability/spam issue, or a code regression.

Report back to Claude with the file path once done.
