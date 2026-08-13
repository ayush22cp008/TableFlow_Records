# Chat 8: Manager Bug 3b (Reservation Table Release) Investigation

## 1. Owner's Working Flow (Generate Bill)
- **File:** `app/dashboard/billing/[orderId]/page.tsx`
- **Lines:** 62-69
- **Code:**
  ```typescript
  if (tableData) {
    const currentOccupied = tableData.occupied_seats || 0
    const newOccupiedSeats = Math.max(0, currentOccupied - order.party_size)
    await supabase.from('restaurant_tables').update({
      occupied_seats: newOccupiedSeats,
      status: newOccupiedSeats === 0 ? 'available' : 'occupied',
      reserved_from: null // <--- Unconditionally clears it
    }).eq('id', order.table_id)
  }
  ```

## 2. Owner's Handling of `reserved_from`
- The Owner's billing flow explicitly and **unconditionally** sets `reserved_from: null` whenever ANY bill is paid on a table.
- This forces the table to lose its "Reserved" visual state immediately.

## 3. Current `mark_order_paid` RPC
- As instructed in the previous fix, `mark_order_paid` strictly adhered to the cancellation logic template and did **not** touch `reserved_from`.
- Therefore, when Manager marks a reservation order Paid, `occupied_seats` drops to 0, but `reserved_from` retains its timestamp.

## 4. Root Cause Conclusion
- In `app/dashboard/tables/page.tsx`, the `getTableDisplay` function prioritizes the `reserved_from` column. If it is non-null and within a 30-minute window, the table is styled purple ("Reserved"), **overriding** the `status = 'available'` or `occupied_seats = 0` checks.
- Because `mark_order_paid` never clears `reserved_from`, the table stays visually purple. Walk-in orders don't have this issue because they never had a `reserved_from` value set in the first place.

## 5. Recommended Fix Approach
- **Multi-order Risk:** Blindly copying the Owner's approach (unconditionally setting `reserved_from: null`) would introduce a subtle bug for tables with multiple orders. If a reserved table has two orders, paying the first one would instantly strip the "Reserved" state, even though the table is still occupied by the same party.
- **Recommendation:** Update the `mark_order_paid` RPC to **conditionally** clear `reserved_from` ONLY when the table is fully releasing (i.e., when `occupied_seats <= 0`).
  ```sql
  reserved_from = CASE WHEN (occupied_seats - v_order_record.party_size) <= 0 THEN NULL ELSE reserved_from END
  ```
  This safely fixes the reservation release bug for the Manager, while natively supporting multi-order scenarios. (Note: Owner's billing flow has the premature-clear bug and should eventually just call this same RPC, but that's out of scope for now.)

**Status:** Investigation complete. Awaiting fix instruction. No code has been modified.
