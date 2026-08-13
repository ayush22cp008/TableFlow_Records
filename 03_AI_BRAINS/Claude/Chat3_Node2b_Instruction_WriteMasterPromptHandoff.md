Chat #3 | Node 2b | Instruction — Write Your Own Master Prompt for Chat Handoff

This chat is getting too long. Before it ends, write your own master prompt to Drive so the next chat can resume your work accurately — you know your actual current state better than anyone else.

## Save to
`TableFlow_Staff_Role_System/Chat3_Node2b_MasterPrompt_Antigravity_Handoff.md`

## Required sections (cover all of these, based on YOUR actual current state — don't guess, report what you actually did/know)

1. **Project** — TableFlow Staff Role System, bridge folder path.

2. **Current task** — Invite Email Delivery Redesign for staff (Waiter/Cook/Manager) signup only. Summarize the new flow: Owner generates code (no email) → staff enters email → system looks up matching unused/non-expired invite_codes row → emails code via Resend → staff enters code+password → 3-layer validation (unchanged logic) → account created via Supabase Admin SDK createUser() with email_confirm:true → invite_codes marked used. Owner/Customer signup is untouched — do not confuse with this task.

3. **Actual current build status** — this is the most important section. Report exactly:
   - Which of these 3 files have you touched, and what's the state of each: `app/api/send-invite/route.ts`, the staff account-creation route, `components/AuthForm.tsx`
   - Does the build currently pass or fail?
   - What's fully done vs. partially done vs. not started?
   - Any errors, blockers, or open questions you hit?
   - Is `RESEND_API_KEY` created/added yet?

4. **Key technical decisions already locked** (do not re-litigate, just carry forward):
   - Resend npm SDK (not raw fetch)
   - Admin SDK createUser() with email_confirm:true, using Service Role key server-side only (never in client bundle)
   - Owner/Customer signup path stays on client-side signUp() + OTP, untouched

5. **Evidence required before merge** — build must pass, exact files changed/created, confirmation Service Role key never exposed client-side.

6. **Standing rules** — investigation and fix always separate prompts, no fix without evidence, no browser/UI testing (Ayush does that manually), push to GitHub only on Ayush's explicit go-ahead, Drive is for coordination/specs only — not source code.

## Output
Just create the file at the path above with these sections filled from your real state. Confirm back with the file path once done — no other action needed right now.
