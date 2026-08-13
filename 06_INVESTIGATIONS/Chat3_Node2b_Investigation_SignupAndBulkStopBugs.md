# Investigation Report: Signup & Bulk Stop Bugs

Chat #3 | Node 2b

## Bug 1: Signup page missing role selection (Waiter/Cook/Manager)
**Status:** Built but disconnected.
**File Paths:** `app/signup/page.tsx`, `components/AuthForm.tsx`, `app/auth/select-role/page.tsx`
**Root Cause:**
The 3-layer validation staff role panel was indeed fully implemented, but it was built in a standalone page (`app/auth/select-role/page.tsx`). The main signup flow goes through `app/signup/page.tsx` which renders `<AuthForm view="signup" />`. Inside `components/AuthForm.tsx` (lines 245-263), the role selection step strictly hardcodes the array to only `['customer', 'owner']`. The new staff signup page was never linked or integrated into this primary flow.

## Bug 2: "Cancel ALL Active Orders" button does nothing
**Status:** Implemented but silently swallowing errors.
**File Path:** `app/dashboard/orders/page.tsx`
**Root Cause:**
In `app/dashboard/orders/page.tsx` (lines 108-124), the `submitBulkCancel` handler makes the call to the RPC:
```typescript
await supabase.rpc('cancel_active_orders', { ... })
```
However, it does not destructure or check for an `{ error }` response, nor is it wrapped in a `try/catch` block. If the RPC fails for any reason (e.g. database error, missing function, permissions), the error is completely swallowed. The function blindly proceeds to close the modal (`setBulkModalOpen(false)`) and refetch the orders, making it appear as if the button "did nothing".

## Bug 3: "Select Specific Orders" has no selection UI
**Status:** Implemented but blocked by UI layering (z-index/overlay issue).
**File Path:** `app/dashboard/orders/page.tsx`
**Root Cause:**
The checkbox selection UI *was* actually implemented on the order cards (lines 180-187), conditioned on `isSelectable = bulkModalOpen && bulkMode === 'specific'`. 
However, when the bulk modal opens, it renders a full-screen backdrop overlay: 
```tsx
<div className="fixed inset-0 z-50 flex flex-col items-center justify-center bg-black/80 backdrop-blur-sm px-4 pt-10">
```
This `fixed inset-0 z-50` overlay completely blocks pointer events to everything underneath it. As a result, the user cannot physically click the checkboxes on the order cards behind the modal.
