# Investigation Results — Realtime Coverage Audit (Pre-Node 9)

| Page | Realtime Subscription | Auto-updates vs Manual Refresh | File Path |
| --- | --- | --- | --- |
| 1. Owner Dashboard (Live Orders) | Y (`orders_board`, `orders: *`) | Auto-updates | `app/dashboard/orders/page.tsx` |
| 2. Manager Dashboard | N | Needs manual refresh (button) | `app/dashboard/manager/page.tsx` |
| 3. Cook Dashboard | N | Needs manual refresh (button) | `app/dashboard/cook/page.tsx` |
| 4. Waiter Dashboard | N | Needs manual refresh (button) | `app/dashboard/waiter/page.tsx` |
| 5. Tables Page | Y (`tables_realtime`, `restaurant_tables: *`, `waitlist: *`, `reservation_requests: *`, `orders: *`) | Auto-updates | `app/dashboard/tables/page.tsx` |
| 6. Customer — My Orders | Y (`my_orders_realtime`, `orders: UPDATE`) | Auto-updates | `app/order/my-orders/page.tsx` |
| 7. Customer — Reservation Status | N | Needs manual refresh (on load) | `app/order/reservation/page.tsx` |
| 8. Customer — Menu/Ordering | Y (`menu_realtime`, `menu_items: *`) | Auto-updates | `app/order/page.tsx` |
| 9. Staff Management | N | Needs manual refresh (on load) | `app/dashboard/staff/page.tsx` |
| 10. Owner Menu Management | N | Needs manual refresh (on load) | `app/dashboard/menu/page.tsx` |
| 11. Owner Analytics | N | Needs manual refresh (on load) | `app/dashboard/analytics/page.tsx` |
| 12. Owner Insights | N | Needs manual refresh (on load) | `app/dashboard/insights/page.tsx` |
| 13. Owner Main Dashboard | N | Needs manual refresh (on load) | `app/dashboard/page.tsx` |
