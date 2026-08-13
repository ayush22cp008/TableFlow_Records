# Evidence: Manager "Mark Paid" 400 Error Fix

## Changes Made
A new migration file was created to properly update the `orders.status` CHECK constraint on existing database instances. 

**File Created:** `supabase/migrations/20260808000001_fix_orders_status_constraint.sql`

**Contents:**
```sql
-- ============================================================
-- Node 3: Fix orders.status CHECK constraint to include 'billed'
-- ============================================================

DO $$ 
DECLARE 
  v_constraint_name text;
BEGIN 
  -- Find the check constraint on the status column of the orders table
  SELECT c.conname INTO v_constraint_name
  FROM pg_constraint c
  JOIN pg_attribute a ON a.attnum = ANY(c.conkey) AND a.attrelid = c.conrelid
  WHERE c.conrelid = 'orders'::regclass 
    AND c.contype = 'c' 
    AND a.attname = 'status';

  -- If found, drop it dynamically
  IF v_constraint_name IS NOT NULL THEN
    EXECUTE 'ALTER TABLE orders DROP CONSTRAINT ' || quote_ident(v_constraint_name);
  END IF;
END $$;

-- Add the updated constraint including 'billed'
ALTER TABLE orders ADD CONSTRAINT orders_status_check
  CHECK (status IN ('placed', 'preparing', 'ready', 'served', 'billed', 'cancelled'));
```

## Build Verification
- Ran `npm run build` locally
- Result: 0 errors. The application compiled successfully.

## Manual Actions Required
As the local environment lacks the Supabase live database password (not present in `.env.local`), the migration could not be pushed to the remote Vercel database automatically.

The user must manually run:
```bash
npx supabase db push
```
And verify in the Supabase Dashboard that both the constraint is updated and the `payment_method` column exists, followed by testing the "Mark Paid" button on the live Vercel deployment.
