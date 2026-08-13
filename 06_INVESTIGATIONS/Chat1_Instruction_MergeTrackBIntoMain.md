# Instruction — Merge Track B into main

Chat #1 | Node: Track B — merge to main and push

## Task

Merge branch `feature-track-b-priority-numbering` into `main` and push to origin.

## Context

Build already verified (`npm run build` — 0 errors). Local automated verification script was skipped (no test users/tables found in DB), but that's acceptable — manual verification will happen directly on the live site after deploy by placing a real reservation order and a real walk-in order and checking R1/W1 display + sort order.

## Steps

1. Checkout `main`, pull latest from origin.
2. Merge `feature-track-b-priority-numbering` into `main` (fast-forward if possible, no rebase).
3. Push `main` to origin.
4. Confirm the merge includes: schema migration file, RPC update, `types/index.ts` changes, `app/order/cart/page.tsx` changes, `app/dashboard/orders/page.tsx` changes.
5. Report: merge commit hash, push success/failure, files changed in the merge.

## Reminder

The SQL migration was already manually run in Supabase SQL Editor by Ayush — do not attempt to re-run or re-apply it via any CLI/script as part of this merge.

## Do Not

- Do not touch any other branch.
- Do not start any new feature work.
