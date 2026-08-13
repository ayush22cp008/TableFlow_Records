# Master Prompt: Antigravity Handoff

## 1. Project
**Project Name:** TableFlow Staff Role System
**Bridge Folder Path:** `G:\My Drive\TableFlow_Staff_Role_System\`
**Project Root Path:** `C:\Users\ayush\Desktop\vibethon_project`

## 2. Current Task
**Task:** Invite Email Delivery Redesign for staff (Waiter/Cook/Manager) signup only.
**New Flow Summary:**
1. Owner generates code (no email).
2. Staff selects role & enters email in signup form.
3. System looks up matching unused/non-expired `invite_codes` row for this email+role.
4. If valid, emails code via Resend.
5. Staff enters the received code + sets a password.
6. System runs the 3-layer validation (unchanged logic).
7. If valid, account created via **Supabase Admin SDK `createUser()` with `email_confirm: true`**.
8. `invite_codes` marked as used.
*(Owner/Customer signup is untouched — standard client-side `signUp()` + OTP).*

## 3. Actual Current Build Status
- `app/api/send-invite/route.ts`: **Not started.**
- Staff account-creation API route (for Admin SDK): **Not started.**
- `components/AuthForm.tsx`: **Not updated for this redesign yet.** (It currently has the fixes from the previous bug hunt, but the Email Delivery UI changes are untouched).
- **Build Status:** The build currently **PASSES** perfectly.
- **Completion Status:** Fully investigated & planned. Execution of this redesign is **Not started**.
- **Blockers/Errors:** None. The previous step was just to submit an investigation/implementation plan. This handoff prompt bridges into the actual execution phase.
- **Environment:** `RESEND_API_KEY` and Supabase Service Role Key are documented in the README but need to be properly added/configured in the environment and `package.json` needs the `resend` SDK and Supabase JS SDK (for Admin functions) updated if necessary.

## 4. Key Technical Decisions Already Locked
- Use **Resend npm SDK** (not raw fetch).
- Use **Admin SDK `createUser()` with `email_confirm: true`**. Must use **Service Role key server-side only** (never expose in the client bundle).
- Owner/Customer signup path stays entirely on client-side `signUp()` + OTP and remains completely untouched.

## 5. Evidence Required Before Merge
- The Next.js build must pass (`npm run build`).
- Exact files changed/created must be explicitly reported.
- Explicit confirmation that the Supabase Service Role key is strictly isolated in server-side API routes and NEVER exposed client-side.

## 6. Standing Rules
- Investigation and fix are always separate prompts.
- No fixes/merges without evidence (build logs).
- No browser/UI testing (Ayush does that manually).
- Push to GitHub only on Ayush's explicit go-ahead.
- Google Drive is for coordination/specs only, not source code.
