I'll pull the actual current source to verify the claims in the investigation report before reviewing.Important — Node 9's migrations exist (`20260814000001_node9_schema.sql`, `20260817000001_node9_triggers_realtime.sql`), dated *after* the investigation report's referenced files but the investigation/design docs say Node 9 is only "investigation complete." Let me pull the key files to verify current reality vs. the docs.This confirms the RLS facts in the investigation report. Now Node 9's actual migrations, which the docs claim aren't built yet.This is a critical finding — Node 9 is **fully implemented** via DB triggers already (dated 2026-08-17), contradicting both the investigation report and Chat 16 handoff which describe Node 9 as "investigation complete, not yet implemented." Let me check the schema file and the current Cook/Waiter/Manager page code before finalizing.Excellent — this is actually good news for Node 13 design compatibility (schema already supports `recipient_id` targeting, not just role). Now let's check Cook/Waiter/Manager dashboard pages for exact current code, and check for any claim-related columns already on `orders`.Confirms: no claim button, no ownership check, bare `.update()`. Now let me check `mark_order_paid` RPC (template pattern) and confirm `orders` table has no claim columns, plus check the staff management page for `staff_name` usage.**Important**: this RPC template has **no `search_path` set** and **no explicit role/auth check inside the function** — it relies entirely on RLS being satisfied before the RPC is called (SECURITY DEFINER bypasses RLS by default unless `SET (row_security = on)` or caller checks are added). This is a real gap the Node 13 design must not blindly copy. Let me check staff_management and cancellation RPC quickly, then also confirm `staff_name` usage.Good — confirms `profiles` has no `staff_name` column; `invite_codes.staff_name` is the only source, matching the design doc's concern. `force_logout_all_staff` at least does an explicit caller check, unlike `mark_order_paid`. Neither sets `search_path`. This confirms a real gap to flag. I have enough now.

---

# Node 13 Design Review

## 1. Overall Result: **FITS WITH REQUIRED CHANGES**

The core approach (two claim columns, partial unique indexes, atomic RPCs) is architecturally sound and fits TableFlow's established patterns. But several parts of the design and its source docs are stale or unsafe as written, and must be corrected before implementation.

---

## 2. Confirmed Good Parts

- **No claim system exists today** — verified directly: `orders` table has none of `claimed_by_cook_id`, `claimed_by_waiter_id`, or any assignment table. Cook/Waiter transitions are bare `.update()` calls with no ownership check (`app/dashboard/cook/page.tsx`, confirmed above — `markReady()` does a direct `.update({status:'ready'})` with no claim gate).
- **Race condition is real and unguarded**, exactly as reported: RLS `USING`/`WITH CHECK` only constrain status, not "who." Two Cooks can both pass RLS on the same row.
- **Two-column design (`claimed_by_cook_id`/`claimed_by_waiter_id`) over a separate table** is reasonable given the codebase's existing style — every other feature (priority, payment) is a flat column on `orders`, not a side table. Consistent with project conventions.
- **Partial unique index approach** (`UNIQUE (claimed_by_cook_id) WHERE claimed_by_cook_id IS NOT NULL`) is the correct Postgres idiom for "one active claim per worker" and will work.
- **Atomic conditional UPDATE / RPC-only mutation** is the right fix, and the project already has two working `SECURITY DEFINER` RPC precedents (`mark_order_paid`, `place_order_and_occupy_table`) to follow structurally.
- **`profiles.staff_name` skepticism is justified** — confirmed `profiles` has no `staff_name` column; only `invite_codes.staff_name` exists, and `invite_codes` rows are **auto-deleted** (`20260810000001_invite_codes_auto_delete.sql` — I haven't read this file's content, but its existence plus the "reconstructs names" claim signals this data may not be durably queryable). This needs verification, but the underlying worry — no durable staff-name source on `profiles` — is real and worth resolving in design, not deferred.

---

## 3. Major Issues

