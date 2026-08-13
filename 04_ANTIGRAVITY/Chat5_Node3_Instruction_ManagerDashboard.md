# TableFlow — Chat #5 / Node 3 — Instruction (Build): Manager Dashboard

**To:** Antigravity
**From:** Claude
**Type:** Build — implement per locked spec below (self-contained; no separate master prompt this session)

---

## Context

Third of three role dashboards (Cook and Waiter patterns already exist/locked). Manager is the **order gatekeeper**: owns intake (Placed→Preparing) AND billing closure (Completed/Paid), plus full R/W on tables, reservations, menu. No analytics/staff/settings access.

## Grounding — Permission Matrix (Manager row, locked Node 1)

| Resource | Manager access |
|---|---|
| Order status: Placed → Preparing | R/W (intake) |
| Order status: Completed/Paid | R/W (billing closure) |
| Order status (all other states) | R (view only) |
| Tables (manage) | R/W |
| Reservations (manage) | R/W |
| Bills (generate/close) | R/W |
| Menu (manage) | R/W |
| Menu (browse) | R |
| Sales Analytics, AI Insights, Staff Mgmt, Settings | No access |

## Decisions Locked This Session (Chat #5)

1. **Payment method tracking:** simple dropdown (Cash / Card / UPI) captured when Manager marks an order Paid. No payment gateway integration — this is a record field only, not a transaction processor.
2. **Bill detail:** itemized (items, qty, price, line total, grand total) shown on-screen.
3. **Bill export:** downloadable/printable PDF.
4. **Bulk actions:** none for Manager — per-order actions only (matches Cook/Waiter pattern; Owner keeps sole bulk emergency-stop capability).

## Build Tasks

### 1. Routing
- `app/auth/callback/route.ts` — add branch: `if (role === 'manager') redirect to '/dashboard/manager'`
- `middleware.ts` — add matching `manager` branch alongside existing `owner`/`cook` logic

### 2. Schema change (do first, separately verify before UI work)
- Add `payment_method` column to `orders` table — text/enum: `'cash' | 'card' | 'upi'`, nullable (only set when order reaches Paid)
- This is a manual DB change — per engineering discipline rule, commit the migration file immediately, don't defer

### 3. New page: `app/dashboard/manager/page.tsx`
Two sections/tabs:

**A. Intake queue** — orders where `status = 'placed'`
- Query: `.select('*, order_items(quantity, price, menu_items(name)), restaurant_tables(table_number)')`
- Card shows: order ID, table number, itemized list, total
- Action: "Accept" button → `orders.status = 'preparing'`

**B. Billing queue** — orders where `status = 'served'`
- Same query shape
- Card shows: itemized bill (item, qty, price, line total, grand total), table number
- Action: payment method dropdown (Cash/Card/UPI) + "Mark Paid" button → sets `orders.payment_method` and `orders.status = 'completed'` together in one update (single source of truth — don't split into two calls)
- "Download PDF" button per bill — generate itemized PDF client-side (check if a PDF lib is already in the project; if not, lightweight option only, don't add heavy new deps for this)

### 4. Reuse existing pattern
Base card/query structure on `app/dashboard/orders/page.tsx` Kanban pattern, same as Cook dashboard did. Manager version keeps table number, price, and full item detail (unlike Cook's stripped kitchen-ticket view) since Manager's permission row allows it.

### 5. Tables/Reservations/Menu management
Manager needs R/W access to existing Tables, Reservations, and Menu management UI — confirm whether these already exist as shared components/pages the Owner uses (likely `app/dashboard/tables`, `app/dashboard/menu`) and whether Manager role can already reach them via existing RLS, or if routing/nav needs to expose them to Manager too. **Investigate and report before building new UI** — don't duplicate if reusable.

## Explicitly Do Not Build

- Auto-refresh/polling (Node 4, later)
- Bulk actions of any kind
- Payment gateway integration (real transaction processing)
- Analytics, staff management, or settings access for Manager

## Verification Before Reporting Back

- `npm run build` — 0 errors
- Confirm RLS: Manager role can perform `placed→preparing` and `served→completed` transitions (check existing policies cover this per Permission Matrix; flag if a new RLS policy is needed — do not write it yourself, report back first per investigation/fix separation rule)
- Confirm `payment_method` write only happens paired with the `completed` status update (no dual-write)

## Output

Do NOT push to GitHub yet — push is manually triggered by Ayush after review. Report back with: files touched, build result, and findings on Task 5 (existing Tables/Reservations/Menu reuse). Ayush will test live after Vercel deploy, per evidence rule.
