# Parked Issue — Owner Signup Hardcoded Invite Code (NOT part of Node 2b)

## Found during
Investigation: Chat3_Node2b_Investigation_OwnerSignupAndOTPFlow.md

## Issue
Owner signup requires a code, but it's a hardcoded client-side string 'TableFlow12' in components/AuthForm.tsx (lines 44-48) — not a DB-validated per-user code like the staff invite_codes system. This is visible in the client JS bundle, so anyone who inspects it can create an Owner account.

## Status
PARKED — user explicitly deferred this to a future node/task. Not in scope for current Node 2b work (Staff Role System: invite codes, RLS, cancellation system).

## Confirmed working correctly (no action needed)
Staff invite code timing (separate from this issue) was verified correct: invite_codes row is marked 'used' only AFTER OTP verification succeeds (end of handleVerifyOtp), not right after 3-layer validation. This means if OTP fails/is abandoned, the code remains reusable — matches intended design.

## Next
Revisit as its own node/task later — decide on a proper mechanism (e.g. env-var server-side secret, DB-validated owner invite similar to staff system, or manual admin approval) when picked up.