### A. The source documents are internally inconsistent about Node 9's status — and Node 9 is actually DONE
Both the Investigation Report and the Chat 16 Master Prompt describe Node 9 as "investigation complete, ready for schema/implementation" / "🔄 ACTIVE." That is **stale**. I found two later migrations already implementing Node 9 in full:
- `20260814000001_node9_schema.sql` — `notifications` + `notification_reads` tables, RLS, **already supports per-person targeting** (`recipient_id uuid REFERENCES profiles(id)`, with a `one_target_only` CHECK constraint), not just role-broadcast.
- `20260817000001_node9_triggers_realtime.sql` — DB triggers (`notify_order_status_change`, etc.) already fire on `orders` status transitions and insert into `notifications`, and Realtime is already enabled on the table.

This matters directly for Node 13:
- Node 13's plan to "reuse the existing Node 9 notifications infrastructure" is valid, and it's actually **easier than the docs assume** — the schema already has a `recipient_id` column built for exactly the "notify Manager, a specific person" case Node 13 needs. No schema change to `notifications` should be needed, only a new trigger/insert at the claim RPC and a new `type` value added to the existing `valid_type` CHECK constraint.
- However, the existing `notify_order_status_change` trigger **already fires role-broadcast notifications on `preparing`/`ready`** transitions. Node 13's claim RPCs update `orders` status too (on completion: `preparing→ready`, `ready→served`). If Node 13 doesn't explicitly account for this, completion will double-fire: the existing Node 9 trigger will send `order_ready`/role-broadcast notifications at the same status changes, on top of whatever Node 13's `order_claimed` notification does. The design doc's Section 10 doesn't address this overlap — it should.

### B. `mark_order_paid` — the design's own template RPC — is not a safe `SECURITY DEFINER` example
Verified content of `mark_order_paid`: it is `SECURITY DEFINER`, has **no `SET search_path`**, and performs **no caller-role check inside the function body** — it relies entirely on the RLS policy already having gated the caller before the RPC call... but RPCs called via `supabase.rpc()` do **not** re-run table RLS unless the function explicitly checks `auth.uid()`/role, because `SECURITY DEFINER` executes with the *function owner's* privileges, bypassing RLS on the tables it touches internally. `mark_order_paid` has no `auth.uid()` check at all — any authenticated user who can call the RPC (which is any logged-in Supabase user by default, unless `REVOKE`/`GRANT` was set explicitly, and I could not find such a grant in the migrations) can call `mark_order_paid` on any `served` order.

This is a **pre-existing vulnerability**, not one Node 13 introduces — but the design doc tells Claude to "follow the same `SECURITY DEFINER` + `LANGUAGE plpgsql` pattern established by `mark_order_paid`." Copying that pattern as-is would replicate the same hole into the new claim RPCs. By contrast, `force_logout_all_staff` **does** check caller role internally (`SELECT role INTO v_role FROM profiles WHERE id = auth.uid(); IF v_role != 'owner' THEN RAISE EXCEPTION`) — that is the pattern to copy, not `mark_order_paid`.

**Required for Node 13 RPCs:** every claim/complete RPC must do its own explicit `auth.uid()` + role + `is_active` check inside the function body (not just rely on RLS), and should set `SET search_path = public, pg_temp` to avoid search-path hijacking, consistent with `SECURITY DEFINER` best practice that this codebase has not consistently followed so far.

### C. Design doc's Section 12 says "close direct mutation paths that bypass claim ownership" but doesn't name the specific existing policies that must be dropped
The current `orders_update_cook` and `orders_update_waiter` RLS policies (confirmed in `20260804000001_node2b_schema_rls.sql`) allow **any** Cook/Waiter to update **any** matching-status order directly via `.update()` — this is exactly the bypass. If these two policies are left in place alongside new claim RPCs, a Cook can simply skip the RPC and call `.update({status:'ready'})` directly from the client, bypassing the claim check entirely, same as today. The design review question list acknowledges this risk (Q4) but the design body doesn't commit to the fix. **Required:** `orders_update_cook` and `orders_update_waiter` must be dropped or rewritten to add `claimed_by_cook_id = auth.uid()` / `claimed_by_waiter_id = auth.uid()` to their `USING` clause, or removed entirely in favor of RPC-only mutation (revoking direct table UPDATE grants for those roles on `orders`).

