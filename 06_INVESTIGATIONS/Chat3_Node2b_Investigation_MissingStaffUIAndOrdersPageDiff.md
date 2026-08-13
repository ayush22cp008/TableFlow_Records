# Investigation: Missing Staff UI and Orders Page Diff

## Issue 1: Signup page shows only Customer/Owner, no staff role option
**Investigation Findings:**
- **Code Status:** The staff roles and the new invite email flow are **fully implemented** in `components/AuthForm.tsx`. Lines 322-337 explicitly map over `['customer', 'owner', 'waiter', 'cook', 'manager']` to render the role selection cards. The new `/api/send-invite` and `/api/auth/staff-signup` logic is also wired up here.
- **Root Cause (Live URL):** Since `AuthForm.tsx` locally has all 5 roles and no CSS hiding them, if the live URL (`table-flow-nu.vercel.app/signup`) is showing only 2 roles, it means **the live environment is running an older commit** (possibly because the latest commit we pushed, `be97449`, hasn't finished deploying on Vercel yet, or a previous deployment failed).
- **Secondary Discovery:** The separate `app/auth/select-role/page.tsx` file (used for Google OAuth fallbacks) **still exists and has regressed/not been updated**. It still uses the old logic (asking for a code directly without the email step). This needs fixing separately.

## Issue 2: No "Staff Management" panel on Owner Dashboard
**Investigation Findings:**
- **Code Status:** A fully functional Staff Management page **was built** and exists at `app/dashboard/staff/page.tsx`.
- **Root Cause:** It is not showing on the dashboard because it was **never wired up** to the Owner Dashboard navigation grid. In `app/dashboard/page.tsx` (Lines 24-31), the array of quick-action cards hardcodes 6 routes (`/orders`, `/menu`, `/tables`, `/billing/demo`, `/analytics`, `/insights`) but completely omits `/dashboard/staff`.

## Issue 3: Unexpected change to app/dashboard/orders/page.tsx
**Investigation Findings:**
- **Code Status:** Commit `be97449` included changes to `app/dashboard/orders/page.tsx` that were unstaged local modifications present on the machine before our email delivery redesign.
- **Exact Diff Summary:** 
  - Added a `bulkError` state variable.
  - Updated `submitBulkCancel` to check for RPC execution errors from `cancel_active_orders` and set the `bulkError` state.
  - Added a UI panel to display `bulkError` if it occurs during the Bulk Emergency Stop.
- **Intentionality & Impact:** This was an **accidental/unrelated inclusion** caused by running `git add .` which scooped up an uncommitted fix from a previous session (likely related to fixing a bulk cancellation bug). It **does affect** the cancellation system by improving the error handling of the Bulk Emergency Stop feature, but it doesn't break it.

```diff
--- a/app/dashboard/orders/page.tsx
+++ b/app/dashboard/orders/page.tsx
@@ -36,6 +36,7 @@ export default function OrdersBoardPage() {
   const [bulkCategory, setBulkCategory] = useState('fire')
   const [bulkReason, setBulkReason] = useState('')
   const [selectedOrderIds, setSelectedOrderIds] = useState<string[]>([])
+  const [bulkError, setBulkError] = useState<string | null>(null)
 
   const fetchOrders = useCallback(async () => {
@@ -107,16 +108,25 @@ export default function OrdersBoardPage() {
   // Submit Bulk Cancel
   async function submitBulkCancel(e: React.FormEvent) {
     e.preventDefault()
-    if (bulkMode === 'specific' && selectedOrderIds.length === 0) return
+    setBulkError(null)
+    if (bulkMode === 'specific' && selectedOrderIds.length === 0) {
+      setBulkError('Please select at least one order.')
+      return
+    }
 
     const payloadIds = bulkMode === 'all' ? null : selectedOrderIds
 
-    await supabase.rpc('cancel_active_orders', {
+    const { error } = await supabase.rpc('cancel_active_orders', {
       p_reason: bulkReason.trim() || null,
       p_category: bulkCategory,
       p_order_ids: payloadIds
     })
 
+    if (error) {
+      setBulkError(error.message || 'Failed to cancel orders. Check permissions or data.')
+      return
+    }
```
