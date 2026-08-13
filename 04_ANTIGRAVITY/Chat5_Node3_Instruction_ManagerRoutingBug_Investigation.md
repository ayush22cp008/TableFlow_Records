# TableFlow — Chat #5 / Node 3 — Instruction (Investigation): Manager Login Routes to Owner Dashboard

**To:** Antigravity
**From:** Claude
**Type:** Investigation ONLY — do not fix yet, report findings back first

---

## Bug Report

Ayush generated a staff invite code for role `manager` (email: `ayushhalpati008@gmail.com`), received the invite email correctly (confirms `manager` in the email body), entered the invite code on signup, clicked Continue — and landed on the **Owner Dashboard**, not `/dashboard/manager`.

## Confirmed via DB (screenshot evidence)

`public.profiles` table checked directly:
- Exactly **one row** exists for `ayushhalpati008@gmail.com`
- `role` column = `manager` (correct, not overwritten)
- `id` = `309e8722-1bdc-41c1-b456-ba...`
- No duplicate rows for this email
- The only two `owner` role rows belong to different emails entirely (`owner.demo@tableflow.systems` and `ayushhalpati09@gmail.com` — note the **09**, different from the manager account's **008**)

**Conclusion: the database is correct.** `profiles.role = 'manager'` for this user. The bug is in routing/session logic, not data.

## Hypotheses to Investigate (do not assume — check both)

1. **Stale session/cookie:** Ayush was likely already logged in as Owner (`ayushhalpati09@gmail.com`) in the same browser before testing the manager invite flow. Check whether the signup/invite-accept flow properly signs out any existing session before creating/signing in the new user, or whether it's possible the old Owner session cookie persisted and the post-signup redirect read role from a stale session instead of the freshly created one.
2. **Routing logic bug:** Inspect `app/auth/callback/route.ts` and `middleware.ts` for how role is read and matched. Check the order of conditions — confirm there isn't a fallback/default that resolves to `owner` before the `manager` branch is checked, and confirm the role value is being fetched fresh from `profiles` (not from a cached JWT claim or stale client state) at the moment of redirect.

## Do NOT fix yet

Per the investigation/fix separation rule — report back with:
- Which hypothesis (or both, or neither) is confirmed
- Exact code location(s) where the issue occurs
- Proposed fix (description only, do not implement)

Ayush will confirm the fix approach before you implement.
