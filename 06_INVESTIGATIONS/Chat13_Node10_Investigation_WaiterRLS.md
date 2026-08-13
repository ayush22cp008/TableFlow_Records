# Investigation: Waiter Role in Tables & Waitlist RLS Policies

**Query:** Why is the `waiter` role present in the `tables_write` and `waitlist_update` policies? Was it added during Fix B?

## Conclusion
**No, the `waiter` role was not newly added.** It was already present in the original database schema. Fix B only removed the `owner` role and left the existing roles (`waiter` and `manager`) intact.

## 1. Original Database RLS Policies
In the original migration file `20260804000001_node2b_schema_rls.sql`:

**Original Tables Policy (Lines 76-78):**
```sql
CREATE POLICY "tables_write" ON restaurant_tables FOR ALL
USING (has_role(ARRAY['waiter', 'manager', 'owner'])) 
WITH CHECK (has_role(ARRAY['waiter', 'manager', 'owner']));
```

**Original Waitlist Policy (Lines 102-103):**
```sql
CREATE POLICY "waitlist_update" ON waitlist FOR UPDATE 
USING (has_role(ARRAY['waiter', 'manager', 'owner']));
```
*Notice that `waiter` has always been part of these permissions to allow normal waitering operations (e.g., modifying table status when a customer sits down, adding orders).*

## 2. What Changed in Fix B (DB Level)
Fix B generated new SQL to explicitly drop the `owner` from these policies. The generated SQL simply removed `owner` but retained what was already there (`waiter`, `manager`):

```sql
-- Fix B Tables Policy
CREATE POLICY "tables_write_staff" ON restaurant_tables FOR ALL
USING (has_role(ARRAY['waiter', 'manager'])) 
WITH CHECK (has_role(ARRAY['waiter', 'manager']));
```

## 3. Frontend Code Restrictions
Despite the database allowing the `waiter` role to modify these tables, the **administrative frontend actions** on the Tables dashboard (`app/dashboard/tables/page.tsx`) were strictly restricted to `manager` only during Fix B.

For example, waitlist seat/cancel buttons:
```tsx
{role === 'manager' && (
  <div className="flex gap-2">
    <button onClick={() => seatWaitlistEntry(entry)}>Seat</button>
    <button onClick={() => cancelWaitlistEntry(entry)}>Cancel</button>
  </div>
)}
```

**Summary for Claude:**
The `waiter` role is authorized at the database level for their own waiter dashboard operations, but administrative actions on the shared Tables dashboard are fully locked down to `manager` at the UI level. No new DB access was granted to waiters.