### D. Cancellation race (Section 7) has a specific hazard the design doc doesn't resolve
Verified: `submitBulkCancel` (per Chat 16 notes) already fires `cancel_active_orders` RPC — I did not fetch this RPC's source, so I cannot confirm its exact locking behavior; this is **UNKNOWN** and must be checked before implementation, not assumed. What is knowable from the design: if the cancellation RPC and a claim RPC run concurrently without both taking a row lock on the same `orders` row (e.g. via `SELECT ... FOR UPDATE` inside both functions), it's possible to end up with a `cancelled` order that still has `claimed_by_cook_id` set (claim won the race after cancellation logically should have blocked it), leaking a Cook's "slot." The design doc flags this as an open question (Q6) but the "Final corrected design" needs to state the actual mechanism, not just note it's unresolved.

---

## 4. Required Design Changes

1. **Update the checkpoint doc**: Node 9 is `IMPLEMENTED`, not `ACTIVE`/investigation-only. Downstream Node 13/12 planning should build on the real Node 9 schema (per-recipient `notifications.recipient_id` already exists).
2. **New `order_claimed` notification** should use `recipient_id` (specific Manager `profiles.id`), not `recipient_role='manager'` — since Node 9's schema already supports exact single-recipient targeting and Manager is a single-person role per the confirmed role cardinality.
3. **Audit and explicitly suppress or coordinate** with the existing `notify_order_status_change` trigger so claim-completion status transitions (`preparing→ready`, `ready→served`) don't produce duplicate/confusing notifications alongside the new claim-specific one.
4. **Every new RPC** (`claim_order_as_cook`, `claim_order_as_waiter`, `complete_order_as_cook`, `complete_order_as_waiter`) must:
   - explicitly check `auth.uid()`, role, and `is_active` inside the function body (do not rely on RLS alone) — follow `force_logout_all_staff`'s pattern, not `mark_order_paid`'s;
   - set `SET search_path = public, pg_temp`;
   - use `SELECT ... FOR UPDATE` on the target order row to serialize against concurrent claim/cancel/complete calls on the same row, given the confirmed cancellation-race hazard.
5. **Drop or rewrite `orders_update_cook` / `orders_update_waiter` RLS policies** so direct client `.update()` calls can no longer bypass the claim RPCs — this must be explicit in the migration, not just implied.
6. **Verify `cancel_active_orders`'s actual locking behavior** (UNKNOWN — not yet inspected) before finalizing the cancellation-race design; don't assume it's safe.
7. **Resolve `staff_name` before implementation**: confirm whether `invite_codes` rows (source of `staff_name`) still exist/are queryable after the auto-delete migration for staff who onboarded long ago — if not, Manager's claimant-name display (Section 9) will break for older staff, and `profiles.staff_name` becomes necessary, not optional.

---

## 5. Final Corrected Node 13 Design (delta from v0.1)

Everything in the submitted `Chat18_Node13_Design_v0.1.md` stands **except**:

- **Notification target:** `order_claimed` inserts use `recipient_id = <manager's profiles.id>` (looked up via `has_role`/role query), not `recipient_role = 'manager'`.
- **RPC bodies:** add explicit `auth.uid()`/role/`is_active` checks and `SET search_path = public, pg_temp` to all four RPCs; wrap the core update in `SELECT ... FOR UPDATE` on the order row.
- **RLS:** migration must explicitly `DROP POLICY orders_update_cook` and `orders_update_waiter` (or rewrite their `USING` clauses to require matching claim ownership), closing the direct-update bypass.
- **Pre-implementation checklist addition:** read `cancel_active_orders` and `invite_codes_auto_delete` migrations in full before writing the Node 13 migration — both are currently UNKNOWN/unverified inputs this design depends on.
- **Notification overlap:** decide explicitly (in the next design pass, not this review) whether `notify_order_status_change`'s existing `preparing`/`ready` role-broadcast fires should be suppressed for claimed orders, or left as-is alongside the new per-person `order_claimed` notice.

Everything else in Section 2–14 of the v0.1 design (data model shape, lifecycle preservation, one-order-at-a-time index strategy, scope boundary) is confirmed to fit the actual codebase and needs no structural change.
