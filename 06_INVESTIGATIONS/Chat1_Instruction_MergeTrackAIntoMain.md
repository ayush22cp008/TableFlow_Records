# Instruction — Merge Track A fix into main

Chat #1 | Node: Bug Fix — merge stale-reservation-flow fix

## Task

Merge branch `fix-stale-reservation-flow` into `main` and push to origin.

## Steps

1. Checkout `main`, pull latest from origin.
2. Merge `fix-stale-reservation-flow` into `main` (no rebase, keep it simple — fast-forward if possible).
3. Push `main` to origin.
4. Confirm the merge commit includes the Confirm Arrival removal changes (`app/dashboard/tables/page.tsx`, `app/order/cart/page.tsx`).
5. Report: merge commit hash, push success/failure, files changed in the merge.

## Do Not

- Do not touch any other branch.
- Do not start Track B (priority/numbering) work.
- Do not modify code beyond what's already in `fix-stale-reservation-flow`.
