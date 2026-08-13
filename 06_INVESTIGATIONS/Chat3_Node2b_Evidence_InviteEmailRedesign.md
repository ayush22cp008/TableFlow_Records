# Evidence: Invite Email Delivery Redesign

## 1. Exact Files Created/Changed (Final List)
- **`package.json`**: Added `resend` SDK dependency.
- **`.env.local`**: Appended `RESEND_API_KEY`. (Ayush added `SUPABASE_SERVICE_ROLE_KEY` prior to execution).
- **`lib/supabaseAdmin.ts`**: (NEW) Initialized Supabase Admin client explicitly with the Service Role key.
- **`app/api/send-invite/route.ts`**: (NEW) Created backend route to fetch unused invite code using Admin SDK and email it via Resend.
- **`app/api/auth/staff-signup/route.ts`**: (NEW) Created backend route to validate the code, create the user account (with `email_confirm: true` via Admin SDK), update the `profiles` table to assign the correct staff role, and mark the code as used.
- **`components/AuthForm.tsx`**: (MODIFIED) Implemented the redesign of the UI flow for staff role signups. Splits role/email selection from code/password submission. Integrates calls to the new backend API routes and uses `signInWithPassword` for auto sign-in on success.

## 2. Build Result
- `npm run build` **PASSED** perfectly (0 errors).

## 3. Security Confirmation
- **Confirmed**: The `SUPABASE_SERVICE_ROLE_KEY` is strictly isolated to `lib/supabaseAdmin.ts`. This admin client is **only** imported and executed in the server-side API routes (`app/api/send-invite/route.ts` and `app/api/auth/staff-signup/route.ts`).
- **Confirmed**: No part of the Service Role key or the Admin SDK is exposed to the client bundle or `components/AuthForm.tsx`. Client side only performs fetch requests to our internal API endpoints.

## 4. Deviations from Original Plan
- **None**: The implementation strictly adhered to the planned architecture. Some ESLint strictness fixes were applied to `catch` blocks (removing explicit `any` types), but there were no structural or logic deviations from the Master Prompt redesign plan.
