v1.0 addresses my review well, but it makes two new factual claims I haven't verified yet — the hard-delete deactivation route and the invite_codes auto-delete-after-10 behavior. Both are load-bearing for its design (Section 8 and 9), so I need to check them before signing off.Confirmed accurate — this is a **hard delete**, not a soft `is_active=false` flag as the original investigation report implied ("Deactivation mechanism: `profiles.is_active` set to `false`"). v1.0 caught and corrected this. Now the invite_codes auto-delete claim.Confirmed accurate too. Both load-bearing facts in v1.0 check out against the real source. Quick check on the staff-signup route for the `staff_name` capture point, since that's the last unverified claim.One nuance v1.0's claim slightly overstates: it says this route "already has access to `codeData.staff_name`, so this is a localized onboarding/data-model change" — true, `codeData` is fetched with `select('*')` so `staff_name` is in scope, but the profile `.update()` calls in both the new-user and re-activation branches currently only set `{ role }` / `{ role, is_active: true, is_logged_in: false }`. Adding `staff_name` isn't automatic — both `.update()` calls need the field added explicitly. This is a correct but understated implementation detail, not a design flaw.

---

## Review of v1.0: **FITS — approved for spec/implementation stage**

Everything I flagged in my v0.1 review has been addressed correctly, and I independently re-verified the two new facts v1.0 introduces:

**Confirmed accurate:**
- **Deactivation is a hard delete** (`app/api/staff/deactivate/route.ts`) — deletes the `profiles` row, then deletes the `auth.users` row. This is materially different from what the original investigation report claimed (`is_active = false` soft-deactivate). v1.0's `ON DELETE SET NULL` foreign-key design is the correct fix given this — much cleaner than a separate deactivation trigger, since the hard delete already fires a native FK cascade.
- **`invite_codes` auto-delete is real and worse than "stale after 30 min"** — confirmed trigger deletes the oldest 10 `used` codes once 10 accumulate, unconditionally on time. So `invite_codes.staff_name` isn't just non-durable, it actively churns. Adding `profiles.staff_name`, populated at signup/reactivation, is the right call — not optional as v0.1's design left it.
- **RPC security corrections** (explicit `auth.uid()`/role/`is_active` checks, `search_path`, revoke-from-`PUBLIC`) match the `force_logout_all_staff` pattern I flagged as the correct template, not `mark_order_paid`.
- **RLS bypass closure** (Section 13) explicitly commits to removing the direct Cook/Waiter update policies rather than leaving it as an open question.
- **Cancellation race resolution** (Section 7) — the row-lock-first-wins mechanism plus an invariant trigger as a backstop for *all* paths (not just the claim RPC) is a stronger answer than v0.1 had. This also correctly covers Manager/Owner direct updates and `cancel_active_orders`, which was the actual gap.
- **Notification overlap** (Section 12) — correctly reasoned: claim doesn't change `status`, so the existing Node 9 status-based trigger won't double-fire on claim itself. This resolves the ambiguity I raised. (Completion RPCs do still change status, so `order_ready`/`order_served` will fire as before — that's intended, not a duplicate of `order_claimed`.)

**One implementation-level (not design-level) note to carry into the spec:**
- Section 18/9: populating `profiles.staff_name` requires adding the field to **both** `.update()` calls in `staff-signup/route.ts` (new-user path and reactivation path) — currently neither sets it. Trivial, but worth stating explicitly in the spec so it isn't missed in just one branch.

No other gaps found. Design v1.0 is sound and ready to move to spec/implementation stage.
