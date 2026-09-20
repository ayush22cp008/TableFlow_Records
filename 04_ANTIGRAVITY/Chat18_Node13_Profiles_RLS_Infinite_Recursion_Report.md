# Chat 18 — Node 13 Profiles RLS Infinite Recursion Report

**Date:** 2026-09-20  
**Project:** TableFlow  
**Node:** Node 13 — Cook/Waiter Order Claiming System  

---

## 1. Root Cause Analysis

The investigation identified that Supabase was returning HTTP `500` and `SQLSTATE 42P17` (`infinite recursion detected in policy for relation "profiles"`) on `GET /rest/v1/profiles`. 

This is directly caused by the `profiles_manager_select` policy introduced in Node 13:

```sql
CREATE POLICY "profiles_manager_select" ON public.profiles
FOR SELECT USING (
  (SELECT role FROM public.profiles WHERE id = auth.uid()) = 'manager'
);
```

Because this policy applies to `SELECT` on `profiles`, and its condition itself executes a `SELECT` on `profiles`, PostgreSQL detects an infinite evaluation loop.

This failure cascades to the Next.js `middleware.ts`. When `middleware.ts` attempts to query the user's role on navigation, the query fails. The middleware defaults the `role` to `'customer'`, resulting in all authenticated staff (including Manager, Cook, and Waiter) being incorrectly routed to the `/order` customer interface.

## 2. Fix Path

To break the recursion, we must check the user's role without triggering RLS on the `profiles` table. TableFlow already implements a solution for this in previous migrations: the `public.has_role(allowed_roles text[])` function.

Because `has_role` is declared as `SECURITY DEFINER`, it executes with elevated privileges and **bypasses Row Level Security**. 

Therefore, modifying the policy to use `has_role` will securely authorize Managers without triggering an infinite recursion loop.

## 3. Required Migration (Next Step)

Create a new Supabase migration to replace the recursive policy:

```sql
DROP POLICY IF EXISTS "profiles_manager_select" ON public.profiles;

CREATE POLICY "profiles_manager_select" ON public.profiles
FOR SELECT USING (
  public.has_role(ARRAY['manager'])
);
```

This single change will resolve the routing regression and restore the staff dashboards.