# Chat 8: Manager Bug 3 (Mark Paid Table Release) Investigation

## 1. Mark Paid Handler
- **File:** `app/dashboard/manager/page.tsx`
- **Lines:** 69-78
- **Code:**
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

## 2. Current DB Calls
- The `markPaid` function currently **only** updates the `orders` table (setting `status = 'billed'` and `payment_method`).
- **Missing:** It makes zero calls to update the `restaurant_tables` table.

## 3. Working Cancellation Flow Mechanism
- The cancellation flow uses a dedicated RPC: `cancel_active_orders` (`supabase/migrations/20260804000002_node2b_cancellation.sql`).
- It releases tables by dynamically decrementing the `occupied_seats` based on the specific order's `party_size`. If the seats hit 0, it changes the status back to `'available'`:
  ```sql
  UPDATE restaurant_tables
  SET occupied_seats = GREATEST(0, occupied_seats - v_order_record.party_size),
      status = CASE WHEN (occupied_seats - v_order_record.party_size) <= 0 THEN 'available' ELSE 'occupied' END,
      reserved_from = NULL
  WHERE id = v_order_record.table_id;
  ```

## 4. Table Occupancy Schema & Source of Truth
- **Table:** `restaurant_tables`
- **Columns:** `occupied_seats` (integer) tracks the exact number of people seated, and `status` (text) tracks `'available'` vs `'occupied'`.
- **Order Placement:** When an order is placed (`place_order_and_occupy_table` RPC), it increments `occupied_seats = occupied_seats + p_party_size` and sets `status = 'occupied'`.

## 5. Root Cause Conclusion
- **Root Cause:** The `markPaid` handler is missing the logic to decrement `occupied_seats` and update the table status.
- **Handling Multiple Orders per Table:** Because multiple orders can be assigned to the same table (incrementing `occupied_seats` cumulatively), we **must** account for multiple orders. We cannot just blindly set the table to `'available'` when one order is paid. 
- **Recommended Fix:** The fix must replicate the cancellation flow's logic: decrement `occupied_seats` by the paid order's `party_size`. The table will naturally flip to `'available'` only when the last order's seats are decremented (i.e., `occupied_seats <= 0`). This will likely require moving `markPaid` to an RPC (like `mark_order_paid`) to securely and atomically execute both the order update and the table decrement in one transaction.

**Status:** Investigation complete. Awaiting fix instruction. No code has been modified.
