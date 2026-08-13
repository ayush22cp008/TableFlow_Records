# Reservation Label Bug Investigation

As requested, I have scanned the TableFlow codebase to investigate the logic and state management surrounding reservation labels.

## 1. Where Reservation Labels are Computed and Displayed

The primary logic for computing and displaying the reservation label is located on the **Owner-side Tables Dashboard**.
There does not appear to be a customer-facing visual table map that renders these text labels, though the reservation state is checked during order placement in `app/order/cart/page.tsx`.

*   **File:** `app/dashboard/tables/page.tsx`
*   **Function:** `getTableDisplay(table: RestaurantTable)` (Lines 17-37)
*   **Rendered at:** `app/dashboard/tables/page.tsx` (Line 228) inside the table grid: `<div className="text-xs mt-1 font-medium">{label}</div>`

## 2. Possible Label Values and Logic

The `getTableDisplay` function dictates the label and color styling using the following conditional cascade:

1.  **Time-Based Reservation:**
    *   **Logic:** `if (table.reserved_from)` AND `Date.now() >= reservedTime - 30 * 60 * 1000`
    *   **Label:** `"Reserved for [Time]"` (e.g., "Reserved for 07:30 PM")
    *   **Style:** Purple (`bg-purple-500/20 text-purple-300`)
2.  **Manual Override Reservation:**
    *   **Logic:** `else if (table.status === 'reserved')`
    *   **Label:** `"Reserved"`
    *   **Style:** Purple (`bg-purple-500/20 text-purple-300`)
3.  **Available:**
    *   **Logic:** `else if ((table.occupied_seats || 0) === 0)`
    *   **Label:** `"Available"`
    *   **Style:** Green (`bg-green-500/20 text-green-300`)
4.  **Partially Occupied:**
    *   **Logic:** `else if ((table.occupied_seats || 0) < table.capacity)`
    *   **Label:** `"Partially Occupied"`
    *   **Style:** Orange (`bg-orange-500/20 text-orange-300`)
5.  **Full:**
    *   **Logic:** `else`
    *   **Label:** `"Full"`
    *   **Style:** Red (`bg-red-500/20 text-red-300`)

## 3. Findings: Bugs and Stale States

I identified a significant **stale state / conditional bug** in the time-based reservation logic:

> [!WARNING]
> **Permanent "Reserved" State (No Upper Bound)**
> The condition `Date.now() >= reservedTime - 30 * 60 * 1000` correctly activates the reservation label 30 minutes *before* the reserved time. However, because it only checks if the current time is greater than that threshold, **it remains true forever into the future**.
> 
> If a reservation time passes and `table.reserved_from` is never explicitly cleared, the table will be permanently labeled as `"Reserved for [Time]"` on the owner's dashboard.

> [!IMPORTANT]
> **Missing State Clearance on Arrival**
> In `app/dashboard/tables/page.tsx` (Line 125, `confirmArrival` function), when the owner clicks "Confirm Arrival" for a customer, it updates the `reservation_requests` status to `'arrived'`, but it **does not clear `table.reserved_from`**. As a result, the table continues to show the "Reserved for [Time]" label even after the customer has arrived and sat down.

*Note: The `reserved_from` field is eventually cleared if the customer places an order via the cart (`app/order/cart/page.tsx`, Line 182), but if they never order (e.g. walk out) or just haven't ordered yet, the dashboard UI remains stale.*

## 4. Relevant File Paths and Line Numbers

1.  **Label Display Logic:** 
    *   [app/dashboard/tables/page.tsx](file:///C:/Users/ayush/Desktop/vibethon_project/app/dashboard/tables/page.tsx#L17-L37) (Lines 17-37)
2.  **Label Rendering in UI:** 
    *   [app/dashboard/tables/page.tsx](file:///C:/Users/ayush/Desktop/vibethon_project/app/dashboard/tables/page.tsx#L217) (Line 217 - extraction)
    *   [app/dashboard/tables/page.tsx](file:///C:/Users/ayush/Desktop/vibethon_project/app/dashboard/tables/page.tsx#L228) (Line 228 - render)
3.  **Confirm Arrival (Missing Clear Logic):** 
    *   [app/dashboard/tables/page.tsx](file:///C:/Users/ayush/Desktop/vibethon_project/app/dashboard/tables/page.tsx#L125-L134) (Lines 125-134)
4.  **Order Cart (Where it is actually cleared):**
    *   [app/order/cart/page.tsx](file:///C:/Users/ayush/Desktop/vibethon_project/app/order/cart/page.tsx#L182) (Line 182)
