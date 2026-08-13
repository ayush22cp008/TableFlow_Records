# Investigation: Google Invite Email Not Sent

## 1. Tracing the Email Trigger Logic
The logic to send the invite code email lives in `/api/send-invite/route.ts`. 

- **In the non-Google path (`/signup`)**: When a user selects a staff role and enters their email, clicking "Continue" triggers an explicit `fetch('/api/send-invite')` call in `components/AuthForm.tsx`. This successfully fires the Resend API, and the UI transitions to a screen that says "We sent an invite code to {email}".
- **In the Google path (`/auth/select-role`)**: A code search confirms that `fetch('/api/send-invite')` is **never called**. 

## 2. Execution Trace on `/auth/select-role`
When a user arrives at the `/auth/select-role` screen after Google auth and selects a staff role (Cook/Waiter/Manager), the UI immediately displays a text input labeled "Staff Invite Code" and waits for the user to submit it. 

**Does execution ever reach the email trigger?**
**No.** The email trigger is not blocked by the RLS bug found earlier, nor does it fail silently—the code to trigger the email simply does not exist in `app/auth/select-role/page.tsx`. The component assumes the user somehow already has the code, and makes no attempt to auto-detect the pending invite or send the email.

## 3. Resend Logs & Root Cause Conclusion
Because the API call is entirely missing from the Google signup flow, there are **zero** send attempts reaching Resend when a user goes down this path. 

**Conclusion:**
This confirms the hypothesis that this is a symptom of the same overarching architectural divergence found in the previous investigation (`Chat4_Investigation_GoogleStaffSignup.md`). The Google auth path (`/auth/select-role`) and the standard path (`AuthForm.tsx`) were built with entirely separate logic. The standard path was fully wired up with the email trigger and Admin SDK validation, whereas the Google path was left as a rudimentary form that neither sends the email nor has the RLS permissions to validate the code once entered.
