# Chat 18 — Node 13 Migration Recovery Investigation

**Date:** 2026-09-20  
**Project:** TableFlow  
**Node:** Node 13 — Cook/Waiter Order Claiming System  
**Investigation Type:** Migration repository-state verification  
**Status:** OPEN — migration SQL must be restored/verified before Supabase execution

---

## 1. Purpose

This investigation records the migration-state issue discovered after the Node 13 implementation was committed and pushed to the TableFlow source repository.

The Node 13 application/source changes were pushed to `main`, but verification found that:

`supabase/migrations/20260920000001_node13_order_claiming.sql`

exists in the GitHub repository while its fetched content is empty.

Therefore the migration must be restored and verified before it is executed in Supabase.

---

## 2. Source Repository State

Repository:

`https://github.com/ayush22cp008/TableFlow`

Node 13 implementation commit:

`12c2c6e58b56c8a1cf0bcf3d95f35b6c6de83b2b`

Commit message:

`Implement Node 13: Cook/Waiter Order Claiming System`

The commit contains the Node 13 source implementation.

---

## 3. Migration Verification Finding

Expected migration:

`supabase/migrations/20260920000001_node13_order_claiming.sql`

GitHub confirms the file exists, but the fetched file content is empty apart from the UTF-8 BOM.

This means the repository currently does **not** contain usable Node 13 migration SQL in that file.

**Important:** The file must not be executed in its current state.

---

## 4. Required Migration Contents

The restored migration must be reconciled against the approved documents:

- `03_AI_BRAINS/ChatGPT/Chat18_Node13_Implementation_Spec_v1.0.md`
- `04_ANTIGRAVITY/Chat18_Node13_Implementation_revised_plan.md`
- `03_AI_BRAINS/Claude/REVIEWED_Chat18_Node13_Implementation_revised_plan.md`

It must contain all approved Node 13 database changes, including:

- durable `profiles.staff_name`
- `orders.claimed_by_cook_id`
- `orders.claimed_by_waiter_id`
- partial unique indexes
- Manager profile read policy
- removal of direct Cook/Waiter completion bypass policies
- complete claim cleanup invariant trigger
- four Node 13 claim/completion RPCs
- SECURITY DEFINER hardening
- authentication, role and active-status validation
- row locking and one-active-order enforcement
- exactly-one-active-manager validation
- Manager `order_claimed` notification
- notification type constraint update
- required function grants/revokes

---

## 5. Required Trigger Invariant

The final trigger must enforce the approved claim lifecycle:

- `ready` → clear Cook claim
- `placed` / `preparing` → clear Waiter claim
- `served` → clear both claims
- `billed` → clear both claims
- `cancelled` → clear both claims

The trigger must be reviewed in the complete SQL before migration execution.

---

## 6. Current Safety Boundary

At the time of this investigation:

- Node 13 source implementation: pushed
- Migration file: present but empty
- Supabase migration: **NOT EXECUTED**
- Manual Vercel testing: pending
- Final regression: pending
- Node 13 lock: pending

No production/database state should be assumed from the existence of an empty migration file.

---

## 7. Required Next Action

Antigravity must:

1. Restore the complete approved SQL into the existing migration file.
2. Inspect the full migration for completeness and syntax issues.
3. Run:
   `npm run build`
   `npx tsc --noEmit`
4. Commit the migration correction with:
   `fix: restore Node 13 migration SQL`
5. Push the correction to TableFlow `main`.
6. Report the new commit SHA.
7. Do **not** execute the migration in Supabase.
8. Do **not** mark Node 13 locked.

After the correction is pushed, the migration file must be independently re-verified before manual Supabase execution.

---

## 8. Conclusion

The Node 13 source implementation has been pushed to GitHub, but the required database migration was empty at the time of verification.

The correct next state is:

**restore migration SQL → verify repository migration → run manager-count pre-check → execute migration manually in Supabase → manual Vercel testing → regression testing → Node 13 lock**

This investigation remains open until the migration SQL is present and verified.
