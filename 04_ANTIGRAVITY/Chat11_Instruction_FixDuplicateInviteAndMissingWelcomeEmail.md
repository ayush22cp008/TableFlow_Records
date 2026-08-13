# Fix Instruction: Duplicate Invite Email + Missing Welcome Email

**Context:** Chat 11 — fixes based on confirmed root causes in `Chat11_Investigation_EmailBugs_Result.md`. Implement all 4 items below.

## Fix 1: Prevent duplicate invite email — `components/AuthForm.tsx`

Add `disabled={loading}` to the "Continue" button (`id="role-next"`, ~line 375) so a double-click can't fire `handleRoleStep` twice in parallel.

## Fix 2: Prevent duplicate invite email — `app/auth/select-role/page.tsx`

Replace the `sentRoles` state-based check with a `useRef` guard so the Strict Mode double-fire of `useEffect` can't race past an async `setSentRoles` update. Ref should be set synchronously the moment the send starts, before the API call, so the second invocation sees it immediately.

## Fix 3: Add welcome email to `app/api/auth/verify-invite/route.ts`

This route currently marks the invite code "used" but never sends the welcome email. Port the same welcome-email logic (Resend call, staff name/role/restaurant content) used in `app/api/auth/staff-signup/route.ts` into this route, after successful role assignment. Keep it consistent with the existing template/content already used in `staff-signup`.

## Fix 4: Stop silent email failures — both routes

In both `staff-signup/route.ts` and `verify-invite/route.ts` (after Fix 3), check the `{ data, error }` object returned by `resend.emails.send()`. If `error` is present, log it clearly (e.g. `console.error('[welcome-email] Resend error:', error)`) so failures are visible in build/server logs. Per standing rule: email failure must NOT block or fail the main signup/role-assignment flow — this is logging only, not a thrown error.

## Constraints

- Do not touch deactivation/hard-delete logic (LOCKED node).
- Do not change the Resend email template/wording beyond what's needed to port Fix 3 — keep visual consistency with the existing `staff-signup` welcome email.
- After implementing, run `npm run build` and report clean output as evidence (per evidence rule) — no manual UI testing needed from Antigravity side, Ayush will do that.

## Deliverable

Build result saved to `03_Investigation_and_Errors/` as `Chat11_Evidence_BuildResult_EmailFixes.md`, plus a short note listing exactly which files were changed.
