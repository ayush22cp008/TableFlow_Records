# TableFlow — Chat #18 — Mark Paid / Table Release Investigation

**Type:** Investigation Only — NO code changes, NO fixes, NO migration changes  
**Repository:** https://github.com/ayush22cp008/TableFlow  
**Records Repository:** https://github.com/ayush22cp008/TableFlow_Records  
**Investigation Location:** `06_INVESTIGATIONS/`  
**Status:** READY FOR ANTIGRAVITY INVESTIGATION  
**Related Area:** Order completion / table occupancy lifecycle  
**Node Assignment:** Do not assume a node number until the investigation/report confirms it.

---

## 1. Problem Being Investigated

Manual Vercel testing shows the following behavior:

1. A restaurant order is processed normally.
2. The order reaches **Served** and appears in the Manager **Billing Queue**.
3. Manager clicks **Mark Paid**.
4. The order leaves the Billing Queue successfully.
5. The corresponding restaurant table still appears occupied/full on the Tables page instead of being released.

This behavior needs a source-level + database-level investigation before any fix is made.

A previous cancellation flow has been observed to release table occupancy correctly. The investigation must compare the two completion paths rather than changing the existing system blindly.

---

## 2. Important Current Evidence to Verify

The following are starting observations from the current `main` source and must be independently re-verified by Antigravity:

### Manager UI

File:
`app/dashboard/manager/page.tsx`

Current `markPaid(orderId)` implementation appears to perform a direct order update:

```ts
async function markPaid(orderId: string) {
  await supabase.from('orders').update({ 
    status: 'billed', 
    updated_at: new Date().toISOString() 
  }).eq('id', orderId)
  
  fetchOrders()
}
```

The investigation must confirm whether the deployed application is using this exact behavior.

### Existing database RPC

File:
`supabase/migrations/20260809000001_mark_order_paid_rpc.sql`

The existing `public.mark_order_paid(p_order_id, p_payment_method)` function contains logic that:

- finds a `served` order;
- changes the order to `billed`;
- updates `payment_method`;
- decrements `restaurant_tables.occupied_seats`;
- recalculates table `status`.

The investigation must verify whether this RPC is actually called by the Manager UI in the deployed application.

### Order placement occupancy

File:
`supabase/migrations/20260809000003_place_order_capacity_guard.sql`

The `place_order_and_occupy_table` RPC increments:

`restaurant_tables.occupied_seats`

by `p_party_size` when an order is placed.

### Cancellation comparison

File:
`supabase/migrations/20260804000002_node2b_cancellation.sql`

The cancellation RPC `cancel_active_orders(...)` explicitly decrements:

`restaurant_tables.occupied_seats`

for cancelled active orders.

These facts must be verified against the actual source and current database state, not merely assumed.

---

## 3. Investigation Objectives

Antigravity must determine, with evidence:

### A. Exact Manager Mark Paid flow

Identify:

- exact file;
- exact function/handler;
- exact Supabase operation(s);
- whether the operation is a direct `orders` update or an RPC;
- whether any table-release logic is executed as part of the same action.

### B. Exact database release behavior

Verify:

- current `mark_order_paid` function definition;
- whether it contains table-release logic;
- whether the UI bypasses that function;
- current permissions/RLS that allow Manager completion;
- whether the table-release logic is actually reachable from the UI.

### C. Database truth before and after payment

Use one real test order tied to a real table.

Record before Mark Paid:

- order id;
- daily number;
- order status;
- table id;
- party size;
- table number;
- table capacity;
- `restaurant_tables.occupied_seats`;
- table status;
- count/sum of other active orders for that same table.

Then click **Mark Paid** in the Vercel UI.

Record after Mark Paid:

- order status;
- payment method;
- same table id/number;
- `restaurant_tables.occupied_seats`;
- table status;
- remaining active orders for that table.

The report must compare the exact before/after values.

### D. Rule out a frontend-only display problem

Verify the database directly after payment.

Do not classify the issue as a UI refresh bug unless database state proves the release happened correctly and only the UI is stale.

### E. Rule out multiple-order effects

Check whether the affected table has:

- zero remaining active orders after the paid order; or
- one or more remaining active orders.

Determine whether the intended release rule is:

- decrement by this order's party size;
- release only when all active orders on the table are completed;
- or another documented invariant.

The report must state the rule supported by the existing code.

### F. Compare working cancellation behavior

Compare:

`Mark Paid -> billed`

against:

