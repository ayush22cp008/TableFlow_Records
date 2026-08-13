# Instruction: Reservation Nav Move + Customer-Linked Status Page

**Node:** 8 — Customer Dashboard Revamp
**Type:** FIX — Frontend only. Requires `customer_id` migration (previous instruction) to be done first.

## Prerequisite
Confirm `reservation_requests.customer_id` column exists before starting this.

## Changes required

### 1. Navbar
Move "Reserve a Table" out of the Menu page (`/order`) into the global customer navbar: `Menu | My Orders | Reservation | Sign Out`. Remove the small pill button currently inside `/order`.

### 2. Reservation submit — capture customer_id
In the `/reserve` submission logic, save `customer_id: user.id` (from the already-authenticated session) alongside the existing `customer_name` field. Do not remove `customer_name` — keep both.

### 3. New page: `/order/reservation`
Create a new page at `/order/reservation` (replacing the old `/reserve` + `/reserve/status` combo for logged-in customers):
- Query: `reservation_requests` filtered by `customer_id = user.id`, most recent one only (order by `requested_time` or `created_at` desc, limit 1). Join `restaurant_tables(table_number)` for display.
- Display: table number, party size, status (`pending | approved | rejected | arrived | completed | cancelled`), requested time.
- **No cancel/modify button** — out of scope per Node 8 decision.
- If no reservation found (or `customer_id` is NULL for old data), show a simple "no active reservation" state with a way to create a new one (reuse existing `/reserve` form logic, just re-pointed/embedded here).
- Optional: auto-fill name field from `profiles` on the reservation form, since login is already mandatory.

### 4. Old routes
Decide whether `/reserve` and `/reserve/status` become redirects to `/order/reservation` or are deprecated — flag this as a question back to Ayush if unclear, don't assume.

## Output
Confirm build passes, list files touched, and flag the old-route decision (point 4) explicitly for Ayush if not resolved. No push — wait for explicit go-ahead per project rule.
