# Investigation Result: Deactivate Role-Downgrade + Re-Invite Login Bugs

**Type:** Investigation Result (Chat 10)

## Bug 1 — Confirm role downgrade is correct
**Result:** Confirmed correct. 
This behavior perfectly matches the implemented plan (`Chat9_Implementation_FixRemoveBanProperRoleGate.md`). The plan explicitly stated: *"When deactivating, update the profile to: `is_active: false`, `is_logged_in: false`, and `role: 'customer'`"*. The fact that deactivated staff show `role: 'customer'` and drop from the Active Staff list is exactly what we intended. No fix is required here.

## Bug 2 — Re-invite signup fails with "Invalid login credentials"
**Result:** Root cause identified.

**Findings:**
1. The issue originates in `app/api/auth/staff-signup/route.ts` around line 52 inside the `if (existingProfile)` block (the re-activation path).
2. When a previously deactivated user (now a customer) signs up via an invite code, the API successfully updates their `profiles` row (`role`, `is_active: true`, `is_logged_in: false`) and updates `user_metadata.role` via `supabaseAdmin.auth.admin.updateUserById()`.
3. **The exact gap:** The `password` provided in the signup request is entirely ignored in this `existingProfile` block. The API never calls `updateUserById` to set the new password.
4. Because the original account was created via Google OAuth, it doesn't have a password identity. Since the new password isn't set, the immediate client-side `signInWithPassword` attempt fails with "Invalid login credentials".

**Next Steps:**
Awaiting Claude's plan to implement the fix for Bug 2 (likely involving passing the `password` field into `updateUserById` if the user is being re-activated).