`Cancellation -> cancelled`

and explicitly identify which path changes `restaurant_tables.occupied_seats`.

### G. Check for race/transaction issues

Determine whether table release is:

- inside the same transaction as order completion;
- separate from it;
- protected by row locking;
- vulnerable to partial completion.

Do not redesign the system during investigation. Report the current behavior only.

---

## 4. Required SQL Evidence

Antigravity may use SQL Editor queries similar to these, replacing placeholders with real values.

### 4.1 Inspect the target order

```sql
SELECT
  o.id,
  o.daily_number,
  o.status,
  o.table_id,
  o.party_size,
  o.payment_method,
  rt.table_number,
  rt.capacity,
  rt.occupied_seats,
  rt.status AS table_status
FROM public.orders o
LEFT JOIN public.restaurant_tables rt
  ON rt.id = o.table_id
WHERE o.id = 'PUT_REAL_ORDER_UUID_HERE';
```

### 4.2 Check all active orders on the same table

```sql
SELECT
  o.id,
  o.daily_number,
  o.status,
  o.party_size,
  o.table_id
FROM public.orders o
WHERE o.table_id = 'PUT_REAL_TABLE_UUID_HERE'
  AND o.status IN ('placed', 'preparing', 'ready', 'served')
ORDER BY o.created_at;
```

### 4.3 Verify the current RPC definition

```sql
SELECT
  n.nspname AS schema_name,
  p.proname AS function_name,
  pg_get_function_identity_arguments(p.oid) AS arguments,
  pg_get_functiondef(p.oid) AS function_definition
FROM pg_proc p
JOIN pg_namespace n
  ON n.oid = p.pronamespace
WHERE n.nspname = 'public'
  AND p.proname = 'mark_order_paid';
```

### 4.4 Inspect table occupancy for a target table

```sql
SELECT
  rt.id,
  rt.table_number,
  rt.capacity,
  rt.occupied_seats,
  rt.status
FROM public.restaurant_tables rt
WHERE rt.id = 'PUT_REAL_TABLE_UUID_HERE';
```

Antigravity may add additional queries where necessary.

---

## 5. Important Investigation Constraint

**Do NOT fix anything in this investigation.**

Do not:

- edit `app/dashboard/manager/page.tsx`;
- change `mark_order_paid`;
- change RLS;
- create migrations;
- alter `restaurant_tables`;
- manually change occupancy values to hide the bug;
- change the deployed application;
- introduce a new table-release mechanism.

The purpose of this file is to produce a definitive root-cause report so the next stage can go directly to a controlled fix.

---

## 6. Root Cause Standard

Do not report a suspected root cause as confirmed without evidence.

The final report must classify the finding as one of:

- **CONFIRMED ROOT CAUSE**
- **PARTIAL / CONTRIBUTING CAUSE**
- **NOT THE ROOT CAUSE**
- **INCONCLUSIVE — MORE EVIDENCE REQUIRED**

A root cause is **CONFIRMED** only when the report shows the exact execution path and demonstrates why the observed database state follows from that path.

---

## 7. Required Antigravity Result Report

Save the final investigation report as:

`06_INVESTIGATIONS/Chat18_TableRelease_MarkPaid_Investigation_Result.md`

The report must contain:

1. **Executive Summary**
2. **Exact Manager Mark Paid Handler**
3. **Exact Supabase Call Used by the UI**
4. **Current `mark_order_paid` Function Definition / Behavior**
5. **Before-State SQL Evidence**
6. **After-State SQL Evidence**
7. **Cancellation Flow Comparison**
8. **Multiple-Order / Table Occupancy Analysis**
9. **RLS / Permission Findings**
10. **Frontend vs Database Determination**
11. **Confirmed Root Cause**
12. **Minimal Fix Direction**
13. **Potential Regression Risks**
14. **Recommended Fix Test Cases**
15. **Files Examined**
16. **Commands / SQL Queries Used**
17. **Final Conclusion**

The report must clearly distinguish:

- **Observed**
- **Verified from source**
- **Verified from database**
- **Inference**
- **Recommendation**

---

## 8. Handoff Requirement

When the result report is complete, Antigravity must stop.

Do not implement the fix in the same pass.

The next stage will use:

`Chat18_TableRelease_MarkPaid_Investigation_Result.md`

as the primary evidence file for the separate fix/design step.

**Goal:** finish the investigation once, establish the exact root cause, and avoid repeatedly re-investigating the same Mark Paid/table-release problem.
