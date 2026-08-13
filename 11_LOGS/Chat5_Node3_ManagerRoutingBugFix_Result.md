# Chat #5 / Node 3 — Result: Manager Routing Bug Fix

## Status
⏳ Build verification running in background.
⏳ Awaiting manual UI/Browser testing by Ayush.
⏳ Pending Git Commit & Push (Manual trigger required).

## Changes Implemented
1. **Direct Fix (`app/auth/select-role/page.tsx`):**
   - Updated line 76 to properly route `role === 'manager'` to `/dashboard/manager` instead of `/dashboard` after invite verification.
2. **Safeguard Added (`middleware.ts`):**
   - Added a new route guard. If a user with the role of `manager`, `cook`, or `waiter` explicitly navigates to the exact `/dashboard` path, they are now automatically redirected to their specific dashboard (`/dashboard/${role}`). The Owner role is unaffected and will still load `/dashboard` normally.

## Waiter Gap Found!
As requested, I checked the `select-role` redirect logic for the `waiter` branch. It currently falls through to the `/order` route:
```typescript
window.location.href = role === 'manager' ? '/dashboard/manager' : role === 'cook' ? '/dashboard/cook' : '/order'
```
Because the Node 3 Waiter Dashboard is not started yet, I have flagged this. A `waiter` finishing signup will currently land on the `/order` page instead of a dashboard.

## Files Touched
- `[MODIFY]` app/auth/select-role/page.tsx
- `[MODIFY]` middleware.ts
