# Plan: Implement Invite Code Email-Delivery Redesign

This plan outlines the redesign of the Staff Signup sequence (Waiter, Cook, Manager) to use email-delivered invite codes, without altering the Owner/Customer OTP flow.

## Investigation Report

Per the instructions, I first investigated how Resend is invoked in the codebase and checked the schema.

1. **How is Resend currently invoked?**
   - **Finding:** There is **NO** custom Resend integration (no API routes, no `resend` SDK package) currently in the Next.js codebase. 
   - The current OTP and confirmation emails are handled **entirely by Supabase Auth internally**. Supabase is configured via its own dashboard to use the Resend SMTP/API for `tableflow.systems`. 
   - Because of this, there is no existing custom email-sending mechanism to "reuse" in the Next.js code.

2. **Invite Codes Schema Support:**
   - **Finding:** The `invite_codes` table schema fully supports the required lookups. It has `code`, `role`, `staff_email`, and `status` columns.

## Open Questions

> [!IMPORTANT]
> Since there is no existing Resend implementation in the Next.js codebase to reuse, I will need to build a new API route (`app/api/send-invite/route.ts`). I propose using standard `fetch` to `https://api.resend.com/emails` (or installing the `resend` npm package) to keep it simple. Do you approve this approach?

> [!WARNING]  
> If Supabase Auth requires email confirmation for ALL signups, `signUp()` will still send a default Supabase OTP email. The instruction says: "The current OTP step (supabase.auth.verifyOtp) for this staff path only... must be removed". If we remove it, the staff user will be created but might remain unverified in Supabase. We can bypass Supabase email confirmation by either:
> 1. We cannot easily bypass it client-side. We might need to use the Supabase Admin SDK to auto-confirm the user, OR we simply let `signUp()` work and we just don't do OTP, assuming `supabase` allows unverified signins? 
> Please advise how to handle Supabase's native email confirmation requirement for Staff roles if we skip OTP verification.

## Proposed Changes

### 1. `package.json`
*(If using the SDK is preferred over raw `fetch`)*
- Install the `resend` package to safely and easily dispatch emails from the API route.

### 2. `app/api/send-invite/route.ts`
#### [NEW] [route.ts](file:///C:/Users/ayush/Desktop/vibethon_project/app/api/send-invite/route.ts)
- Create a new POST endpoint that accepts `{ email, role }`.
- Validate the input.
- Lookup the `invite_codes` table to find an `unused` code for this email and role.
- If found, use Resend to send an email from `noreply@tableflow.systems` containing the code.
- Return success (without exposing the code to the client).

### 3. `components/AuthForm.tsx`
#### [MODIFY] [AuthForm.tsx](file:///C:/Users/ayush/Desktop/vibethon_project/components/AuthForm.tsx)
Redesign the state machine specifically for staff roles:
- **Step 1 (Unchanged):** User selects Waiter, Cook, or Manager.
- **Step 2 (Email Entry):** 
  - Change UI: Hide the invite code input. Show only the Email input.
  - On submit: Call `/api/send-invite`. 
  - If it succeeds, transition to Step 3. If it fails (e.g. no invite code found), show an error and stop.
- **Step 3 (Code + Password Entry):**
  - Change UI: Show an input for the "Invite Code (from email)" and an input for "Password".
  - On submit: 
    - Run the existing full 3-layer DB validation on the provided code.
    - If valid, call `supabase.auth.signUp()` (this will NOT trigger OTP because we won't call `verifyOtp`).

## Verification Plan

### Automated Tests
- Run `npm run build` to ensure type-safety.

### Manual Verification
- Attempt to sign up as a Waiter with an email that has NO invite code -> verify it is blocked at Step 2.
- Attempt to sign up with a valid invite code email -> verify email is "sent" and UI progresses to Step 3.
- Submit correct code and password -> verify account creation and dashboard redirect.
