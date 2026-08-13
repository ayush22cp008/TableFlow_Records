Chat #3 | Node 2b | Instruction — Service Role Key Added, Proceed with Implementation

SUPABASE_SERVICE_ROLE_KEY has been added to .env.local by Ayush. Proceed with the plan exactly as written in Chat3_Node2b_InviteEmailRedesign_Antigravity_Implementation_Plan.md.

## Before building
Confirm SUPABASE_SERVICE_ROLE_KEY is present and readable in .env.local (do not print the key value anywhere, just confirm it exists).

## Proceed with
1. Install `resend` npm package
2. Create `lib/supabaseAdmin.ts` — Admin client using Service Role key, server-side only
3. Create `app/api/send-invite/route.ts` — email/role lookup + Resend send
4. Create `app/api/auth/staff-signup/route.ts` — validate code, createUser() with email_confirm:true, mark invite_codes used
5. Modify `components/AuthForm.tsx` — new staff signup steps (role → email → send-invite → code+password → staff-signup → auto sign-in via signInWithPassword)
6. Do not touch Owner/Customer signup path, RLS policies, or cancellation system

## Verification
- Run `npm run build` — must pass
- Confirm Service Role key is only referenced in `lib/supabaseAdmin.ts` and server-side API routes — never in any client component or client bundle

## Evidence required (report before this gets merged)
Report to: `TableFlow_Staff_Role_System/03_Investigation_and_Errors/Chat3_Node2b_Evidence_InviteEmailRedesign.md`

Include:
- Exact files created/changed (final list)
- Build pass/fail result
- Confirmation Service Role key never appears client-side (which files reference it — should only be lib/supabaseAdmin.ts and the two API routes)
- Any deviations from the original plan, with reasoning

No push to GitHub yet — that happens only after Ayush completes manual browser testing and gives explicit go-ahead.
