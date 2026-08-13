Chat #3 | Node 2b | Instruction — Investigate Existing Owner Signup + OTP Flow (INVESTIGATION ONLY, no fix)

Before deciding invite-code-vs-OTP timing for staff signup, need to understand the EXISTING owner/customer signup flow first.

Investigate:
1. Does Owner signup require any kind of code/invite at all currently? Confirm from AuthForm.tsx / signup flow — expected answer is NO (Owner signs up directly), but confirm from actual code, don't assume.
2. Where exactly does OTP verification happen in the current signup flow (which step, which file/function)? Is OTP verification BEFORE or AFTER the account/profile row is actually created in the DB?
3. For the newly merged staff role path (Fix 1, just applied) — trace through the exact current order of operations: invite code validation (3-layer check) -> OTP send -> OTP verify -> account creation -> invite code marked used. Report the ACTUAL current order as implemented, with file/line references, not the intended order.
4. Specifically confirm: at what exact point in this current code does the invite_codes row get updated to used/status change? Is it right after 3-layer validation passes (before OTP), or right after OTP verification succeeds?

Output: report findings only to 03_Investigation_and_Errors/Chat3_Node2b_Investigation_OwnerSignupAndOTPFlow.md

No fixes, no changes. Just trace and report the actual current implementation with file/line evidence.
