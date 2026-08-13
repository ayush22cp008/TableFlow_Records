# Chat 9: Node 4 — Reservation Double Booking Investigation

## 1. Locate Approve Handler
- **File:** `app/dashboard/tables/page.tsx`
- **Function Name:** `approveRequest(req: ReservationRequest, tableId: string)`
- **Code Snippet:**
  ```typescript
  async function approveRequest(req: ReservationRequest, tableId: string) {
    if (!tableId) { window.alert('Select a table first'); return }
    const uniqueCode = Math.floor(100000 + Math.random() * 900000).toString()
    
    const { error: reqErr } = await supabase.from('reservation_requests')
      .update({ status: 'approved', unique_code: uniqueCode, table_id: tableId })
      .eq('id', req.id)
    
    if (reqErr) { window.alert('Error approving request: ' + reqErr.message); return }

    await supabase.from('restaurant_tables')
      .update({ reserved_from: req.requested_time })
      .eq('id', tableId)
      
    fetchData()
  }
  ```

## 2. Validation Checks
- **Overlap check:** No. Neither the UI dropdown nor the `approveRequest` function checks for overlapping time windows against existing approved reservations.
- **Current occupancy check:** No. The dropdown only checks if the table's absolute total `capacity` is $\ge$ the request's `party_size` (`t.capacity >= req.party_size`). It completely ignores current `occupied_seats`. 
- **Server-side guards:** None. The DB allows the update unconditionally.

## 3. Relevant Schema Fields Mutated
At approval time, the following fields are mutated:
1. `reservation_requests.status` $\rightarrow$ `'approved'`
2. `reservation_requests.unique_code` $\rightarrow$ Random 6-digit string
3. `reservation_requests.table_id` $\rightarrow$ Assigned table UUID
4. `restaurant_tables.reserved_from` $\rightarrow$ Overwritten with the new `requested_time`
**Note:** `occupied_seats` is **not** mutated here. It only gets incremented later when the customer arrives and uses their code to place an order (`place_order_and_occupy_table` RPC).

## 4. Root Cause (How the bug occurred)
**Yes, approving a second reservation on an already-reserved table is currently possible with no guard at all.**
- A manager can select Table 1 for party "Ar" and approve it. `reserved_from` becomes 3:45 PM.
- A minute later, the manager can select Table 1 for party "Ak" and approve it. `reserved_from` is blindly overwritten to 3:55 PM. Both parties now have valid approval codes tied to Table 1.
- When both parties arrive and enter their codes, the `place_order_and_occupy_table` RPC fires twice. Since that RPC also blindly does `occupied_seats = occupied_seats + p_party_size` (without checking a hard cap against `capacity`), Table 1 ends up with an impossible `4/2` seated count.

**Status:** Investigation complete. Awaiting fix instructions. No code has been modified.
