# TableFlow — Chat 17.1
## Authentication & Email Delivery Bug Investigation Report

**Project:** TableFlow  
**Chat:** 17.1  
**Incident:** Authentication / Email Delivery Failure  
**Status:** RESOLVED ✅  
**Affected area:** Authentication and email delivery  
**Node 11 relationship:** Separate from Node 11 — Node 11 remained CLOSED

---

## 1. Incident Summary

During a fresh production retest after returning to the TableFlow project, two email-related authentication problems were discovered:

1. **Staff invite email failed** with a generic UI message:
   `Failed to send invite email. Please try again later.`
2. **Customer signup failed** with a visible `{}` error instead of progressing to OTP verification.

The staff invite-code generation itself continued to work, so the failure was isolated to email delivery.

The two flows use different Resend integration paths:

- **Staff invite:** TableFlow API → Resend API using Vercel `RESEND_API_KEY`
- **Customer OTP:** Supabase Auth → Custom SMTP → Resend SMTP

---

## 2. Staff Invite Email — Root Cause

### Observed behavior

The Manager/Staff signup screen successfully generated and stored an invite code, but the invite email was not delivered.

Vercel production logs showed:

```text
POST /api/send-invite → 500
Resend error: {
  statusCode: 401,
  name: 'validation_error',
  ...
}
```

### Evidence

- `invite_codes` record was successfully created.
- `/api/send-invite` was reached successfully.
- The API then called Resend.
- Resend rejected the request with HTTP **401**.

The TableFlow source code sends from:

```text
TableFlow Staff System <noreply@tableflow.systems>
```

The Resend domain `tableflow.systems` was verified and showed:

> Domain verified: Your domain is ready to send emails.

Therefore the verified domain itself was not the issue. The failing component was the Resend API authentication/credential path.

---

## 3. Staff Invite Email — Fix Applied

A new Resend API key was created specifically for TableFlow with:

```text
Permission: Sending access
Domain: tableflow.systems
```

The new key was then:

1. Added/replaced in Vercel as `RESEND_API_KEY`.
2. Enabled for **Production and Preview**.
3. Saved as a **Secret**.
4. Vercel Production was redeployed so the new secret was loaded.

### Result

Staff invite email delivery was manually retested and confirmed successful.

**Staff invite email: FIXED ✅**

---

## 4. Customer Signup / OTP — Observed Problem

After the staff email issue was fixed, Customer signup was tested separately.

The UI reached the Customer credentials screen, but instead of successfully progressing to OTP verification it displayed:

```text
{}
```

Supabase Logs showed:

```text
POST /auth/v1/signup → 500
```

Example production event:

```json
{
  "method": "POST",
  "pathname": "/auth/v1/signup",
  "status": "500",
  "level": "error",
  "log_type": "edge",
  "auth_user": null
}
```

The available raw Auth log did not expose the underlying internal error message.

---

## 5. Investigation of Customer Signup

The current TableFlow frontend authentication code was inspected.

Customer signup uses the normal Supabase client flow:

```text
supabase.auth.signUp({
  email,
  password,
  options: {
    data: { role }
  }
})
```

No malformed request or obvious frontend implementation error was found.

The repository was also checked for the database-side auth trigger:

```text
auth.users
   ↓
on_auth_user_created
   ↓
public.handle_new_user()
   ↓
public.profiles
```

The trigger/function exists in the current TableFlow codebase.

Additional 403/406 entries were examined and determined to be unrelated stale-session effects from a previously deleted user:

- `/auth/v1/user` → 403
- `/rest/v1/profiles` → 406

Those events occurred before/around the signup testing and did not explain the `/auth/v1/signup` 500.

---

## 6. Customer OTP Email — Configuration Found

Supabase Authentication → Emails → SMTP Settings was inspected.

Custom SMTP was enabled with:

```text
Host: smtp.resend.com
Port: 587
Username: resend
Sender: hello@tableflow.systems
Sender name: TableFlow
```

The `tableflow.systems` domain was also verified in Resend.

The critical point was that Supabase stores the SMTP password securely and does not reveal it after saving. The SMTP password is the Resend credential used by Supabase Auth for SMTP delivery.

Because a new valid Resend API key had just been created for `tableflow.systems`, the Supabase Custom SMTP password was updated to use the new Resend credential as well.

---

## 7. Customer OTP Email — Fix Applied

The following fix was applied in Supabase:

1. Opened **Authentication → Emails → SMTP Settings**.
2. Kept Custom SMTP enabled.
3. Kept:
   - `smtp.resend.com`
   - Port `587`
   - Username `resend`
   - Sender `hello@tableflow.systems`
4. Replaced the saved SMTP password with the newly created Resend credential.
5. Saved the SMTP configuration.

No TableFlow application code change was required.

### Result

A fresh Customer signup was retested.

The Customer successfully completed email verification and reached the **Customer Dashboard**.

**Customer OTP/authentication: FIXED ✅**

---

## 8. Final Root Cause Assessment

### Confirmed

The Staff invite failure was caused by an invalid/rejected Resend API credential path, demonstrated directly by the production **401** returned by Resend.

The Customer OTP failure was resolved by updating the Resend credential used by Supabase Custom SMTP, indicating that the stored SMTP credential/configuration was stale or invalid.

### Not identified as root cause

- Customer signup frontend code: no obvious defect found.
- Node 11 realtime implementation: unrelated.
- Verified sending domain `tableflow.systems`: verified and ready to send.
- The 403/406 stale-session logs: unrelated to the signup 500.

---

## 9. Verification Results

| Area | Result |
|---|---|
| Staff invite-code generation | ✅ Working |
| Staff invite email | ✅ Working after new Resend API key + redeploy |
| Customer signup | ✅ Working |
| Customer OTP email | ✅ Working after Supabase SMTP credential update |
| Customer login | ✅ Working |
| Customer Dashboard access | ✅ Confirmed |
| Node 11 realtime functionality | ✅ Separately retested and passed |

---

## 10. Node 11 Isolation

This incident is **separate from Node 11**.

Node 11 — Realtime Auto-Updates — remained **CLOSED / LOCKED** throughout this investigation.

Node 11 was subsequently retested across the relevant dashboards and realtime flows. Updates were observed without manual browser refresh, including Manager, Cook, Waiter, Customer reservation status, Owner staff management, and related realtime dashboard behavior.

No Node 11 reopening is required because of this authentication/email incident.

---

## 11. Final Status

**CHAT 17.1 — AUTHENTICATION & EMAIL INCIDENT: RESOLVED ✅**

### Production verification

- Staff invite email: **PASS**
- Customer OTP email: **PASS**
- Customer signup: **PASS**
- Customer login: **PASS**
- Customer Dashboard: **PASS**
- Node 11: **CLOSED / PASS**

### Operational lesson

TableFlow has two separate Resend credential paths. Updating the Vercel `RESEND_API_KEY` fixes the application-side Resend API usage, but Supabase Auth Custom SMTP keeps its own stored credential and must be updated separately when the Resend credential changes.

---

## 12. Related Technical References

- TableFlow repository: https://github.com/ayush22cp008/TableFlow
- `components/AuthForm.tsx`
- `app/api/send-invite/route.ts`
- `supabase/migrations/20260817000001_node9_triggers_realtime.sql`
- `profiles.sql` (contains `handle_new_user()` trigger)

---

**Document:** Chat 17.1 Authentication & Email Delivery Bug Investigation  
**Status:** CLOSED / RESOLVED  
