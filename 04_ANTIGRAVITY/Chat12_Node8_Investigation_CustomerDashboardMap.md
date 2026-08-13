# Investigation: Full Customer Dashboard Map (Backend + Navigation)

**Node:** 8 — Customer Dashboard Revamp
**Type:** Investigation ONLY — no fix, no code changes.

## Context
Before making any Node 8 changes (reservation nav move, reservation status, my-orders table label), we need the full picture of the customer-facing side — not just the 3 isolated pages already discussed. Isolated frontend fixes risk missing backend gaps (already found one: `reservation_requests` has no `customer_id` link).

## What to investigate

1. **Route inventory** — List every route under `app/order/*` (and any other customer-facing route, e.g. `/reserve/*`). For each: what it does, what component renders it.

2. **Navigation graph** — For each customer route, where can the user go FROM it and where do they arrive FROM (i.e. what links to what). Include the current Menu page's "Reserve a Table" button flow specifically.

3. **Backend connection per route** — For each customer route, list: which Supabase table(s)/RPC(s) it reads/writes, and whether that data is scoped to the logged-in customer (via `user_id`/`profiles.id`) or is anonymous/name-matched.

4. **Owner-side connection points** — For each customer-facing feature (orders, reservations), confirm which owner-side dashboard/table currently reads the SAME data (e.g. owner's Tables page reads `reservation_requests`, Manager/Cook dashboards read `orders`). Just confirm the shared backend surface — no need to re-investigate owner-side logic already locked.

5. **Auth/identity check** — Is the customer definitely logged in with a stable `user_id` across all these routes (via `profiles` table), or are any flows still anonymous/session-based? This directly affects whether reservation-customer linking (Node 8 item #2) can use `user_id`.

## Output
Write findings to `03_Investigation_and_Errors/Chat12_Node8_Investigation_CustomerDashboardMap.md` — route inventory, navigation graph, backend connection table (route → table/RPC → customer-scoped y/n), and owner-side shared connection points. No fix, no code edits, no recommendations — just the map.
