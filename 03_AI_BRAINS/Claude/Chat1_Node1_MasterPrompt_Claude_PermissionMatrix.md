# TableFlow — Staff Role System — Permission Matrix (LOCKED)

Chat #1 | Node 1 | Status: Spec locked, implementation NOT started

## Order Status Lifecycle

| Status | Who transitions it |
|---|---|
| Placed | Customer |
| Preparing | Manager (accepts into kitchen queue) |
| Ready | Cook |
| Served | Waiter |
| Completed/Paid | Manager (billing/closure) |

## Full Access Matrix

| Resource | Waiter | Cook | Manager | Owner | Customer |
|---|---|---|---|---|---|
| Orders (create) | — | — | — | — | R/W (own) |
| Order items (kitchen ticket: item+qty only, no table) | — | R | — | — | — |
| Order status: Placed→Preparing | — | — | R/W | — | — |
| Order status: Preparing→Ready | — | R/W | — | — | — |
| Order status: Ready→Served | R/W | — | — | — | — |
| Order status: Completed/Paid | — | — | R/W | R/W | — |
| Order status (view only) | R | R | R | R | R (own) |
| Tables (manage) | R/W | — | R/W | R/W | — |
| Tables (status board: Reserved/Free) | R | — | R | R | R |
| Reservations (manage) | R/W | — | R/W | R/W | — |
| Reservation own detail (code/table/order) | — | — | — | — | R (own) |
| Bills (generate/close) | — | — | R/W | R/W | R (own, view only) |
| Menu (manage) | — | — | R/W | R/W | — |
| Menu (browse) | R | — | R | R | R |
| Sales Analytics | — | — | — | R (Owner-only) | — |
| AI Menu Insights | — | — | — | R (Owner-only) | — |
| Staff management | — | — | — | R/W (Owner-only) | — |
| Restaurant settings | — | — | — | R/W (Owner-only) | — |

## Role Summary

- **Customer**: places orders/reservations, views own live order + table status board (Reserved/Free). Reservation customers additionally see verification code. Cannot self-change order status.
- **Cook**: kitchen-ticket view only (item+qty, no table/customer info). Owns Preparing→Ready transition only.
- **Waiter**: floor service — manages tables, reservations. Owns Ready→Served transition only. No billing/money access.
- **Manager**: order gatekeeper — owns Placed→Preparing (intake) AND Completed/Paid (billing closure). Full R/W on orders/tables/reservations/bills/menu. No access to analytics, staff management, or restaurant settings.
- **Owner**: strategic-only — Sales Analytics, AI Menu Insights, Staff management (create/edit staff accounts), Restaurant settings (domain, OAuth, etc.). No self-signup/invite-code for staff — Owner manually creates all staff accounts.

## Customer-Side New Feature (Table Status Board)

**Top-level (any logged-in customer):** Table number + status only (Reserved/Free) — no code, no name, no order info.

**Detail view (own reservation/order only):**
- Reservation customer: table number, verification code, order number, live placed items
- Walk-in customer: table number, order number, live placed items (no code)

Both variants: view-only for order status, no self-transition rights.

## Access Model (pre-decided, unchanged)

- Granular RLS enforcement per role
- Owner manually creates/manages staff accounts — no self-signup, no invite-code flow for staff
- Permission matrix above is the single source of truth for all RLS policy design — do not infer or assume beyond this

## Next Action

Begin schema/RLS implementation for the Staff Role System based on this locked matrix. Suggested order: staff role enum/table → RLS policies per resource → status transition enforcement → customer-side table status board UI.
