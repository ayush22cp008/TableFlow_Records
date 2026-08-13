# Investigation: Two Email Bugs (Deactivation/Welcome Email Feature)

**Context:** Chat 11 — manual testing after build (`Chat11_Evidence_BuildResult_DeactivationAndWelcomeEmails.md`) surfaced two bugs. Investigation only — do NOT fix yet.

## Evidence timeline (test staff: ayushhalpati.2004@gmail.com)

| Time | Event | Invite code | Role |
|---|---|---|---|
| 16:40 | Invite email sent | `CKDC0MN` | Manager |
| ~16:40–16:43 | Staff signup completed (code marked "used" in DB) | `CKDC0MN` | — |
| — | **No "Congratulations/Welcome" email received for this signup** | — | — |
| 16:43 | Owner deactivated staff → "Access Removed" email sent correctly | — | — |
| 16:46 | Invite email sent **TWICE** (duplicate, same code, same minute) | `PWT66DS` | Waiter |
| ~16:46–16:47 | Staff signup completed (code marked "used" in DB) | `PWT66DS` | — |
| 16:47 | "Congratulations/Welcome" email received correctly | — | — |

Invite Codes History confirms both `CKDC0MN` and `PWT66DS` as status = "used" — both signups succeeded at the DB level regardless of email outcome.

## Bug 1: Duplicate invite-code email

- Same invite code, same email address, sent twice within the same minute (16:46).
- Investigate: is `send-invite`/equivalent route being called twice from the client (double-submit / no debounce / React double-invoke), or is the email-sending logic itself looping/retrying inside the route?
- Check: any client-side form double-submit protection missing on "Invite Staff" action, and any retry logic in the Resend call.

## Bug 2: Welcome/congrats email inconsistent — worked on 2nd signup, not 1st

- Both signups (Manager role via `CKDC0MN`, Waiter role via `PWT66DS`) followed the same `staff-signup` code path and both succeeded (DB confirms "used").
- Welcome email fired for the 2nd signup but NOT the 1st, despite identical flow.
- Investigate: any state/condition in the welcome-email trigger logic that depends on prior staff history for that email (e.g. checking if a profile previously existed, or an existing-user branch behaving differently for a fresh account vs a "previously deactivated then re-invited" account). Since deactivation now does a **hard delete** (Node: Deactivate → Full Hard Delete), the DB should treat both signups as fresh/new users — confirm this assumption holds in the actual signup code path, don't assume.
- Check for silent failures: is the welcome-email call wrapped in try/catch that swallows errors without logging? If so, 1st signup may have failed silently — need logs/error output as evidence, not just the absence of an email.

## What NOT to do

- Do not fix either bug in this pass — investigation only, per standing rule (investigation and fix are separate prompts).
- Do not touch deactivation/hard-delete logic — that node is LOCKED.

## Deliverable

Root-cause findings for both bugs (grep output, relevant code snippets, logs) — saved back to `03_Investigation_and_Errors/` as `Chat11_Investigation_EmailBugs_Result.md`. No code changes in this pass.
