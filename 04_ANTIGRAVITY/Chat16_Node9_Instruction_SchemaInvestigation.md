# Instruction — Node 9 Schema Investigation (Chat 16)

**Type: Investigation only. Read-only. No code changes, no file edits.**

## Goal
Gather exact DB schema facts needed to design the `notifications` table + RLS policies for Node 9. Do not propose a schema — just report facts.

## What to check

1. **Staff role storage**
   - Which table stores each user's role (Owner/Manager/Cook/Waiter/Customer)? (likely `profiles`, but confirm exact table name)
   - Exact column name for the role value (e.g. `role`)
   - Is it a Postgres `enum` type or plain `text`? If enum, list all values.
   - How is this table keyed to `auth.users` — same `id` (1:1), or a separate `user_id` foreign key column?

2. **Orders table reference**
   - Paste the exact column list of the `orders` table (names + types), for foreign-key reference (e.g. `customer_id`, `status` type/values).

3. **Existing RLS pattern (for consistency)**
   - Paste one existing RLS policy from any staff-facing table (e.g. `orders` or `reservation_requests`) — SELECT and UPDATE policy definitions — so the new `notifications` table's RLS can follow the same auth-check pattern already used in this project.

## Output
Write findings as a new file in `03_Investigation_and_Errors/`:
`Chat16_Node9_Investigation_SchemaFacts.md`

Tag every fact VERIFIED (from actual schema/code) — do not infer or guess. If something can't be found, say UNKNOWN, don't fill in a guess.

No fix, no schema design, no migration SQL in this pass — facts only.
