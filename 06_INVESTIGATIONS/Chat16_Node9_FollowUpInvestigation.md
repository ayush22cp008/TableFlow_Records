# Follow-Up Investigation Report: Node 9 (Chat 16)
**Type: Code-Read Only (No changes made)**

## Gap 1 — Bulk Cancel status-snapshot logic (VERIFIED)

- **Pre-RPC Status Fetching:** 
  The code inside `submitBulkCancel` (in `app/dashboard/orders/page.tsx`) does **not** fetch or map each affected order's current status before executing the RPC. It merely constructs a payload of IDs (`payloadIds`) based on the mode:
  ```typescript
  const payloadIds = bulkMode === 'all' ? null : selectedOrderIds
  
  const { error } = await supabase.rpc('cancel_active_orders', {
    p_reason: bulkReason.trim() || null,
    p_category: bulkCategory,
    p_order_ids: payloadIds
  })
  ```
- **Availability of Status in Memory:** 
  For "Select Specific" mode, the status *is* available in the client's `orders` state array (which renders the board), but the current logic ignores it and just passes the `selectedOrderIds` directly to the database. For "Cancel ALL", the client doesn't know all the exact IDs that the RPC will affect in the background.
- **Conclusion:** 
  **NO** such pre-RPC status snapshot currently exists in the code. The UI just triggers the RPC, and the database handles the state changes.

## Gap 2 — Full realtime channel inventory (VERIFIED)

The complete and exact list of realtime channels currently used in the codebase (identified via `.channel(` calls):

1. `orders_board` — `app/dashboard/orders/page.tsx`
2. `manager_orders_realtime` — `app/dashboard/manager/page.tsx`
3. `cook_orders_realtime` — `app/dashboard/cook/page.tsx`
4. `waiter_orders_realtime` — `app/dashboard/waiter/page.tsx`
5. `owner_analytics_realtime` — `app/dashboard/analytics/page.tsx`
6. `staff_management_realtime` — `app/dashboard/staff/page.tsx`
7. `owner_menu_realtime` — `app/dashboard/menu/page.tsx`
8. `tables_realtime` — `app/dashboard/tables/page.tsx`
9. `my_orders_realtime` — `app/order/my-orders/page.tsx`
10. `menu_realtime` — `app/order/page.tsx`
11. `reservation_status_realtime` — `app/order/reservation/page.tsx`

**Discrepancy Notes:**
- The list matches all the expected ones from the Node 11 locked master prompt (including `waiter_orders_realtime`, `owner_analytics_realtime`, `staff_management_realtime`, `owner_menu_realtime`, `tables_realtime`, and `menu_realtime`).
- There are no channels in `app/order/cart/page.tsx`.
