# TableFlow — Chat #8 — Instruction (Fix Only)

**To:** Antigravity
**From:** Claude
**Type:** Fix — Manager Dashboard W#/R# label retrofit

---

## Context

Per investigation (`Chat8_Node_Investigation_ManagerWR_LabelRetrofit.md`): Manager Dashboard hardcodes raw order UUIDs (`Order #{id.slice(0,6)}`) instead of the `W#`/`R#` format used elsewhere. The underlying data (`daily_number`, `is_priority`) is already fetched by Manager's queries — only the render logic is wrong.

**Scope decision (Ayush):** Fix Manager Dashboard ONLY. Do NOT touch Cook, Waiter, or Orders pages — their inline logic stays as-is for now.

## Fix

**1. Create a shared utility function** in `lib/utils.ts` (or the project's existing shared utils location):
```typescript
export function formatOrderNumber(order: { id: string; daily_number?: number | null; is_priority?: boolean }): string {
  return order.daily_number
    ? `${order.is_priority ? 'R' : 'W'}${order.daily_number}`
    : `#${order.id.slice(0, 6)}`;
}
```

**2. Update `app/dashboard/manager/page.tsx`:**
- Import `formatOrderNumber` from the shared utils location
- **Intake Queue (line ~150):** replace `Order #{order.id.slice(0, 6)}` with `Order #{formatOrderNumber(order)}` — adjust exact call to match how the component renders (may not need the literal "Order #" prefix if `formatOrderNumber` already returns `R3`/`W12` style; match the visual style already used in Cook/Waiter dashboards, e.g. just `R3` not `Order #R3`)
- **Billing Queue (line ~198):** same replacement

**Match existing visual convention:** Owner/Cook/Waiter show plain `R3`, `W12` (no "Order #" prefix, no id fallback shown to user). Manager Dashboard's labels should look the same for consistency — confirm current Manager UI text wrapping (e.g. "Order #R3" vs just "R3") and adjust to match the established pattern from Cook/Waiter/Owner views.

## Scope

Only touch:
- New shared util function (`formatOrderNumber` in `lib/utils.ts` or equivalent)
- `app/dashboard/manager/page.tsx` (Intake Queue + Billing Queue label rendering)

Do NOT touch `app/dashboard/cook/page.tsx`, `app/dashboard/waiter/page.tsx`, or `app/dashboard/orders/page.tsx` — their existing inline logic must remain untouched and working exactly as before.

## Evidence required

- Build must pass
- Screenshot or Ayush's manual confirmation showing Manager Dashboard's Intake Queue and Billing Queue now display `R#`/`W#` labels matching the style used in Cook/Waiter/Owner dashboards
- Report to: `03_Investigation_and_Errors/Chat8_Node_Evidence_ManagerWR_LabelFixed.md`

## Next

Do NOT commit/push yet — wait for Ayush's explicit go-ahead after manual verification, per standing GitHub push protocol.
