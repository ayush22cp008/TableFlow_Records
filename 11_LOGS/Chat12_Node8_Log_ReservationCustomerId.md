# Manual DB Change Log: reservation_requests.customer_id

- **Timestamp:** 2026-08-11T03:36:00+05:30
- **Table:** `reservation_requests`
- **Change:** Added column `customer_id UUID REFERENCES profiles(id)` (Nullable)
- **Reason:** Node 8 — Customer Dashboard Revamp. Needed to link reservations to authenticated customers so they can view their status automatically on their own dashboard.
- **Query:** 
  ```sql
  ALTER TABLE reservation_requests
  ADD COLUMN customer_id UUID REFERENCES profiles(id);
  ```
