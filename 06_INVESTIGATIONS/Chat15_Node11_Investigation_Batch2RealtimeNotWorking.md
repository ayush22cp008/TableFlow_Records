# Investigation Results — Node 11 Batch 2 Bug: Realtime Not Working

## 1. Exact Current Subscription Code

**app/order/reservation/page.tsx**
```tsx
    if (!user) return
    const channel = supabase.channel('reservation_status_realtime')
      .on('postgres_changes', { event: '*', schema: 'public', table: 'reservation_requests', filter: `customer_id=eq.${user.id}` }, checkReservation)
      .subscribe()
    return () => { supabase.removeChannel(channel) }
```

**app/dashboard/staff/page.tsx**
```tsx
    const channel = supabase.channel('staff_management_realtime')
      .on('postgres_changes', { event: '*', schema: 'public', table: 'profiles' }, fetchData)
      .subscribe()
    return () => { supabase.removeChannel(channel) }
```

**app/dashboard/menu/page.tsx**
```tsx
    const channel = supabase.channel('owner_menu_realtime')
      .on('postgres_changes', { event: '*', schema: 'public', table: 'menu_items' }, fetchItems)
      .subscribe()
    return () => { supabase.removeChannel(channel) }
```

## 2. Compare Against Known-Working Batch 1 Pattern

**app/dashboard/manager/page.tsx (Working Reference)**
```tsx
    const channel = supabase.channel('manager_orders_realtime')
      .on('postgres_changes', { event: '*', schema: 'public', table: 'orders' }, fetchOrders)
      .subscribe()
    return () => { supabase.removeChannel(channel) }
```

**Comparison:**
- The frontend code is **100% correct and structurally identical** to the working Batch 1 pattern.
- `.subscribe()` is called correctly on all.
- Table names and event parameters are correct.
- Cleanup functions are perfectly matched.

## 3. Supabase Realtime Replication Enabled?

- A search of `supabase/migrations/` reveals **no** `ALTER PUBLICATION supabase_realtime ADD TABLE` statements for any table (not even `orders`). This indicates that Realtime replication is being managed manually via the Supabase Dashboard UI.
- **The `menu_items` mystery:** If Realtime is not toggled ON in the dashboard for `menu_items`, `reservation_requests`, and `profiles`, the frontend code will silently do nothing. If the Pre-Node 9 audit claimed `menu_realtime` in `app/order/page.tsx` was working, it might have been a false positive (e.g. data refreshed via a component remount). If it *was* truly working, then Realtime is enabled for `menu_items` but RLS is blocking the Owner (see section 5).

## 4. Reservation Page Filter Syntax

- The filter syntax used is: `filter: \`customer_id=eq.${user.id}\``. This perfectly matches the official Supabase Realtime filter specification (e.g. `column=eq.value`).
- The subscription is wrapped with `if (!user) return`, so `user.id` is guaranteed to be fully populated and not `undefined` when the subscription initializes.

## 5. RLS Policies (The likely culprit)

Supabase Realtime strictly respects RLS `SELECT` policies. If the Realtime server evaluates the RLS policy and it fails (or throws an error), the event is silently dropped for that client.

- **`reservation_requests`:** 
  `CREATE POLICY "Allow public select" ON reservation_requests FOR SELECT TO public USING (true);`
  *(This is fully open, so RLS is not blocking this one. If this fails, Realtime is simply toggled OFF in the Dashboard for this table).*

- **`menu_items`:**
  `CREATE POLICY "menu_read" ON menu_items FOR SELECT USING (is_available = true AND is_active = true OR has_role(ARRAY['waiter', 'cook', 'manager', 'owner']));`
  *Note: The `has_role()` function relies on `auth.uid()`. Supabase Realtime sometimes struggles to evaluate complex PL/pgSQL functions like `has_role` during background event broadcasting. If the Realtime server errors evaluating `has_role`, it defaults to `false`. This would explain why customers see active menu items (first half of the condition is simple boolean), but the Owner cannot see hidden/inactive items via Realtime!*

- **`profiles`:**
  The `profiles` table and its initial RLS policies were created before SQL migrations were tracked (likely in the Dashboard). You will need to check the `SELECT` policy for `profiles` manually in the Dashboard to see if it's too restrictive or uses complex functions.

**Conclusion/Action for Ayush:**
The frontend code is perfectly wired. You need to manually check the Supabase Dashboard:
1. Database -> Replication -> Verify `reservation_requests`, `profiles`, and `menu_items` are toggled ON.
2. If they are ON, the issue is RLS blocking the Realtime server. We will need to adjust the RLS policies to be more Realtime-friendly (e.g. bypassing complex `has_role` checks for Realtime, or restructuring the policy).
