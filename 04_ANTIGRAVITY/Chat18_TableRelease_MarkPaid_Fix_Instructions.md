# Chat 18 — Table Release / Mark Paid Fix Instructions

**Project:** TableFlow  
**Date:** 2026-09-22  
**Purpose:** Implementation instructions based on the completed investigation report.

---

## 1. Source of Truth

Use the following investigation report as the basis for this fix:

`04_ANTIGRAVITY/Chat18_TableRelease_MarkPaid_Investigation_Report.md`

The investigation identified the specific issue:

- `app/dashboard/manager/page.tsx` → `markPaid(orderId: string)` currently performs a direct update on `orders`.
- The existing database RPC `public.mark_order_paid(p_order_id uuid, p_payment_method text)` contains the table-release logic.
- The cancellation flow already uses its RPC correctly and is not the target of this change.

Do not redesign the table system or change unrelated order flows.

## 2. Required Fix

In:

`app/dashboard/manager/page.tsx`

Change the Manager **Mark Paid** action so it calls the existing RPC:

```typescript
await supabase.rpc('mark_order_paid', {
  p_order_id: orderId,
  p_payment_method: paymentMethods[orderId] || 'cash'
})
```

Remove the current direct `orders` update used by `markPaid()`.

The existing UI payment-method selection must continue to be used.

## 3. Preserve Existing Behavior

Do not modify:

- Cook claiming/completion logic from Node 13.
- Waiter claiming/completion logic from Node 13.
- Cancellation RPC behavior.
- Order creation flow.
- Notification system.
- Table reservation logic.
- Existing Manager queues except the implementation of **Mark Paid**.
- Existing database migrations.

Do **not** create a new table-release mechanism. Reuse the existing `mark_order_paid` RPC.

## 4. Database Rules

No Supabase SQL migration is required for this fix unless source inspection proves an unexpected dependency.

Do not execute SQL against the production database as part of coding.

Do not change the existing `mark_order_paid` RPC without first reporting a concrete source-level reason.

## 5. Required Source Inspection Before Editing

Before making the change, inspect:

1. `app/dashboard/manager/page.tsx`
2. The existing `mark_order_paid` RPC definition in the Supabase migrations.
3. The current payment-method state used by the Manager UI.

Confirm that the RPC arguments match the current source before editing.

## 6. Verification Required

After the change:

### Code verification
Run:

```bash
npm run build
npx tsc --noEmit
```

Both must pass.

### Static/source verification
Confirm:

- `markPaid()` no longer performs a direct `orders.update({ status: 'billed', ... })` for completion.
- `markPaid()` calls `mark_order_paid`.
- The selected payment method is passed to the RPC.
- No unrelated files were changed.

### Runtime verification to be performed by the user
Do not claim this as completed by Antigravity.

The user will manually verify on Vercel:

1. Create/process an order through the normal flow.
2. Reach **Billing Queue (Served)** in the Manager dashboard.
3. Click **Mark Paid**.
4. Confirm the order leaves the Billing Queue / is completed.
5. Open **Tables**.
6. Confirm the table occupancy decreases correctly and the table becomes **Available** when all seats are released.
7. Confirm the existing receipt/payment behavior still works.

Also verify a partial-occupancy case where applicable: paying one order must reduce only that order's occupied seats.

## 7. Regression Expectations

The fix must not break:

- Manager order intake.
- Cook claim → preparing → ready flow.
- Waiter claim → ready → served flow.
- Node 13 ownership enforcement.
- Cancellation/table release.
- Manager notifications.

## 8. Scope Control

This is a **minimal bug fix**.

Do not:

- refactor unrelated Manager code,
- modify database schema,
- add new RPCs,
- alter Node 13 behavior,
- alter cancellation permissions,
- redesign table occupancy,
- change UI styling unless required for the fix.

## 9. Implementation Report

After coding, report:

- Files changed.
- Exact change made.
- RPC name and arguments used.
- Build result.
- TypeScript result.
- Any remaining concerns.
- Explicitly state that production/Vercel manual testing is still pending.

Do not state that the issue is fully fixed until the user completes the manual Vercel/table verification.
