# Chat #3 | Node 2b — Decision: invite_codes Expiry Mechanism (LOCKED)

## Context
Investigation (`Chat3_Node2b_Investigation_ReservationExpiryMechanism.md`) found the reservation-request expiry is NOT a DB mechanism — it's a client-side runtime filter, no deletion ever happens. Node 2a spec required invite_codes to "auto-delete" after 30 min, which the reservation pattern doesn't actually do.

## Decision
invite_codes gets its OWN mechanism, diverging from reservation pattern — real deletion, because invite codes carry PII (staff email/name) and letting expired rows pile up is worse than for reservations.

**Mechanism: pg_cron, 2-step**
1. **Mark step** — cron job runs periodically, sets `status = 'expired'` on any `invite_codes` row where `used = false` AND `created_at + interval '30 minutes' < now()`
2. **Delete step** — batch DELETE all rows where `status = 'expired'`

## Schema implication
`invite_codes` needs a `status` column (or reuse/extend existing `used` boolean into a proper status enum: `unused` / `used` / `expired`) rather than just a plain `used` boolean, since we now track a third state.

## Next
Antigravity to implement as part of Node 2b schema work — pg_cron job creation + `invite_codes` table with status field per this design.
