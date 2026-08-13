Chat #3 | Node 2b | Instruction — Implement Invite Email Redesign (answers to open questions, proceed)

Per Chat3_Node2b_InviteEmailRedesign_ImplementationPlan.md. Both open questions answered:

## Answer 1: Email sending mechanism
Install the `resend` npm package (official SDK). Build app/api/send-invite/route.ts as proposed:
- Accept { email, role }
- Look up invite_codes for unused, matching, not-expired row
- If found: send email via Resend SDK from hello@tableflow.systems (reuse the existing verified domain — a NEW Resend API key needs to be created in the Resend dashboard and added as an env var, e.g. RESEND_API_KEY; this is separate from the SMTP credentials already configured in Supabase Auth settings, which only power Supabase's own default emails)
- Return success without exposing the code to the client
- If not found: return an error response

## Answer 2: Supabase native email confirmation bypass
Do NOT use client-side supabase.auth.signUp() for the staff path. Instead, use the Supabase Admin SDK's createUser() method with `email_confirm: true` set at creation time. This creates the account already confirmed in one step — no separate confirm call needed, no default Supabase confirmation email triggered. This requires the Supabase Service Role key (server-side only, in an API route — never expose this key to the client). Owner/Customer signup path is UNTOUCHED and continues using client-side signUp() + OTP exactly as before.

## Proceed with implementation
1. app/api/send-invite/route.ts — new route as scoped above.
2. A new server-side route or extend an existing one to handle final staff account creation via Admin SDK createUser() (accepts email, password, role, invite code; re-validates the 3-layer check server-side before creating; marks invite_codes row as 'used' on success).
3. components/AuthForm.tsx — redesign staff signup steps per the original plan (Step 1 role select, Step 2 email entry -> calls send-invite, Step 3 code+password entry -> calls the new account-creation route).
4. Do not touch Owner/Customer signup path, RLS policies, or cancellation system.

## Evidence required
Build must pass. Report exact files changed/created, and confirm the Service Role key is only used server-side (never in client bundle), before this gets merged.
