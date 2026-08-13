# TableFlow — Chat #5 / Node 3 — Instruction (Fix): Manager Two Remaining Bugs

**To:** Antigravity
**From:** Claude
**Type:** Fix — approved, proceed with both fixes from your investigation

---

## Approved

Both root causes from `Chat5_Node3_Investigation_ManagerTwoRemainingBugs_Result.md` are confirmed and approved. Fix both in this pass — they are independent (different files, different bug classes), no shared cause, safe to bundle.

## 1. Fix Bug A — `components/AuthForm.tsx` missing `manager` branch

The redirect logic is hardcoded in three places: `handleStaffSignup`, `handleVerifyOtp`, `handleLogin`. Example current line:

```typescript
window.location.href = role === 'owner' ? '/dashboard' : role === 'cook' ? '/dashboard/cook' : '/order'
```

Update all three locations to explicitly check `role === 'manager'` and redirect to `/dashboard/manager`.

**Scope — Manager only:** do NOT add a `waiter` branch in this pass, even though the same gap exists for `waiter` in these three locations. That will be handled separately after Manager Dashboard is fully tested and locked. Leave the `waiter` fallthrough (`/order`) untouched for now.

## 2. Fix Bug B — infinite loop in `app/dashboard/manager/page.tsx`

Current pattern:

```typescript
const fetchOrders = useCallback(async () => {
  // ...
  const initialMethods = { ...paymentMethods }
  // ...
  setPaymentMethods(initialMethods)
}, [paymentMethods])

useEffect(() => {
  fetchOrders()
}, [fetchOrders])
```

Fix:
- Remove `paymentMethods` from the `useCallback` dependency array.
- Switch `setPaymentMethods` to the functional-update form (`setPaymentMethods(prev => ({ ...prev, ...newData }))` or equivalent), so it no longer needs `paymentMethods` as a dependency and doesn't force a new reference when nothing changed.

## Verification Before Reporting Back

- `npm run build` — 0 errors
- Confirm Bug A fix: manual signup with a `manager` invite code lands on `/dashboard/manager` (not `/order`)
- Confirm Bug B fix: Manager Dashboard loads and stops spinning — no repeated/looping network requests in the browser console or network tab
- Confirm `waiter` branch was NOT touched in `AuthForm.tsx` (still falls through to `/order` — expected, out of scope)

## Output

Do NOT push to GitHub yet — push is manually triggered by Ayush after review. Report back with files touched + build result. Ayush will test live after Vercel deploy — both bugs, controlled steps (manual signup for Bug A, Google Sign-In then dashboard load for Bug B) — per evidence rule.
