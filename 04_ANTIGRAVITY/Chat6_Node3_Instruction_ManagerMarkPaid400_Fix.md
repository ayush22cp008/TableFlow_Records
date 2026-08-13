# TableFlow — Chat #6 / Node 3 — Instruction (Fix): Manager "Mark Paid" 400 Error

**To:** Antigravity
**From:** Claude
**Type:** Fix — approved, root cause confirmed via investigation

---

## Approved

Root cause confirmed in `Chat6_Node3_Investigation_ManagerMarkPaid400_Result.md`: the live database's `orders.status` CHECK constraint was never migrated to include `'billed'`. The schema file (`profiles.sql`) has the correct definition, but `CREATE TABLE IF NOT EXISTS` skipped it on the already-existing live table — so the live constraint is stale.

## Fix — Write a proper migration

Create a new migration file in `supabase/migrations/` (follow existing naming convention, e.g. `20260808000001_fix_orders_status_constraint.sql`) with:

```sql
ALTER TABLE orders DROP CONSTRAINT IF EXISTS orders_status_check;

ALTER TABLE orders ADD CONSTRAINT orders_status_check
  CHECK (status IN ('placed', 'preparing', 'ready', 'served', 'billed', 'cancelled'));
```

Adjust the constraint name if the actual live constraint name differs from `orders_status_check` — check the DB directly (e.g. via Supabase dashboard or `\d orders` / information_schema) to confirm the exact constraint name before writing the `DROP CONSTRAINT` line, so it doesn't silently no-op on a wrong name.

## Also verify — secondary flagged risk

Confirm whether migration `20260804000003_node3_manager_schema.sql` (which adds the `payment_method` column) has actually been applied to the live database. If not applied, apply it. This was flagged as a secondary candidate that could also cause a 400 independently — rule it out explicitly, don't assume.

## Verification Before Reporting Back

- Migration applied to live database (not just written as a file — actually run)
- Confirm via direct query that `orders.status` constraint now includes `'billed'`
- Confirm `payment_method` column exists on live `orders` table
- `npm run build` — 0 errors (should be unaffected, but confirm)

## Output

Do NOT push to GitHub yet — manual trigger by Ayush after live browser test. Report back with: migration file path, confirmation the migration was actually run against the live DB, and constraint verification result. Ayush will test live: click Mark Paid on a served order, confirm it disappears from Billing Queue and status becomes `billed` — per evidence rule.
