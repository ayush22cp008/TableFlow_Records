# Instruction — Fix: Proper Reservation Lifecycle Status (Node 4, Option B)

**Chat #9 — Node 4 — Fix only. Investigation + verification already complete (see `Chat9_Node4_Investigation_VerifyRootCause.md`). Do not re-investigate.**

## Confirmed root cause
`reservation_requests.status = 'approved'` is treated as permanent by the overlap check in `approveRequest`. There is no transition to a non-active status when a table gets released via:
- Manager clicking "Clear" on a table
- `mark_order_paid` RPC (table fully released)
- `cancel_active_orders` RPC
- No-show / manual DB reset

5 confirmed stale rows currently exist (Table 1 x2, Table 2, Table 3, Table 6) stuck in `approved` despite their tables being free.

## Fix — Option B: proper lifecycle status

1. **Add a new terminal status** to `reservation_requests` — use `'cancelled'` (covers no-show, manual clear, and released-without-completion cases). Reuse the existing `'completed'` status where it already applies (customer scanned code, order placed) — do not duplicate that.

2. **Update every flow that releases a table** to also transition any `reservation_requests` row still `status = 'approved'` for that `table_id` to `'cancelled'`:
   - **Clear button handler** (`app/dashboard/tables/page.tsx`) — on clear, find approved `reservation_requests` rows for that `table_id` and set `status = 'cancelled'`.
   - **`mark_order_paid` RPC** — when it fully releases a table (clears `reserved_from`), also update matching `reservation_requests` rows to `'cancelled'`.
   - **`cancel_active_orders` RPC** — same, when it releases a table.

3. **Data cleanup migration** — write a migration to backfill the 5 known stale rows (and any other existing `approved` rows whose table is currently free) to `'cancelled'`. Use the same "table is free" condition from the investigation (table's `occupied_seats = 0` and `reserved_from IS NULL`) to identify which rows to backfill — don't hardcode just the 5 known IDs, in case others exist.

4. **No changes needed to the overlap check itself** (`r.status === 'approved' && r.table_id === tableId`) — it's correct as written; it was only ever wrong because `approved` wasn't being retired properly. Leave it as-is.

## Constraints
- Do not touch Node 3 (Manager Dashboard) — LOCKED. If `mark_order_paid`/`cancel_active_orders` RPC changes risk touching Node 3 behavior, flag it before proceeding rather than assuming.
- Migration file only — do not run `db push` (Supabase CLI unavailable on this machine). Ayush will run it manually via SQL Editor.
- Keep the fix minimal — no unrelated refactors.
- No manual UI testing on your side — Ayush will test manually and confirm before push.
- Report back: files touched, exact diff/snippet per flow (Clear handler, both RPCs), migration file content, and build/compile result.
