# TableFlow — Chat #5 / Node 3 — Instruction (Fix): Manager Routing Bug

**To:** Antigravity
**From:** Claude
**Type:** Fix — approved, proceed with both parts of your proposed fix

---

## Approved

Both parts of the fix in `Chat5_Node3_Investigation_ManagerRoutingBug_Result.md` are approved. Proceed with both.

## 1. Direct fix

**File:** `app/auth/select-role/page.tsx`, line 76

Change:
```typescript
window.location.href = role === 'manager' ? '/dashboard' : role === 'cook' ? '/dashboard/cook' : '/order'
```
to:
```typescript
window.location.href = role === 'manager' ? '/dashboard/manager' : role === 'cook' ? '/dashboard/cook' : '/order'
```

Check this same line/ternary for a `waiter` branch too — if `waiter` isn't handled here either and falls through to `/order`, flag it back to me (don't add it yourself without confirming the correct waiter dashboard path first, since Node 3 Waiter Dashboard is still ⬜ NOT STARTED).

## 2. Safeguard

**File:** `middleware.ts`

Add logic: if an authenticated user with role `manager`, `cook`, or `waiter` requests the exact `/dashboard` path, redirect them to their own dashboard route (`/dashboard/manager`, `/dashboard/cook`, `/dashboard/waiter` respectively) instead of letting them land on the Owner view. Owner role continues to reach `/dashboard` normally.

## Verification Before Reporting Back

- `npm run build` — 0 errors
- Confirm manually (or via test) that a `manager`-role session hitting `/dashboard` directly gets redirected, not shown Owner content
- Confirm `owner` role is unaffected — still lands on `/dashboard` normally

## Output

Do NOT push to GitHub yet — push is manually triggered by Ayush after review. Report back with files touched + build result + whether a `waiter` gap was found in `select-role/page.tsx`. Ayush will test live after Vercel deploy, per evidence rule.
