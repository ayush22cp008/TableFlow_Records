# Chat #5 / Node 3 — Result: Manager Routing Stale Cache Fix

## Status
⏳ Build verification running in background.
⏳ Awaiting manual UI/Browser testing by Ayush.
⏳ Pending Git Commit & Push (Manual trigger required).

## Changes Implemented

### 1. Fix `app/auth/callback/route.ts` (Google Sign-In Path)
Removed the redundant `supabase.auth.getUser()` call that was reading the stale incoming request cookie. The route now directly consumes the fresh `user` object returned by `exchangeCodeForSession()`. This resolves **Outcome 1 (Owner Dashboard)** where the login flow used the identity of a previously logged-in session.

### 2. Fix Next.js fetch caching (Manual Signup Path & General Middleware)
Updated the Supabase server client constructor to enforce bypassing the aggressive Next.js Edge fetch cache:
```typescript
global: {
  fetch: (url, options) => fetch(url, { ...options, cache: 'no-store' })
}
```
This forces `middleware.ts` to always hit the database for the freshest role. This resolves **Outcome 2 (Customer `/order` page)** where the middleware redirected based on a cached version of the user's profile before their role was upgraded. It also solidifies **Outcome 3** so the correct Manager Dashboard is consistently loaded.

## Scope Check Results
As requested, I searched the entire codebase for `createServerClient`. Beyond the two named files, it is also constructed in:
- `app/api/auth/verify-invite/route.ts`
I applied the exact same `cache: 'no-store'` fix to this location as well to ensure total consistency across all Server/Edge client instances.

## Files Touched
- `[MODIFY]` app/auth/callback/route.ts
- `[MODIFY]` middleware.ts
- `[MODIFY]` lib/supabase-server.ts
- `[MODIFY]` app/api/auth/verify-invite/route.ts
