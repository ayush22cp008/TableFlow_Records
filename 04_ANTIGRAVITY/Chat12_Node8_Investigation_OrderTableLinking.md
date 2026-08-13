# Investigation: Table Number + Order Number on My Orders (Customer-Side)

**Node:** 8 — Customer Dashboard Revamp
**Type:** Investigation ONLY — no fix, no code changes.

## Context
Customer's "My Orders" page (`/order/my-orders`) currently shows only order ID (short hash), timestamp, status, total. Need to add **table number** and a **short order label** (matching the R1/W1-style convention used on staff dashboards, e.g. an order number like O1/W1) for consistency with owner/waiter/manager/cook dashboards.

## What to investigate
1. Is a table number already captured/stored when a customer places an order? Check the `orders` table schema and the order-placement flow (cart → place order route) — is `table_id` / `table_number` present on the order record?
2. If yes — confirm exactly which field, and whether it's reliably populated for all orders (or only some flows, e.g. only if reservation exists first).
3. If no — how does the app currently know which table a customer is ordering from at all? (QR code per table? Manual entry? Session-based?)
4. Find how `formatOrderNumber()` (used elsewhere per Chat 10/11 for W#/R# labels) works — can the same numbering be reused/extended for customer-facing order labels, and does it already produce a value accessible for a given order that a customer could see?
5. Confirm the customer's own `my-orders` query — does it fetch full order rows (which would already include table_id if present) or a trimmed subset missing that field?

## Output
Write findings to `03_Investigation_and_Errors/Chat12_Node8_Investigation_OrderTableLinking.md` — whether table_id already exists on orders (with evidence), how table is determined per order today, whether formatOrderNumber() is reusable as-is, and what (if anything) is missing to display table number + order label on my-orders. No fix, no code edits.
