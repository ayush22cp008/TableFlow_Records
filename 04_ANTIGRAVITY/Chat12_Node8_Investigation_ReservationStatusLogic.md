# Investigation: Reservation Status Feature (Customer-Side)

**Node:** 8 — Customer Dashboard Revamp
**Type:** Investigation ONLY — no fix, no code changes.

## Context
Building a new customer-facing Reservation page (separate from Menu). It needs to show the customer their **current/latest reservation status** — table, member count, and current state (e.g. confirmed / seated / expired).

Ayush believes the following may already exist on the owner side, but is not 100% sure:
- A reservation auto-resets to "available" if the customer doesn't show up within 30 minutes of reserved time.
- Owner can also manually clear/release a reservation.

## What to investigate
1. Find the reservation table/schema (likely `reservations` or similar) — what status values/enum exist? (e.g. `pending`, `confirmed`, `seated`, `expired`, `cancelled`, `completed`)
2. Does a 30-minute no-show auto-release mechanism actually exist in code (cron job, DB trigger, or checked-on-read logic)? If yes — where, and how does it work exactly?
3. Does the owner have a manual "clear/release table" action already built? Where (which component/route)?
4. For a given logged-in customer, how would we query "their" current/latest reservation? (Is reservation linked to `user_id` / `profiles.id`, or only to table + phone/email?)
5. What fields are available to display: table number, member/party size, reserved time, current status?

## Output
Write findings to `03_Investigation_and_Errors/Chat12_Node8_Investigation_ReservationStatusLogic.md` — schema/fields found, existing auto-release logic (code location + how it works, or confirmation it does NOT exist), owner manual-clear location, and how to fetch a customer's own latest reservation. No fix, no code edits.
