# Investigation Results — Node 11 Bug: Reservation Reject Not Updating + Refresh Resets to Form

## 1. Reject-Specific Code Path
**In `app/dashboard/tables/page.tsx`:**

The `rejectRequest` function executes an `UPDATE` on the `reservation_requests` table, setting the `status` to `'rejected'`:
```tsx
  async function rejectRequest(req: ReservationRequest) {
    await supabase.from('reservation_requests').update({ status: 'rejected' }).eq('id', req.id)
    fetchData()
  }
```
This is identical in structure to `approveRequest` which updates the same row with `status: 'approved'`. Thus, the `UPDATE` event definitely fires, and the realtime subscription *is* receiving the payload.

## 2. Customer Page's Handling of Different Statuses
**In `app/order/reservation/page.tsx`:**

The `checkReservation` function (which runs on load and on realtime updates) explicitly filters the fetched data in application memory:
```tsx
      if (data && ['pending', 'approved', 'arrived'].includes(data.status)) {
        setActiveReservation(data)
      }
```
**Why the UI doesn't update on reject:**
When the realtime event triggers, `checkReservation` re-fetches the record. It sees `data.status === 'rejected'`. Because `'rejected'` is missing from the allowed list `['pending', 'approved', 'arrived']`, the `if` condition evaluates to `false`. 
Crucially, there is no `else` block to clear `activeReservation` or update it with the rejected state. Thus, `activeReservation` retains its previous state (`'pending'`) in React state, and the UI appears "stuck" and ignores the reject completely.

## 3. Refresh-Resets-to-Form Bug
**The initial data-fetch logic:**

When the page is refreshed, `checkReservation` runs on mount. It runs this exact query to find the customer's reservation:
```tsx
      const { data } = await supabase
        .from('reservation_requests')
        .select('*, restaurant_tables(table_number)')
        .eq('customer_id', user.id)
        .order('created_at', { ascending: false })
        .limit(1)
        .single()
```
**Why they see the form:**
1. The query successfully fetches the most recent reservation, which is now `status: 'rejected'`.
2. It hits the same `if` condition: `['pending', 'approved', 'arrived'].includes('rejected')` -> `false`.
3. Because `activeReservation` initializes to `null` on page load, and the `if` condition is false, `setActiveReservation` is never called.
4. `activeReservation` remains `null`.
5. The component's render logic uses a ternary operator: `activeReservation ? (show status) : (show form)`. Since it is `null`, it renders the "Request a Table" form again.

This allows the customer to submit a duplicate request, because the app assumes they have no active/visible reservation.
