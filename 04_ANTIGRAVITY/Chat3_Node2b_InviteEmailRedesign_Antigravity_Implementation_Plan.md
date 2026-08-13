# Invite Email Delivery Redesign for Staff - Implementation Plan

This plan outlines the redesign of the staff signup flow, implementing the email-based invite code delivery and using the Supabase Admin SDK for account creation without requiring OTP verification.

## User Review Required

> [!IMPORTANT]
> The **Supabase Service Role Key** needs to be added to `.env.local` as `SUPABASE_SERVICE_ROLE_KEY`. It was not found in the project directory. Please retrieve it from your Supabase dashboard and add it before approving this plan, or provide it to me so I can add it.
> I will add the `RESEND_API_KEY` to `.env.local` using the value from `resend_api_key.txt`.

## Proposed Changes

### `package.json`
- Install `resend` SDK.

---

### `.env.local`
#### [MODIFY] `.env.local`
- Add `RESEND_API_KEY=re_6qNodWZT_Mm2FkGabhpoDiz9AxZUx18tT`
- Add `SUPABASE_SERVICE_ROLE_KEY=your_service_role_key` (Requires user action)

---

### Backend API Routes (Server-Side)

#### [NEW] `lib/supabaseAdmin.ts`
- Create a shared Supabase Admin client initialized with the Service Role Key. This ensures the key is strictly isolated server-side.

#### [NEW] `app/api/send-invite/route.ts`
- **POST Route**: Accepts `email` and `role`.
- Looks up the `invite_codes` table for a matching, unused row using the Admin SDK.
- If valid, uses Resend to email the `code` to the staff's email address.
- Returns a success response.

#### [NEW] `app/api/auth/staff-signup/route.ts`
- **POST Route**: Accepts `email`, `role`, `code`, and `password`.
- Validates the `invite_codes` table (code matches email/role and is unused).
- Calls `supabaseAdmin.auth.admin.createUser({ email, password, email_confirm: true, user_metadata: { role } })` to create the account.
- Marks the `invite_codes` row as `used`.
- Returns a success response.

---

### Frontend UI

#### [MODIFY] `components/AuthForm.tsx`
- **Role Selection Step (Staff)**: When a staff role is selected, prompt for **Email** instead of the invite code.
- **Send Invite**: Submitting the role/email calls `/api/send-invite`. On success, transitions to the new `staff-verify` step.
- **Staff Verify Step**: Prompts for the **Invite Code** (received via email) and **Password**.
- **Sign Up**: Submitting calls `/api/auth/staff-signup`. On success, automatically signs the user in via `supabase.auth.signInWithPassword` and redirects them to the appropriate dashboard.
- **Owner/Customer Flow**: Remains untouched (Client-side `signUp()` + OTP).

## Verification Plan

### Automated Tests
- Run `npm run build` to ensure the Next.js build passes.

### Manual Verification
- Verify the Service Role Key is strictly isolated in server-side API routes (`lib/supabaseAdmin.ts` and `app/api/*`).
- Verify the exact files changed match the proposed plan.
- (Manual by Ayush) Test the staff signup flow in the browser to ensure the email is received and the account is created successfully without needing an OTP.
