# Investigation Result: Two Remaining Manager Bugs (Chat 5 / Node 3)

## Bug A — Manual signup routes to Customer page
**Root Cause:** The manual Email/Password signup and login flows are handled by `components/AuthForm.tsx`. In this file, the client-side redirect logic is hardcoded in three different places (`handleStaffSignup`, `handleVerifyOtp`, and `handleLogin`). 

For example, in `handleStaffSignup`:
```typescript
window.location.href = role === 'owner' ? '/dashboard' : role === 'cook' ? '/dashboard/cook' : '/order'
```
Because this file was not touched during the earlier fixes (which focused on the magic link flow and the `/auth/select-role` page), it is missing the `manager` branch entirely. Thus, when a manager signs up manually, they are successfully assigned the role in the DB, but the local logic redirects them to the fallback path (`/order`).

**Proposed Fix:** Update the three hardcoded redirects in `components/AuthForm.tsx` to explicitly check for `role === 'manager'` and redirect to `/dashboard/manager`. (Note: The `waiter` role is also missing here and currently falls through to `/order`).

---

## Bug B — Google Sign-In reaches `/dashboard/manager` but gets infinite spinner
**Root Cause:** This is not a hanging Supabase query or RLS error, but a classic **React Infinite Loop**.

In `app/dashboard/manager/page.tsx`, the `fetchOrders` function is wrapped in `useCallback` and includes `paymentMethods` in its dependency array. However, inside `fetchOrders`, it calls `setPaymentMethods()` with a newly cloned object (`{ ...paymentMethods }`) after fetching data. 
```typescript
  const fetchOrders = useCallback(async () => {
    // ...
    const initialMethods = { ...paymentMethods }
    // ...
    setPaymentMethods(initialMethods) // State updated with new object reference!
  }, [paymentMethods])

  useEffect(() => {
    fetchOrders()
  }, [fetchOrders])
```
**Execution flow causing the hang:**
1. `useEffect` calls `fetchOrders()`.
2. `fetchOrders` sets `loading` to `true`, finishes fetching, and sets `paymentMethods` to a new object.
3. The component re-renders.
4. `useCallback` sees `paymentMethods` changed, so it creates a new reference for `fetchOrders`.
5. `useEffect` sees `fetchOrders` changed, so it triggers again.
6. The loop repeats instantly, causing thousands of network requests in the background while the UI stays trapped with `loading = true`.

**Proposed Fix:** Remove `paymentMethods` from the `useCallback` dependency array and use the functional update pattern for `setPaymentMethods` (e.g., `setPaymentMethods(prev => { ... })`). By returning the same `prev` reference when no new orders are found, we break the infinite render loop.
