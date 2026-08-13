# Investigation Results: Email Bugs

## Bug 1: Duplicate Invite-Code Email
**Root Cause:**
There are two places where `/api/send-invite` is called on the frontend, and both lack proper double-submit protection:

1. **`components/AuthForm.tsx` (Likely if testing via `/signup`)**:
   In the "role" step (around line 375), the "Continue" button does **not** have the `disabled={loading}` attribute, unlike all other buttons in the form:
   ```tsx
   <button
     id="role-next"
     type="submit"
     className="w-full py-3..."
     // MISSING: disabled={loading}
   >
     Continue
   </button>
   ```
   A quick double-click on this button fires the `handleRoleStep` function twice in parallel, sending two emails instantly.

2. **`app/auth/select-role/page.tsx` (Likely if testing while already logged in)**:
   The invite email is triggered via a `useEffect` that checks state (`!sentRoles.has(role)`). In React Strict Mode (common in dev), `useEffect` double-fires synchronously, and because `setSentRoles` is an async state update, the check passes twice, sending two emails. A `useRef` is required here to prevent race conditions.

## Bug 2: Missing Welcome Email (Worked on 2nd, but not 1st)
**Root Cause:**
The inconsistency is caused by the existence of **two different code paths** for staff signup, and a missing error check in the new code:

1. **The Missing Implementation Path (`verify-invite`)**:
   If the first signup (`CKDC0MN`) was done while the user was already authenticated (e.g. logging in as a customer, then going to "Select Role"), the frontend hits `app/api/auth/verify-invite/route.ts`. **This route does not contain the Welcome Email logic.** It successfully marks the code as used in the DB, but sends no email.
   The second signup happened *after* you performed a "Hard Delete", which logged the user out. Being logged out forced the second signup through the fresh `/signup` flow (`components/AuthForm.tsx`), which hits `app/api/auth/staff-signup/route.ts` where the Welcome email logic *does* exist.

2. **The Silent Failure Flaw (`staff-signup`)**:
   In `app/api/auth/staff-signup/route.ts`, the `resend.emails.send()` method does *not* throw an exception if the Resend API rejects the request (e.g. rate limit). It returns an object `{ data, error }`. Because the code was wrapped in a `try/catch` but the returned `error` object was ignored, any API rejection was swallowed completely silently without hitting the `catch` block or logging to the console.

**Conclusion:** We need to add `disabled={loading}` to AuthForm, fix the Strict Mode effect in `select-role`, duplicate the welcome email logic into `verify-invite`, and properly check for the `error` object returned by Resend in both places to ensure failures are actually logged.
