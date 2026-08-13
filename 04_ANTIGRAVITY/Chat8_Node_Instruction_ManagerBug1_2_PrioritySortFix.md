# TableFlow — Chat #8 — Instruction (Fix Only)

**To:** Antigravity
**From:** Claude
**Type:** Fix — Bug 1 & 2 (Manager priority sort)

---

## Context

Per investigation (`Chat8_Node_Investigation_ManagerBug1_2_PrioritySort.md`): both Intake Queue and Billing Queue in `app/dashboard/manager/page.tsx` are missing the `is_priority` sort clause — they only sort by `created_at`. Root cause confirmed identical for both bugs.

## Fix

In `app/dashboard/manager/page.tsx`:

**1. Intake Queue query (lines ~22-27)** — add `is_priority` sort before `created_at`:
```typescript
const { data: placedData } = await supabase
  .from('orders')
  .select('*, order_items(quantity, unit_price, menu_items(name)), restaurant_tables(table_number)')
  .eq('status', 'placed')
  .order('is_priority', { ascending: false })
  .order('created_at', { ascending: true })
```

**2. Billing Queue query (lines ~28-33)** — same change:
```typescript
const { data: servedData } = await supabase
  .from('orders')
  .select('*, order_items(quantity, unit_price, menu_items(name)), restaurant_tables(table_number)')
  .eq('status', 'served')
  .order('is_priority', { ascending: false })
  .order('created_at', { ascending: true })
```

## Scope

Only touch these two `.order()` clauses in `app/dashboard/manager/page.tsx`. Do not touch anything else — Bug 3 (Mark Paid table release) is a separate, unrelated fix and out of scope for this prompt.

## Evidence required

- Build must pass
- Screenshot or Ayush's manual confirmation showing Reservation (`R`) orders now appear above Walk-in (`W`) orders in both Intake Queue and Billing Queue
- Report to: `03_Investigation_and_Errors/Chat8_Node_Evidence_ManagerBug1_2_Fixed.md`

## Next

Do NOT commit/push yet — wait for Ayush's explicit go-ahead after manual verification, per standing GitHub push protocol.
