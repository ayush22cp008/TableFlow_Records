# Waiter Google OAuth Routing — Fix Instruction

**Based on:** `Chat7_Node_Waiter_Investigation_GoogleOAuthRoutingBug_Result.md` (confirmed root cause, both files)

## Fix 1 — `app/auth/select-role/page.tsx` (line 76)

**Current:**
```typescript
window.location.href = role === 'manager' ? '/dashboard/manager' : role === 'cook' ? '/dashboard/cook' : '/order'
```

**New:**
```typescript
window.location.href = role === 'manager' ? '/dashboard/manager' : role === 'cook' ? '/dashboard/cook' : role === 'waiter' ? '/dashboard/waiter' : '/order'
```

## Fix 2 — `app/auth/callback/route.ts` (lines 33-40)

**Current:**
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

**New:** add a `waiter` branch before the final fallback, same pattern as `manager`:
```typescript
if (profile?.role === 'owner' || metadataRole === 'owner') {
  return NextResponse.redirect(`${origin}/dashboard`)
} else if (profile?.role === 'cook' || metadataRole === 'cook') {
  return NextResponse.redirect(`${origin}/dashboard/cook`)
} else if (profile?.role === 'manager' || metadataRole === 'manager') {
  return NextResponse.redirect(`${origin}/dashboard/manager`)
} else if (profile?.role === 'waiter' || metadataRole === 'waiter') {
  return NextResponse.redirect(`${origin}/dashboard/waiter`)
}
return NextResponse.redirect(`${origin}/order`)
```

## Verification

1. `npm run build` — must compile clean.
2. Ayush to manually test: Google Sign-In with a waiter invite code → should land on `/dashboard/waiter`, not `/order`.
3. Re-confirm manual signup still works (should be unaffected, but verify no regression).

## Scope note

Fix only — no other changes. Do not touch `AuthForm.tsx` (already correct per Chat 7 build). Do not touch `manager`/`cook`/`owner` branches (already working, leave as-is).
