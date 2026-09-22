# Chat 18 — Mark Paid / Table Release Investigation Report

**Date:** 2026-09-22
**Project:** TableFlow
**Type:** Investigation Report

---

## 1. Executive Summary

The investigation confirms that the reported issue (tables failing to release when an order is marked as paid) is caused by a disconnect between the Frontend UI and the Database RPC layer.

The database correctly defines table-release logic inside a dedicated RPC (`mark_order_paid`), but the Manager UI explicitly bypasses this RPC, instead executing a direct `UPDATE` on the `orders` table. As a result, the `orders` table updates successfully (moving to `billed`), but the associated `restaurant_tables` occupancy remains untouched, permanently "orphaning" the table.

## 2. Investigation Findings

### A. Exact Manager Mark Paid flow (UI)
- **File:** `app/dashboard/manager/page.tsx`
- **Function:** `markPaid(orderId: string)`
- **Behavior:** The UI executes a direct Supabase client query: 
  `supabase.from('orders').update({ status: 'billed', updated_at: ... }).eq('id', orderId)`
- **Conclusion:** The frontend operates completely independently of the defined table-release logic. It only updates the `orders` table and ignores `restaurant_tables`.

### B. Exact database release behavior
- **File:** `supabase/migrations/20260809000002_mark_order_paid_reserved_from_fix.sql`
- **RPC:** `public.mark_order_paid(p_order_id uuid, p_payment_method text)`
- **Behavior:** The RPC contains robust release logic. Upon successful order completion, it:
  1. Decrements `occupied_seats` by `party_size`.
  2. Transitions table `status` to `available` if seats reach `0`.
  3. Clears `reserved_from`.
- **Conclusion:** The database logic is flawless, but it is **never called** by the Manager UI during the billing phase.

### C. Cancellation Comparison
- The cancellation flow works perfectly because the UI calls the RPC `cancel_active_orders`, which encapsulates the required table-release decrements inside the database transaction.

## 3. Recommended Fix Action (Future)

To resolve the table locking issue, the frontend must be updated to use the existing RPC. 

**Proposed Change in `app/dashboard/manager/page.tsx`:**
```typescript
// Replace direct update:
// await supabase.from('orders').update(...)

// With RPC call:
await supabase.rpc('mark_order_paid', { 
  p_order_id: orderId, 
  p_payment_method: paymentMethods[orderId] || 'cash' 
})
```

*(Note: As per investigation rules, no code changes or migrations were executed during this report.)*