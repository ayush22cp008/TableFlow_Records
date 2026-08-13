# Chat 7: Waiter Dashboard — Google OAuth Routing Bug Investigation Result

## 1. `app/auth/select-role/page.tsx`
**Status:** Confirmed Bug (Missing `waiter` branch)
**Details:** The ternary on line 76 is missing the `waiter` redirect and defaults to `/order`.

**Current code (Line 76):**
```typescript
window.location.href = role === 'manager' ? '/dashboard/manager' : role === 'cook' ? '/dashboard/cook' : '/order'
```

## 2. `app/auth/callback/route.ts`
**Status:** Confirmed Bug (Missing `waiter` branch)
**Details:** The `if/else if` block starting at line 33 is also missing the `waiter` role check, which causes the fallback to `/order` (line 40).

**Current code (Lines 33-40):**
```typescript
        if (profile?.role === 'owner' || metadataRole === 'owner') {
          return NextResponse.redirect(`${origin}/dashboard`)
        } else if (profile?.role === 'cook' || metadataRole === 'cook') {
          return NextResponse.redirect(`${origin}/dashboard/cook`)
        } else if (profile?.role === 'manager' || metadataRole === 'manager') {
          return NextResponse.redirect(`${origin}/dashboard/manager`)
        }
        return NextResponse.redirect(`${origin}/order`)
```

## Conclusion
The bug exists in exactly the same shape as the Manager's Chat 5 bug. Both the `select-role` client-side redirect and the `callback` server-side redirect were left untouched when the `manager` fix was applied earlier. Both need the `waiter` branch added to correctly route to `/dashboard/waiter`.

**Status:** Awaiting fix instruction from Ayush/Claude. No code changes have been applied.
