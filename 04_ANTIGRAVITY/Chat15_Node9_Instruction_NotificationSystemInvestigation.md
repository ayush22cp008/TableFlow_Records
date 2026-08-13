# Instruction — Node 9: Notification System Investigation

**Type:** Investigation only. Do NOT write/modify any code. Report findings only.

## Context

Building an in-app notification system (bell icon + list) for Phase 1 scope:
1. Order Status changes (Preparing → Ready → Served)
2. Order Cancelled (via Bulk Emergency Stop)
3. Reservation Approved/Rejected → Customer
4. New Reservation Request → Manager
5. New Order Placed → Manager

No email/push in Phase 1 — in-app only.

## What to investigate and report

### 1. Database schema
- Is there any existing `notifications` table or similar? If yes, share full schema (columns, types, RLS policies).
- If no such table exists, confirm that clearly.

### 2. Existing realtime channels
- Report current subscription setup for these 4 known channels: `orders_board`, `my_orders_realtime`, `menu_realtime`, `waitlist_realtime`.
- For each: which file/component subscribes to it, what table/event it listens to, and whether it's Supabase Realtime (postgres_changes) or Broadcast.

### 3. Order status change flow
- File/function where order status transitions happen (Preparing → Ready → Served).
- Confirm exact trigger point (e.g. Cook Dashboard button click → which RPC/mutation).

### 4. New Order Placed → Manager flow
- Locate the existing "Accept (Send to Kitchen)" flow on Manager Dashboard.
- Confirm: is this triggered by a new order insert, or a separate manual action? Report the exact mechanism — do not assume it's the same as new-order-notification-worthy event.

### 5. Bulk Emergency Stop cancellation flow
- File/component where Bulk Emergency Stop modal + confirm action lives.
- Confirm the cancel reason (Category + Details) is already captured and where it's stored (which table/column).
- Confirm whether order status at time of cancellation is accessible at that point in the code (needed for status-based staff targeting: Placed→Manager, Preparing→Cook, Ready→Waiter).

### 6. Reservation approve/reject flow
- File/function where Manager approves/rejects a reservation request.
- Confirm where Customer's user ID is accessible at that point (for targeting).

### 7. Bell icon / notification UI
- Confirm there is currently NO existing bell icon or notification list component anywhere in the codebase. Flag if one exists (even partial/unused).

### 8. Roles and user ID access
- Confirm how each dashboard currently identifies "who is logged in" (e.g. Supabase auth session, user table join) — needed to know how to target notifications per user.

## Explicitly out of scope for this investigation
- Table toggle, new table, menu changes, invite used, Order Billed/Paid, Waitlist Seated/Cancelled, Staff Role Changed/Force Logout — do not investigate these.
- Do not propose a schema or fix. Report only what exists today.

## Output

Save findings to: `03_Investigation_and_Errors/Chat15_Node9_Investigation_NotificationSystemDiscovery.md`
