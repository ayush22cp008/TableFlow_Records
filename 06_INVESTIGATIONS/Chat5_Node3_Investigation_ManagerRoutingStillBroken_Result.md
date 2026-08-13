# Investigation Result: Manager Routing Still Broken (Chat 5 / Node 3)

## 1. Exact Code Paths & Where Role is Read

### Google Sign-In Path (`app/auth/callback/route.ts`)
```typescript
    const { error } = await supabase.auth.exchangeCodeForSession(code)
    
    if (!error) {
      // Get user to check metadata and profile
      const { data: { user } } = await supabase.auth.getUser()
```
**Where it reads:** The code exchanges the auth code for a session (which calls `cookies().set()` to write the new session). However, it then immediately calls `supabase.auth.getUser()`. In Next.js, `cookies().getAll()` only returns the *incoming* request cookies; it does not reflect cookies that were just set in the same request lifecycle. 
**The Bug:** `getUser()` reads the stale incoming cookie. If Ayush previously had an Owner session, `getUser()` returns the Owner user object. The script then fetches the Owner's profile and redirects to `/dashboard`.

### Manual Signup Path (`app/auth/select-role/page.tsx` -> `middleware.ts`)
The client explicitly updates the DB and navigates to `/dashboard/manager`. This triggers a fresh page request, hitting `middleware.ts`:
```typescript
    const { data: profile } = await supabase
      .from('profiles')
      .select('role')
      .eq('id', user.id)
      .single()

    const role = profile?.role ?? 'customer'
```
**Where it reads:** `middleware.ts` extracts the correct user from the fresh cookie and queries the DB.
**The Bug:** Supabase uses `fetch` under the hood. Next.js aggressively caches `fetch` requests in the Edge runtime. If the user hit any protected route *before* selecting their role, Next.js cached their profile as `role = 'customer'`. When `middleware.ts` queries the DB again after the role update, Next.js intercepts the `fetch` and serves the cached `customer` role. The middleware then redirects the "customer" to `/order`.

## 2. Diagnosis of the 3 Outcomes
- **Outcome 1 (Owner Dashboard):** Occurs on the Google Sign-In path. `route.ts` reads the stale incoming cookie of a previous Owner session instead of using the newly authenticated user.
- **Outcome 2 (Customer `/order` page):** Occurs on the Manual Signup path. `middleware.ts` queries the DB, but Next.js serves a cached `fetch` response showing the user still has the default `customer` role.
- **Outcome 3 (Manager Dashboard loading briefly):** Occurs when the Next.js `fetch` cache happens to miss or expire, allowing `middleware.ts` to fetch the fresh `manager` role from the database.

## 3. Proposed Fix (Pending Approval)
1. **Fix `app/auth/callback/route.ts`:**
   Instead of calling `supabase.auth.getUser()`, directly use the user object returned by `exchangeCodeForSession(code)`:
   ```typescript
   const { data: { user }, error } = await supabase.auth.exchangeCodeForSession(code)
   // Remove the redundant and buggy getUser() call
   ```
2. **Fix `middleware.ts` and `lib/supabase-server.ts`:**
   Force Supabase to bypass the Next.js fetch cache by passing a custom global fetch configuration to `createServerClient`:
   ```typescript
   global: {
     fetch: (url, options) => fetch(url, { ...options, cache: 'no-store' })
   }
   ```
   
Please confirm if this fix approach is approved!
