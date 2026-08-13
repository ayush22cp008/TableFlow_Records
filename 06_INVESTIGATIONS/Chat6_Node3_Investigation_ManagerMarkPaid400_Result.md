# Investigation Result: Manager "Mark Paid" 400 Error

## 1. Exact Code Snippet
The update call happens in `app/dashboard/manager/page.tsx` within the `markPaid` function:

```typescript
  async function markPaid(orderId: string) {
    const method = paymentMethods[orderId] || 'cash'
    await supabase.from('orders').update({ 
      status: 'billed',
      payment_method: method,
      updated_at: new Date().toISOString() 
    }).eq('id', orderId)
    
    fetchOrders()
  }
```

## 2. Current Schema / Constraint Definition
In the base schema file `profiles.sql`, the `orders` table is defined as:

```sql
create table if not exists orders (
  id uuid primary key default gen_random_uuid(),
  table_id uuid references restaurant_tables(id) on delete set null,
  customer_id uuid references profiles(id) on delete set null,
  status text not null default 'placed'
    check (status in ('placed', 'preparing', 'ready', 'served', 'billed', 'cancelled')),
  subtotal numeric(10,2) not null default 0,
  -- ... other columns ...
);
```

## 3. Conclusion: Root Cause Identified

There is a **single root cause** for the 400 Bad Request: a **stale CHECK constraint on the live database**.

Although `'billed'` is present in `profiles.sql`, that file uses `CREATE TABLE IF NOT EXISTS orders`. For any existing database (like the live environment or an older local setup), Postgres sees the table already exists and **skips the entire block**. 

Because there is no migration file in `supabase/migrations/` that explicitly runs an `ALTER TABLE orders DROP CONSTRAINT ...` and `ADD CONSTRAINT ...` to update the allowed statuses, the live database still has the older version of the constraint (which only allowed `'placed', 'preparing', 'ready', 'served', 'cancelled'`). 

When `markPaid` tries to update the status to `'billed'`, Postgres rejects it due to the CHECK constraint violation, and PostgREST translates this database error into a 400 Bad Request.

*(A secondary potential candidate is that the `payment_method` migration (`20260804000003_node3_manager_schema.sql`) was never applied to the live database, which would also throw a 400 because the column wouldn't exist. However, the missing CHECK constraint migration for `status` guarantees a 400 even if `payment_method` exists).*
