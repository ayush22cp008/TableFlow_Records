# Investigation Results: Reservation Code Expiry Mechanism

Chat #3 | Node 2b

**1. Table/Column**
- **Table:** `reservation_requests`
- **Column:** `requested_time` (timestamptz). 
- There is no explicit `expires_at` column; the expiration is dynamically calculated as `requested_time + 30 minutes`.

**2. Mechanism Type**
- **Client-Side Filter / Runtime Validation.** 
- There is no pg_cron job, no Supabase Edge Function, and no database trigger handling the expiration. It is entirely enforced by frontend Next.js code when reading or validating the data.

**3. Deletion Logic**
- **No deletion occurs.** 
- Expired rows are *never* deleted from the database. If a customer never uses their reservation code, the row sits in the database permanently with `status = 'approved'`. It is simply "hidden" and "blocked" at the application layer.

**4. Location & Code Snippets**
The expiry is enforced in two specific files:

*   **A. Cart Validation (Blocking use of expired codes):**
    File: `app/order/cart/page.tsx` (Lines 92-97)
    ```typescript
    const requestedTimeMs = new Date(data.requested_time).getTime()
    if (Date.now() > requestedTimeMs + 30 * 60 * 1000) {
      setError('Invalid or unused code, or missing table assignment.')
      setVerifyingCode(false)
      return
    }
    ```

*   **B. Dashboard UI (Auto-hiding expired requests from Staff):**
    File: `app/dashboard/tables/page.tsx` (Lines 200-218)
    ```typescript
    const requestedTime = new Date(req.requested_time).getTime()
    // ... [checks if there is a linked order, if none:]
    return now <= requestedTime + 30 * 60 * 1000
    ```

**5. Frequency**
- **N/A.** Since it is purely a runtime check on the client/Next.js side, it is evaluated instantly every time a user views the dashboard or attempts to enter a code in the cart. There is no background schedule.

**Conclusion for `invite_codes`:**
Yes, this mechanism is highly reusable. We do not need to over-engineer a `pg_cron` or Edge Function for staff `invite_codes`. We can simply store the code in the DB (optionally with an `expires_at` column) and have the signup/invite verification logic reject the code if `Date.now() > expires_at`, just like the cart does for reservations.
