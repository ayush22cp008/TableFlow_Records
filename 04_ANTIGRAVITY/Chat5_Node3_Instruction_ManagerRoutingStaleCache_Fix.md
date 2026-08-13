# TableFlow — Chat #5 / Node 3 — Instruction (Fix): Manager Routing — Stale Cookie + Fetch Cache

**To:** Antigravity
**From:** Claude
**Type:** Fix — approved, proceed with both fixes from your investigation

---

## Approved

Both fixes from `Chat5_Node3_Investigation_ManagerRoutingStillBroken_Result.md` are approved. This resolves two separate root causes behind the 3 inconsistent outcomes previously observed (Owner Dashboard / Customer `/order` page / occasional correct Manager Dashboard).

## 1. Fix `app/auth/callback/route.ts` — stale cookie on Google Sign-In path

Replace the redundant `getUser()` call (which reads the stale incoming request cookie) with the user object already returned by `exchangeCodeForSession()`:

```typescript
const { data: { user }, error } = await supabase.auth.exchangeCodeForSession(code)
// Remove the separate getUser() call — use this `user` directly
```

Update all downstream logic in this file that referenced the old `getUser()` result to use this `user` object instead.

## 2. Fix Next.js fetch caching — `middleware.ts` and `lib/supabase-server.ts`

Force the Supabase server client to bypass Next.js's Edge fetch cache so `profiles.role` is always read fresh:

```typescript
global: {
  fetch: (url, options) => fetch(url, { ...options, cache: 'no-store' })
}
```

Apply this to the `createServerClient` config in **both** `middleware.ts` and `lib/supabase-server.ts` (wherever a server-side Supabase client is constructed) — check if there are other locations constructing a server client with the same pattern and apply consistently, so no stale-cache path is missed.

## Scope check

Search the codebase for any other place that constructs a Supabase server/edge client and confirm the same `cache: 'no-store'` fix is applied there too, not just the two files named above — this bug class (stale Edge fetch cache) could affect any role-gated route, not just Manager.

## Verification Before Reporting Back

- `npm run build` — 0 errors
- Confirm the fix conceptually resolves all 3 observed outcomes (explain briefly how each is addressed)
- Flag if any other server-client construction site was found and fixed beyond the two named files

## Output

Do NOT push to GitHub yet — push is manually triggered by Ayush after review. Report back with files touched + build result. Ayush will test live after Vercel deploy — this time with **controlled, repeatable steps** (same login method, same browser, noted explicitly) so results are conclusive, per evidence rule.
