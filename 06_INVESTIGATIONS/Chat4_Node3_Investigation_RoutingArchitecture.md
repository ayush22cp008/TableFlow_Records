# Routing Architecture Investigation (Node 3)

## Overview
This report analyzes the existing routing and authentication flow to determine the best approach for implementing the `waiter`, `cook`, and `manager` dashboards.

## Option 1: Role-Specific Routes (e.g., `/waiter`, `/cook`, `/manager` or `/dashboard/cook`)

**Current Codebase Alignment:**
The codebase heavily leans toward role-specific routing rather than a shared view. 
- **`app/auth/callback/route.ts` (Lines 34-37):** Explicitly branches based on the `owner` role, redirecting owners to `/dashboard` and everyone else to `/order`.
- **`middleware.ts` (Lines 63-67):** Similar logic; logging in sends owners to `/dashboard` and others to `/order`.
- **`types/index.ts` (Line 5):** `UserRole` is fully defined as `'customer' | 'owner' | 'waiter' | 'cook' | 'manager'`, supporting explicit role branching.
- **`app/dashboard/page.tsx`:** This is currently hardcoded as an "Owner Dashboard" with all owner-specific links (`/dashboard/staff`, `/dashboard/analytics`, etc.) hardcoded in a mapped array.

**Effort Estimate:**
**Low.** Extending the current pattern is straightforward. We simply add new cases to `middleware.ts` and `app/auth/callback/route.ts` (e.g., `if (role === 'waiter') return NextResponse.redirect(new URL('/waiter', request.url))`), and create distinct `page.tsx` files for each role.

**Risks:**
- Minimal risk. Route-level isolation prevents UI components intended for an owner from accidentally rendering for a waiter or cook.
- Requires duplicating some layout/wrapper code for each dashboard, but keeps concerns cleanly separated.

---

## Option 2: Shared `/dashboard` with Conditional Rendering

**Current Codebase Alignment:**
The current codebase is **not** aligned with this pattern.
- `app/dashboard/page.tsx` currently assumes the user is an owner. There is no conditional logic for other roles.
- `middleware.ts` currently blocks customers from `/dashboard`. If we used a shared dashboard, we would have to allow staff in, but then rely entirely on React conditional rendering to hide owner features.

**Effort Estimate:**
**High.** Retrofitting the existing `app/dashboard/page.tsx` would require wrapping the entire owner UI in role checks (`if (role === 'owner') return <OwnerView />`), and importing/creating separate views for each staff role inside the same file. It also requires modifying the existing sub-routes (like `/dashboard/orders`) if they are to be shared, ensuring they conditionally hide owner-only buttons based on the user's role context.

**Risks:**
- **Security/UX Risk:** Higher risk of accidentally exposing an owner-level feature to a waiter if a conditional check is missed in the shared UI.
- **Maintenance:** The `app/dashboard/page.tsx` file would become massive and difficult to maintain as all 4 staff roles (owner, manager, cook, waiter) get crammed into one page.

## Conclusion & Recommendation
The evidence strongly supports **Option 1: Role-specific routes**. It is exactly how the codebase currently distinguishes between `owner` and `customer`, and it provides the cleanest, safest isolation for staff roles.
