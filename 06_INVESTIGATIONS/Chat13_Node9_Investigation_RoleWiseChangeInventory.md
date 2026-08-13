# Investigation: Full-System Role-Wise Change Inventory

This document maps all meaningful UI and database changes in TableFlow where a role transition occurs or another role plausibly needs to be informed.

## 1. Orders

| Event / Transition | Changes (UI/DB) | File / Component | Trigger Role | Affected Role(s) | Current Notification State |
|---|---|---|---|---|---|
| **Order Placed** | Insert `orders`, `order_items`, `restaurant_tables` occupancy update (via RPC). | `app/order/cart/page.tsx` | Customer | Owner, Manager, Cook, Waiter | Silent (Realtime board update only) |
| **Order Status Update (Preparing/Ready/Served)** | Update `orders.status` | `app/dashboard/orders/page.tsx`, `cook/page.tsx`, `waiter/page.tsx`, `manager/page.tsx` | Owner, Manager, Cook, Waiter | Customer, downstream Staff (e.g. Cook -> Waiter) | Silent (Realtime board update only) |
| **Order Billed/Paid** | Update `orders.status` to `billed`, releases table. | `app/dashboard/billing/[orderId]/page.tsx` | Owner, Manager | Customer, Waitlist (freed table) | Silent |
| **Emergency Bulk Cancellation** | Update `orders.status` to `cancelled`, releases table. | `app/dashboard/orders/page.tsx` | Owner | Customer, Cook, Waiter, Manager | Silent (Orders disappear) |

## 2. Reservations

| Event / Transition | Changes (UI/DB) | File / Component | Trigger Role | Affected Role(s) | Current Notification State |
|---|---|---|---|---|---|
| **New Reservation Request** | Insert `reservation_requests` | `app/order/reservation/page.tsx` | Customer | Owner | Silent (Realtime missing, must manually refresh/check tables page) |
| **Reservation Approved** | Update `reservation_requests.status` to `approved`, set `restaurant_tables.reserved_from`, generates code. | `app/dashboard/tables/page.tsx` | Owner | Customer | Silent (Customer must manually check page) |
| **Reservation Rejected** | Update `reservation_requests.status` to `rejected`. | `app/dashboard/tables/page.tsx` | Owner | Customer | Silent |
| **Reservation Cancelled / Cleared** | Update `reservation_requests.status` to `cancelled`, clears `reserved_from`. | `app/dashboard/tables/page.tsx` | Owner | Customer | Silent |
| **Reservation Completed (Arrived)** | Update `reservation_requests.status` to `completed` upon order. | `app/order/cart/page.tsx` | Customer | Owner | Silent |

## 3. Waitlist & Tables

| Event / Transition | Changes (UI/DB) | File / Component | Trigger Role | Affected Role(s) | Current Notification State |
|---|---|---|---|---|---|
| **Joined Waitlist** | Insert `waitlist` | `app/order/cart/page.tsx` | Customer | Owner | Silent (Realtime board update only) |
| **Waitlist Seated** | Update `waitlist.status` to `seated`, updates `restaurant_tables.occupied_seats`. | `app/dashboard/tables/page.tsx` | Owner | Customer | Silent (Realtime UI change, but no audio/push) |
| **Waitlist Cancelled** | Update `waitlist.status` to `cancelled`. | `app/dashboard/tables/page.tsx` | Owner | Customer | Silent |
| **Table Status Toggled (Manual)** | Update `restaurant_tables.status`. | `app/dashboard/tables/page.tsx` | Owner | Waiters/Managers | Silent |
| **New Table Added** | Insert `restaurant_tables`. | `app/dashboard/tables/page.tsx` | Owner | All Roles | Silent |

## 4. Staff & Invites

| Event / Transition | Changes (UI/DB) | File / Component | Trigger Role | Affected Role(s) | Current Notification State |
|---|---|---|---|---|---|
| **Invite Generated** | Insert `invite_codes` | `app/api/send-invite/route.ts` | Owner | Invited Staff | **Already Surfaced** (Email sent via Resend) |
| **Invite Used (Signup)** | Update `invite_codes.status` to `used`, `profiles.role`, `is_active`. | `app/api/auth/staff-signup/route.ts` | Invited Staff | Owner | Silent |
| **Staff Deactivated** | Update `profiles.is_active` to `false`, `is_logged_in` to `false`. | `app/api/staff/deactivate/route.ts` | Owner | Deactivated Staff | **Already Surfaced** (Email sent) |
| **Staff Role Changed** | Update `profiles.role`. | `app/dashboard/staff/page.tsx` | Owner | Staff | Silent (Requires re-login/refresh) |
| **Force Logout All Staff** | Update `profiles.is_logged_in` to `false`. | `app/dashboard/staff/page.tsx` | Owner | Staff | Silent (Session invalidated next check) |

## 5. Menu & Other

| Event / Transition | Changes (UI/DB) | File / Component | Trigger Role | Affected Role(s) | Current Notification State |
|---|---|---|---|---|---|
| **Menu Item Add/Edit** | Insert/Update `menu_items`. | `app/dashboard/menu/page.tsx` | Owner, Manager | Customer, Staff | Silent (Realtime UI update) |
| **Menu Availability Toggled** | Update `menu_items.is_available`. | `app/dashboard/menu/page.tsx` | Owner, Manager | Customer, Waiter, Cook | Silent (Realtime UI update) |

## 6. Existing Mechanisms & Potential Risks

- **Realtime Channels Currently Active:**
  - `orders_board`: Used across all staff dashboards to sync the order state.
  - `my_orders_realtime`: Used by customer dashboard to sync their own order state.
  - `menu_realtime`: Used by customer to see menu changes live.
  - `waitlist_realtime`: Used on Owner Tables page to sync waitlist.
  *(Note: Realtime subscriptions update the UI state but do not trigger explicit "notifications" like popups or sounds. They are silent UI updates.)*
- **Emails:** Used explicitly for Staff Invite generation and Staff Deactivation via the Resend API.
- **Alerts:** Basic `window.alert()` calls on staff dashboards for error catching (e.g., "Error deleting code", "Not enough seats"). There are no "toast" libraries in use.
- **Dual-Source-of-Truth Risk:** Orders can be cancelled both via bulk action (`cancel_active_orders` RPC) and potentially individually. Table freeing logic is split between order placement (RPC), order billing (frontend logic executing DB updates), and manual clearing. If notifications are implemented in frontend logic rather than via generalized realtime events or DB triggers, there is a risk of missed notifications across different code paths.
