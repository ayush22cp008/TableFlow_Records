# Instruction for Antigravity — Chat 16 / Node 9 — Follow-Up Investigation Only

**Type: INVESTIGATION ONLY. No code changes, no fixes, no schema/table creation. This follows up on gaps found in `Chat16_Node9_Investigation_NotificationInsertionPoints.md` — do not repeat what that report already confirmed.**

## Gap 1 — Bulk Cancel status-snapshot logic

In `app/dashboard/orders/page.tsx`, inside/around `submitBulkCancel`, report the EXACT code that runs **before** the `supabase.rpc('cancel_active_orders', {...})` call:

- Does the code fetch/read each affected order's current `status` (`placed` / `preparing` / `ready`) before the RPC executes? Show the exact query or in-memory data used.
- Is this status available per-order at the point where cancellation is confirmed (for both "Cancel ALL" and "Select Specific" modes)?
- If NO such pre-RPC status snapshot currently exists in code, say so explicitly — don't infer one. This is critical: the locked design (Placed→notify Manager, Preparing→notify Cook, Ready→notify Waiter, Customer always) depends on this snapshot existing or being addable at this exact point.

## Gap 2 — Full realtime channel inventory

The first investigation found only 4 channels (`orders_board`, `manager_orders_realtime`, `cook_orders_realtime`, `reservation_status_realtime`) but Node 11's locked master-prompt record lists 7+ channels including `waiter_orders_realtime`, `owner_analytics_realtime`, `staff_management_realtime`, `owner_menu_realtime`, `menu_realtime`, `tables_realtime`.

Specifically check these files and report the exact channel name used in each (grep for `.channel(`):
- `app/dashboard/waiter/page.tsx`
- `app/dashboard/analytics/page.tsx`
- `app/dashboard/staff/page.tsx`
- `app/dashboard/menu/page.tsx`
- `app/order/cart/page.tsx` (if any)
- Any other file with a `.channel(` call not yet listed

Produce a complete, corrected list of every realtime channel currently in the codebase, with file path next to each. Flag any discrepancy against the previously documented list.

## Output format
Two sections (Gap 1, Gap 2), each tagged VERIFIED. If something isn't found, state that explicitly rather than guessing.

## Rules
- No fixes, no refactors, no new files.
- No side quests — unrelated findings go under a separate "Unrelated observations" section, not acted on.
