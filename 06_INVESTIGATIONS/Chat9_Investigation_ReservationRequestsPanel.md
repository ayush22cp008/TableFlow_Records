# Chat 9: Investigation — Reservation Requests Panel Mechanism

## 1. Query / Fetch Logic
- **File:** `app/dashboard/tables/page.tsx`
- **Query:** 
  ```ts
  supabase.from('reservation_requests')
    .select('*')
    .in('status', ['pending', 'approved', 'completed'])
    .order('requested_time')
  ```
- **Analysis:** It fetches all rows matching those 3 statuses, ordered chronologically. It does **not** limit the number of rows (no `.limit()`) and does **not** filter by date (e.g. today only).

## 2. Status Distribution (Database)
I ran a custom DB script to count the raw rows in `reservation_requests`.
**Total rows:** 50
- `completed`: 30
- `arrived`: 12
- `cancelled`: 5 (These are the 5 stale 'approved' rows that were fixed/backfilled by Node 4's migration)
- `rejected`: 3
*(Note: 'approved' is currently 0 because our Node 4 fix successfully cleaned them out).*

## 3. "Seated" Badge Meaning
- **File:** `app/dashboard/tables/page.tsx` (Line 358)
- **Logic:** `{req.status === 'completed' ? 'Seated' : 'Approved'}`
- **Conclusion:** The "Seated" label strictly corresponds to the DB status `'completed'`. A reservation becomes `'completed'` when the customer scans their code, accesses the cart, and officially places an order (meaning they are physically seated at the table).

## 4. Existing Sort / Limit / Pagination
- **Database Query:** Ordered by `requested_time` ASC. No limits, no pagination.
- **Client-Side Filtering (`visibleReservations` array):**
  There is a complex frontend filter that attempts to hide old entries, but it currently has a logic flaw:
  - `pending` requests are ALWAYS visible forever.
  - `approved` & `completed` requests attempt to find a `linkedOrder` (an order on the same table placed within $\pm$ 30 mins).
  - If a `linkedOrder` is found, the reservation stays visible in the panel **as long as the order is active** (not `billed`).
  - It is only hidden 5 minutes after the order finally becomes `billed`.

## 5. Functional Purpose of "Seated" Entries
**None.** 
Once a customer is seated (`completed` status) and their order is active, the staff uses the **Tables grid** to see table occupancy and the **Orders dashboard** to manage the meal. Keeping "Seated" entries visible in the Reservation Requests side-panel for the entire duration of their 1-2 hour meal serves absolutely no functional purpose. 

**Effect:** Because the client-side filter keeps `completed` entries visible for the entire lifecycle of the linked order, the panel gets flooded with a massive history of seated parties (30 rows), pushing any new `pending` requests out of view and creating extreme visual clutter.

**Status:** Investigation complete. Awaiting further instructions.
