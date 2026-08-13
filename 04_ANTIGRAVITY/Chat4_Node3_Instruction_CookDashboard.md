# TableFlow — Chat #4 / Node 3 — Instruction (Build): Cook Dashboard

**To:** Antigravity
**From:** Claude
**Type:** Build — implement per locked spec

---

## Reference

Full spec: `01_Master_Prompts/Claude_Side/Chat4_Node3_MasterPrompt_Claude_CookDashboard.md`

## Build Tasks

### 1. Routing
- `app/auth/callback/route.ts` — add branch: `if (role === 'cook') redirect to '/dashboard/cook'`
- `middleware.ts` — add matching `cook` branch alongside existing `owner` logic

### 2. New page: `app/dashboard/cook/page.tsx`
- Query orders where `status = 'preparing'` using: `.select('*, order_items(quantity, menu_items(name))')`
- Render as cards: order short ID + item list (`{quantity}x {name}`) only — **no price, no table number, no customer name/info**
- Each card: single "Mark Ready" button → updates `orders.status = 'ready'` for that order (relies on existing `cook_prep_to_ready` RLS policy, already live)
- Manual refresh only — a refresh button or simple re-fetch on click. **No polling, no auto-refresh interval.**

### 3. Reuse existing pattern
Base the card/button structure on the existing Kanban card in `app/dashboard/orders/page.tsx` (query join style, `advanceStatus`-style handler) — don't reinvent, adapt the filtered/stripped version per the spec.

## Explicitly Do Not Build

- Auto-refresh/polling (Node 4, later)
- Manager or Waiter dashboards (separate future instructions)
- Any cancellation UI on this page

## Verification Before Reporting Back

- `npm run build` — 0 errors
- Confirm RLS: Cook role can update `preparing → ready` (already covered by existing `cook_prep_to_ready` policy — just confirm the frontend call succeeds against it)

## Output

Do NOT push to GitHub yet — per protocol, push is manually triggered by Ayush after he reviews. Report back with files touched + build result. Ayush will test live after Vercel deploy, per evidence rule.
