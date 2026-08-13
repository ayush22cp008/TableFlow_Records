# Investigation Report: Owner Signup & OTP Flow

Chat #3 | Node 2b

## 1. Does Owner signup require any kind of code/invite currently?
**Yes.** 
In `components/AuthForm.tsx` (lines 44-48), the Owner role is gated by a client-side hardcoded code. If `role === 'owner'`, it strictly requires the user to input `OWNER_INVITE_CODE` (which is hardcoded to `'TableFlow12'`).

## 2. OTP Verification Timing & DB Account Creation
- **Where:** OTP verification happens in `handleVerifyOtp` (Step 3) in `components/AuthForm.tsx` (lines 113-155).
- **Timing:** OTP verification happens **AFTER** the database account is created.
  - In Step 2 (`handleSignup`), calling `supabase.auth.signUp()` immediately creates the row in `auth.users` (which in turn triggers the creation of the `public.profiles` row).
  - The OTP verification step (`supabase.auth.verifyOtp()`) only confirms the email of this already-existing account.

## 3. Exact Current Order of Operations (Staff Role Path)
Based on the newly merged staff role code in `components/AuthForm.tsx`, the actual sequence of operations is:
1. **Step 1 (`handleRoleStep`, lines 50-55):** Basic client-side prefix/tag validation (e.g. `inviteCode.includes('WT')`).
2. **Step 2 (`handleSignup`, lines 64-86):** Full 3-layer DB validation (queries `invite_codes` to ensure the code exists, is 'unused', matches the user's `email`, and matches the `role`).
3. **Step 2 (`handleSignup`, lines 94-101):** Account creation and OTP email dispatched via `supabase.auth.signUp()`.
4. **Step 3 (`handleVerifyOtp`, lines 118-128):** User submits OTP code, verified via `supabase.auth.verifyOtp()`.
5. **Step 3 (`handleVerifyOtp`, lines 136):** Profile role is updated in DB.
6. **Step 3 (`handleVerifyOtp`, lines 138-140):** Invite code is marked as 'used' in the DB.

## 4. Exact Point of Invite Code Status Change
The `invite_codes` row is updated to `status: 'used'` **right AFTER OTP verification succeeds**. 
It does NOT happen after the 3-layer validation in Step 2. It happens at the very end of Step 3 (`handleVerifyOtp`), ensuring the code is only burned if the user successfully validates their email and completes the signup process.
