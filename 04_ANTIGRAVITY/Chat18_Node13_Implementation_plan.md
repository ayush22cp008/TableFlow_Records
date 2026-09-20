# Goal: Implement Node 13 (Cook/Waiter Order Claiming)

This implementation plan strictly follows the provided Node 13 specification to introduce exclusive order claiming for Cooks and Waiters. It will enforce one-order-at-a-time rules at the database level and replace direct status updates with atomic RPCs.

## Proposed Changes

### Database Migration

#### [NEW] [20260920000001_node13_order_claiming.sql](file:///C:/Users/ayush/Desktop/vibethon_project/supabase/migrations/20260920000001_node13_order_claiming.sql)
- **Schema changes:** Add `claimed_by_cook_id` and `claimed_by_waiter_id` to `orders`. Add `staff_name` to `profiles`.
- **Indexes:** Create partial unique indexes on the claim columns to enforce one active claim per staff member.
- **RPCs:** Create 4 atomic RPCs (`claim_order_as_cook`, `complete_order_as_cook`, `claim_order_as_waiter`, `complete_order_as_waiter`) using `FOR UPDATE` locking on both the profile and the order to prevent race conditions.
- **Notifications:** Inside the claim RPCs, insert an `order_claimed` notification for the active Manager.
- **RLS Adjustments:** Drop the existing `cook_prep_to_ready` and `waiter_ready_to_served` policies to close the bypass. Claim/complete will only be possible via the new RPCs. Cancellation policies remain untouched.

---

### Backend API

#### [MODIFY] [route.ts](file:///C:/Users/ayush/Desktop/vibethon_project/app/api/auth/staff-signup/route.ts)
- Persist `staff_name: codeData.staff_name` during both the re-activation (existing profile) and new signup branches.

---

### Types

#### [MODIFY] [index.ts](file:///C:/Users/ayush/Desktop/vibethon_project/types/index.ts)
- Add `order_claimed` to `NotificationType`.
- Update `Order` type to include `claimed_by_cook_id`, `claimed_by_waiter_id`, and joined profile relations for Manager visibility.
- Add `staff_name` to `Profile`.

---

### Frontend UI

#### [MODIFY] [page.tsx (Cook)](file:///C:/Users/ayush/Desktop/vibethon_project/app/dashboard/cook/page.tsx)
- Update query to select `claimed_by_cook_id`.
- Show a "Claim" button for unclaimed orders.
- Show "Mark Ready" only if the order is claimed by the current Cook.
- Grey out orders claimed by other Cooks.
- Replace direct `supabase.from('orders').update(...)` calls with RPC calls to `claim_order_as_cook` and `complete_order_as_cook`.

#### [MODIFY] [page.tsx (Waiter)](file:///C:/Users/ayush/Desktop/vibethon_project/app/dashboard/waiter/page.tsx)
- Update query to select `claimed_by_waiter_id`.
- Add similar "Claim" and "Mark Served" UI states as Cook.
- Replace direct updates with RPC calls to `claim_order_as_waiter` and `complete_order_as_waiter`.

#### [MODIFY] [page.tsx (Manager)](file:///C:/Users/ayush/Desktop/vibethon_project/app/dashboard/manager/page.tsx)
- Update the order fetching query to join `profiles` on `claimed_by_cook_id` and `claimed_by_waiter_id`.
- Display the assigned Cook's or Waiter's name and email on the order cards in the UI for visibility.

## Verification Plan
### Automated Tests
- Run `npm run build` to verify type safety and component builds.
### Manual Verification
- Execute the SQL migration manually in Supabase.
- Log in as Cook A and Waiter A to verify claiming blocks other staff.
- Ensure only one active claim is permitted.
- Check Manager dashboard for assignment visibility and new `order_claimed` notification.
- Verify cancellation still successfully clears claims.
