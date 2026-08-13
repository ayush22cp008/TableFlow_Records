# Node 9 Schema Investigation Facts (Chat 16)
**Type: Code-Read Only (No changes made)**

## 1. Staff Role Storage (VERIFIED)
- **Table Name:** `profiles`
- **Column Name:** `role`
- **Type:** Plain `text` with a `CHECK` constraint (Not a custom Postgres enum type).
  - Valid values enforced by constraint: `'customer', 'owner', 'waiter', 'cook', 'manager'`
- **Relation to `auth.users`:** It is a 1:1 relation using the same `id` column. The `id` column in `profiles` maps directly to `auth.users.id`. This is verified by the existing RLS helper function which queries `WHERE id = auth.uid()`.

## 2. Orders Table Reference (VERIFIED)
Exact column list extracted from schema/types for foreign key references:
- `id` (uuid)
- `table_id` (uuid, nullable)
- `customer_id` (uuid, nullable, maps to `auth.users.id`)
- `status` (text with CHECK constraint: `'placed', 'preparing', 'ready', 'served', 'billed', 'cancelled'`)
- `subtotal` (numeric)
- `service_charge_applied` (boolean)
- `service_charge_amount` (numeric)
- `total` (numeric)
- `party_size` (integer, nullable)
- `is_priority` (boolean)
- `daily_number` (integer, nullable)
- `cancellation_reason` (text, nullable)
- `cancellation_category` (text with CHECK constraint: `'fire', 'food_safety', 'natural_disaster', 'other', 'manual'`, nullable)
- `created_at` (timestamptz)
- `updated_at` (timestamptz)

## 3. Existing RLS Pattern (VERIFIED)
The project uses a standard `has_role(allowed_roles text[])` Postgres helper function to check roles from the `profiles` table.

**Example RLS from `orders` table:**

*SELECT Policy:*
```sql
CREATE POLICY "orders_select" ON orders FOR SELECT 
USING (customer_id = auth.uid() OR has_role(ARRAY['waiter', 'cook', 'manager', 'owner']));
```

*UPDATE Policy:*
```sql
CREATE POLICY "orders_update_manager" ON orders FOR UPDATE
USING ( has_role(ARRAY['manager']) AND status IN ('placed', 'served') )
WITH CHECK ( has_role(ARRAY['manager']) AND status IN ('preparing', 'billed', 'cancelled') );
```
