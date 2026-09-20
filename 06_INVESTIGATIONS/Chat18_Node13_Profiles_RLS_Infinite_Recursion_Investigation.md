# Chat 18 — Node 13 Profiles RLS Infinite Recursion Investigation

**Date:** 2026-09-20  
**Project:** TableFlow  
**Node:** Node 13 — Cook/Waiter Order Claiming System  
**Investigation Type:** Production/Vercel role-routing regression caused by Supabase RLS policy  
**Status:** INVESTIGATION COMPLETE — FIX REQUIRED

---

## 1. Purpose

This investigation documents the role-routing regression observed after the Node 13 migration was executed in Supabase.

Observed behavior:

- Owner, Manager, Cook, and Waiter users can authenticate.
- After refresh/navigation they are redirected to the customer-facing `/order` page.
- The expected staff dashboard is not reached.
- The customer page appears without the normal staff dashboard Navbar.
- Vercel does not show a visible application error.
- Supabase logs repeatedly show PostgreSQL error `42P17` and HTTP `500` responses for `GET /rest/v1/profiles`.

The objective is to identify the concrete database/policy cause and establish the exact fix path before changing production code or applying another migration.

---

## 2. Evidence Observed

Supabase logs show repeated entries with:

- HTTP status: `500`
- PostgreSQL SQLSTATE: `42P17`
- Message:
  `infinite recursion detected in policy for relation "profiles"`
- Failing request:
  `GET /rest/v1/profiles`

The errors recur during authenticated page refresh/navigation.

---

## 3. Node 13 Policy Causing the Problem

The Node 13 migration created this policy:

```sql
CREATE POLICY "profiles_manager_select" ON public.profiles
FOR SELECT USING (
  (SELECT role FROM public.profiles WHERE id = auth.uid()) = 'manager'
);
```

The policy is attached to `public.profiles` while its USING expression performs another SELECT against `public.profiles`.

This creates recursive RLS evaluation:

```text
SELECT from profiles
  -> evaluate profiles_manager_select
     -> SELECT from profiles
        -> evaluate profiles_manager_select again
           -> SELECT from profiles
              -> ...
```

PostgreSQL detects the recursion and returns SQLSTATE `42P17`.

---

## 4. Why All Authenticated Roles Are Affected

The current TableFlow `middleware.ts` queries the logged-in user's profile on requests:

```ts
const { data: profile } = await supabase
  .from('profiles')
  .select('role, is_active')
  .eq('id', user.id)
  .single()
```

The middleware then derives the role using:

```ts
let role = profile?.role ?? 'customer'
```

Therefore when the `profiles` query fails because of the recursive RLS policy:

- `profile` is unavailable;
- `role` falls back to `customer`;
- staff role-specific routing is not selected;
- dashboard access can be redirected to `/order`.

This explains why the same visible routing regression occurs for Owner, Manager, Cook, and Waiter accounts.

This is a routing consequence of the failed profile lookup, not evidence that those accounts' stored database roles were converted to `customer`.

---

## 5. Existing Role Helper in TableFlow

The repository already contains an existing `has_role(allowed_roles text[])` SECURITY DEFINER helper.

Current implementation:

```sql
CREATE OR REPLACE FUNCTION has_role(allowed_roles text[])
RETURNS boolean
LANGUAGE sql
SECURITY DEFINER
STABLE
AS $$
  SELECT EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid()
      AND role = ANY(allowed_roles)
      AND is_active = true
  );
$$;
```

The project already uses this helper in other RLS policies.

The important implication for the investigation is:

> The new Manager profile-read policy should not directly query `profiles` from a `profiles` RLS policy.

The exact replacement should be reviewed against the approved Node 13 security design before execution.

---

## 6. Root Cause

**Confirmed root cause:**

The Node 13 migration introduced a self-referencing RLS policy on `public.profiles` that queries the same `profiles` relation from inside its own policy evaluation.

This causes PostgreSQL `42P17` infinite-recursion errors.

The middleware depends on reading `profiles.role`, so the failed query causes role resolution to fall back to the customer path.

---

## 7. Scope of Impact

### Directly affected

- Authenticated profile reads through the `profiles` table.
- Middleware role resolution.
- Staff dashboard routing after refresh/navigation.
- Manager profile joins/reads that depend on the affected policy.

### Not established by this investigation

This investigation does not establish that:

- the stored role values in `profiles` were modified;
- Node 13 order-claim RPCs are themselves failing;
- Node 9 notification triggers are broken;
- Vercel deployment itself is the root cause.

Those areas require separate verification after the RLS issue is corrected.

---

## 8. Required Fix Path

The fix must be handled as a separate implementation/fix step.

Before applying the fix:

1. Review the approved Node 13 design/spec and the existing RLS architecture.
2. Confirm the exact non-recursive Manager-read policy.
3. Preserve Manager access to claimant profile information required by Node 13.
4. Do not weaken unrelated profile RLS.
5. Do not redesign the Node 13 claim system.

The expected architectural direction is to use the existing role-helper approach rather than recursively selecting from `profiles` inside the `profiles` policy.

---

## 9. Safety Boundary

Until the fix is reviewed and applied:

- Do not mark Node 13 locked.
- Do not treat the current Vercel behavior as a successful Node 13 test.
- Do not begin final regression testing.
- Do not make unrelated UI changes to hide the routing symptom.
- Do not assume the role data itself is corrupted.

---

## 10. Investigation Conclusion

The repeated Supabase `42P17` and `500` errors are explained by the Node 13 `profiles_manager_select` policy recursively querying `public.profiles`.

That database policy failure breaks the middleware's profile lookup, causing authenticated staff users to fall back to the customer route.

**Status: Root cause identified and evidence-supported. Exact fix must now be implemented and verified separately.**

---

## 11. Next Controlled Step

Create a dedicated fix plan for the `profiles` RLS recursion issue, have it reviewed, then apply the minimum required database-policy correction.

After the fix:

```text
RLS fix
  -> verify profiles reads
  -> verify Owner routing
  -> verify Manager routing
  -> verify Cook routing
  -> verify Waiter routing
  -> verify Node 13 claim flow
  -> continue final regression
  -> lock Node 13 only after all required tests pass
```
