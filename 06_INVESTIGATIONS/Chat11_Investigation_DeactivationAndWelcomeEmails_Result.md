# Investigation Results: Deactivation & Welcome Emails

## 1. Existing Email Infrastructure
- **Location:** The email logic is currently used in `app/api/send-invite/route.ts`.
- **Service & Package:** We are using `Resend` via the `resend` npm package.
- **Pattern:**
  ```typescript
  import { Resend } from 'resend'
  const resend = new Resend(process.env.RESEND_API_KEY)

  // Usage
  const { error: emailError } = await resend.emails.send({
    from: 'TableFlow Staff System <noreply@tableflow.systems>',
    to: [email],
    subject: '...',
    html: '...'
  })
  ```
- **Conclusion:** Yes, this exact service and pattern can be seamlessly reused for both the Deactivation and Welcome emails without requiring any new infrastructure setup.

## 2. Deactivation Email (Hook-in Point)
- **Target File:** `app/api/staff/deactivate/route.ts`
- **Data Availability:** Currently, only `userId` is passed to this route. We will need to look up the user's email *before* deleting them. We can do this via `await supabaseAdmin.auth.admin.getUserById(userId)`.
- **Hook-in Point:**
  - **Where:** Right after verifying the caller is an owner (Step 2) and **strictly before** Step 3 (where the `public.profiles` row is deleted). 
  - **Why:** Once the auth user or profile is deleted, we lose the ability to fetch their email address.

## 3. Welcome Email (Hook-in Point)
- **Target File:** `app/api/auth/staff-signup/route.ts`
- **Data Availability:** We already have the staff member's `email` and `role` directly from the request body (`const { email, role, code, password } = await request.json()`).
- **Hook-in Point:**
  - **Where:** At the very end of the successful signup block, right before marking the invite code as used (Step 3) or right before returning `NextResponse.json({ success: true, userId })`.
  - **Why:** At this point, the user account has been successfully created/reactivated.

## 4. Failure Handling
- **Recommendation:** **Do not block the main action.** 
- **Implementation:** Both email dispatch calls should be wrapped in their own `try/catch` blocks (or we just log the `emailError` returned by Resend). We should NOT return a `500` error if the email fails. The deactivation or signup MUST succeed regardless of whether the email goes through. We will simply log the email failure to `console.error` and proceed with returning `{ success: true }`.
