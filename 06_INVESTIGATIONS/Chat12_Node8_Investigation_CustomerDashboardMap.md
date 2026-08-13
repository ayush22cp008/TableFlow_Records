# Investigation Result: Full Customer Dashboard Map (Backend + Navigation)

## 1. Route Inventory

- **`/` (Home):** Landing page with marketing text.
- **`/order` (Menu):** Displays live `menu_items`. Has category filters. Users can add items to their local cart here.
- **`/order/cart` (Cart & Checkout):** Displays cart items from `localStorage`. Auto-finds a table or checks the reservation code. Submits order to backend or places user on waitlist if no tables are free. Shows waitlist status screens if currently waiting or seated.
- **`/order/my-orders` (My Orders):** Displays all past and active orders for the logged-in customer. Includes live status updates and allows submitting feedback for served/billed orders.
- **`/reserve` (Reserve a Table):** A form for requesting a table reservation (Name, Party Size, Time).
- **`/reserve/status` (Check Reservation Status):** A page to search for your reservation by name and view its status (pending/approved/etc.) and get your unique entry code.
- **`/login` & `/signup`:** Standard Supabase authentication pages.

## 2. Navigation Graph

- **`/` (Home)**
  - Links to: `/order` (Browse Menu), `/login` / `/signup`
- **`/order` (Menu)**
  - Links to: `/reserve` (Reserve a Table button), `/order/cart` (Cart button, if items in cart)
- **`/order/cart`**
  - Links to: `/order/my-orders` (auto-redirects on successful order placement), `/order` (Back to Menu from waitlist screens)
- **`/order/my-orders`**
  - Links to: `/order` (if no orders exist)
- **`/reserve`**
  - Links to: `/reserve/status` (via "Check Status" button after successful submission)
- **`/reserve/status`**
  - Links to: Nowhere specific (aside from global Navbar)
- **Global Navbar (Customer)**
  - Links to: `/`, `/order`, `/order/my-orders`, `/login`, Sign Out

## 3. Backend Connection per Route

| Route | Supabase Table(s) / RPC(s) | Customer-Scoped? (`user_id` / `profiles`) |
| :--- | :--- | :--- |
| `/order` | `menu_items` (Select) | **No** (Anonymous access, filters by `is_available` and `is_active`) |
| `/order/cart` | `restaurant_tables` (Select), `reservation_requests` (Select unique code), `place_order_and_occupy_table` (RPC), `order_items` (Insert), `waitlist` (Select/Insert), `profiles` (Select) | **Yes** (Order placement and waitlist entry strictly require `user.id`. `placeOrder` forces `/login` if not authenticated) |
| `/order/my-orders` | `orders` (Select), `feedback` (Insert via `FeedbackButtons.tsx`) | **Yes** (Filters by `customer_id = user.id`) |
| `/reserve` | `reservation_requests` (Insert) | **No** (Fully anonymous. Inserts name as plain text) |
| `/reserve/status`| `reservation_requests` (Select) | **No** (Fully anonymous. Queries using `ilike('customer_name', '%name%')`) |

## 4. Owner-Side Connection Points

- **`/order` (Menu) ↔** `app/dashboard/menu/page.tsx` (Owner/Manager manage the items displayed here).
- **`/order/cart` (Checkout & Waitlist) ↔** `app/dashboard/tables/page.tsx` (Owner sees table capacity update, sees waitlist queue, manually seats waitlisted customers). `app/dashboard/orders/page.tsx` (Owner/Staff sees the new orders on the Live Orders board).
- **`/order/my-orders` (Orders & Feedback) ↔** `app/dashboard/orders/page.tsx` (Owner/Staff update the order statuses which reflect here). `app/dashboard/insights/page.tsx` (Owner views AI insights generated from the feedback submitted here).
- **`/reserve` & `/reserve/status` (Reservations) ↔** `app/dashboard/tables/page.tsx` (Owner views pending requests, approves them, and assigns tables, which updates the status seen by the customer).

## 5. Auth/Identity Check

- **Order Flow (`/order/cart`, `/order/my-orders`):** The customer is **definitely logged in** with a stable `user_id` linked to the `profiles` table. The checkout process hard-blocks anonymous users and forces them to `/login`.
- **Reservation Flow (`/reserve`, `/reserve/status`):** This flow is **completely anonymous**. It does not enforce login, does not record a `user_id`, and relies entirely on matching a manually typed string (`customer_name`). 

**Conclusion for Node 8 linking:** Because reservation requests are currently anonymous, any attempt to link a customer's active reservation automatically to their dashboard (without them searching for their name) will require adding a `customer_id` column to `reservation_requests` and enforcing/capturing auth during the `/reserve` flow.
