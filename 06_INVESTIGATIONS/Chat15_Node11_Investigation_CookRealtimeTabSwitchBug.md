# Investigation Results — Node 11: Cook Dashboard Realtime Dies on Tab Switch

## 1. Compare Subscription Setup Across All 3 Files

The subscription setup is structurally **identical** across all 3 files. There are no differences in how they are implemented.

**app/dashboard/manager/page.tsx**
```tsx
  useEffect(() => {
    fetchOrders()
    const channel = supabase.channel('manager_orders_realtime')
      .on('postgres_changes', { event: '*', schema: 'public', table: 'orders' }, fetchOrders)
      .subscribe()
    return () => { supabase.removeChannel(channel) }
  }, [fetchOrders])
```

**app/dashboard/cook/page.tsx**
```tsx
  useEffect(() => {
    fetchPreparingOrders()
    const channel = supabase.channel('cook_orders_realtime')
      .on('postgres_changes', { event: '*', schema: 'public', table: 'orders' }, fetchPreparingOrders)
      .subscribe()
    return () => { supabase.removeChannel(channel) }
  }, [fetchPreparingOrders])
```

**app/dashboard/waiter/page.tsx**
```tsx
  useEffect(() => {
    fetchReadyOrders()
    const channel = supabase.channel('waiter_orders_realtime')
      .on('postgres_changes', { event: '*', schema: 'public', table: 'orders' }, fetchReadyOrders)
      .subscribe()
    return () => { supabase.removeChannel(channel) }
  }, [fetchReadyOrders])
```

**Observations:**
- **Dependency array:** Identical across all 3 (only the `useCallback` fetch function is passed).
- **Cleanup function:** Identical across all 3 (`supabase.removeChannel(channel)`).
- **Placement:** All 3 are set up at the top-level of their respective page components.
- **Conditional Logic:** None of them have any conditional logic wrapping the subscription.

## 2. Check for Visibility/Focus-Related Code

- **Result:** There is **NO** `document.visibilitychange`, `window.onfocus`, or any browser tab-visibility handling code in `app/dashboard/cook/page.tsx`. 
- **Comparison:** The Manager and Waiter dashboards also do not have any such visibility handling. 
- **Other `useEffect`s:** There are no other `useEffect` hooks in `cook/page.tsx` that could be tearing down or re-running the subscription unexpectedly. The realtime subscription is the only `useEffect` in the component.

## 3. Check Supabase Client-Level Reconnection Behavior

- **Shared vs Unique Instance:** All pages use a **shared Supabase client instance**. It is defined in `lib/supabase.ts` using `createBrowserClient(supabaseUrl, supabaseAnonKey)` and imported across the app.
- **Client Version:** The `package.json` specifies `@supabase/supabase-js` version `^2.110.8` (and `@supabase/ssr` version `^0.12.3`).

## 4. Console Errors (Action Required from Ayush)

Given that the code is identical across all three dashboards and they share the exact same client instance, the issue might be related to how the browser handles background tabs (throttling/sleeping) or how the Supabase JS client handles WebSocket reconnections under the hood.

**Manual Check Needed:** Ayush, please check the browser developer console on the Cook Dashboard when reproducing the tab switch. Look for any WebSocket disconnects (`ws://...`) or Supabase realtime errors. It's possible the connection drops and fails to reconnect properly due to background tab throttling, which might just coincidentally have appeared only on the Cook tab during testing.
