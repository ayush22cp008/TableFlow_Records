# Cook Dashboard Implementation Plan

This plan implements the new Cook Dashboard as specified in the instruction, giving cooks a focused Kitchen Display System (KDS) to mark preparing orders as ready.

## Proposed Changes

### 1. Routing & Middleware Updates
We will add a specific routing branch for the `cook` role to ensure they land on `/dashboard/cook` instead of the customer `/order` page after login or sign up.

#### [MODIFY] [middleware.ts](file:///c:/Users/ayush/Desktop/vibethon_project/middleware.ts)
- Update the `/login` and `/signup` redirect logic to check if the role is `cook` and redirect to `/dashboard/cook`.

#### [MODIFY] [app/auth/callback/route.ts](file:///c:/Users/ayush/Desktop/vibethon_project/app/auth/callback/route.ts)
- Update the OAuth callback redirect logic to handle `cook` -> `/dashboard/cook`.

#### [MODIFY] [components/AuthForm.tsx](file:///c:/Users/ayush/Desktop/vibethon_project/components/AuthForm.tsx)
- Update the client-side `window.location.href` redirects in `handleVerifyOtp`, `handleLogin`, and `handleStaffSignup` to respect the `cook` role.

---

### 2. Cook Dashboard Page

#### [NEW] [app/dashboard/cook/page.tsx](file:///c:/Users/ayush/Desktop/vibethon_project/app/dashboard/cook/page.tsx)
We will build the Cook Dashboard using the kanban card styling from the existing live orders board, but heavily stripped down for the kitchen context.

**Features:**
- **Data Fetching:** Fetch orders where `status = 'preparing'` utilizing `.select('*, order_items(quantity, menu_items(name))')`.
- **UI:** A grid of cards displaying the order short ID and the list of items (`{quantity}x {name}`). We will explicitly hide prices, table numbers, and customer information per the requirements.
- **Action:** A single "Mark Ready" button that advances the order status to `ready`. This uses the existing `cook_prep_to_ready` RLS policy.
- **Refresh:** A manual "Refresh Queue" button that triggers a re-fetch. **No realtime polling or subscriptions** are included in this phase.

## Verification Plan

### Automated Verification
- Run `npm run build` locally to verify 0 build errors.

### Manual Verification
- After deployment (or locally using dev server), sign in as a user with the `cook` role.
- Verify the post-login redirect lands correctly on `/dashboard/cook`.
- Verify the cards show only the required information (ID + items) without prices or table numbers.
- Verify the "Mark Ready" button successfully changes the status of a preparing order to `ready`.
