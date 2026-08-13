# Fix Plan: Invite Codes — Batch Auto-Delete (Trigger) + Manual Delete

**Type:** Fix (implementation) — based on `Chat10_Investigation_InviteCodesCleanup_Result.md`. One-time cleanup already done manually (see `04_Logs/Chat10_ManualDBChange_InviteCodesCleanup.md`).

## Part 1 — Trigger-based batch-of-10 auto-delete

### [NEW MIGRATION FILE]
Create a Postgres trigger + function on `invite_codes`:
- Fires `AFTER UPDATE ON invite_codes` when `status` changes to `'used'`.
- Function checks: `IF (SELECT count(*) FROM invite_codes WHERE status = 'used') >= 10 THEN` delete the oldest 10 used rows (order by `created_at ASC`, limit 10) `END IF`.
- Keep it self-contained in the trigger function — no app-side changes needed, no new API route touches this.

Follow standing constraint: since Supabase CLI is unavailable, output this as a `.sql` migration file for Ayush to run manually via Supabase SQL Editor. Flag it clearly as "Action Required — manual migration."

## Part 2 — Manual per-row delete button

### [MODIFY] Staff Management frontend — Invite Codes History table
- Add a "Delete" action in the Actions column for every row (both `used` and `unused` status) — same visual pattern as the existing "Delete" action in the Active Staff table.
- On click: call `supabase.from('invite_codes').delete().eq('id', codeId)` directly from the client. RLS policy `invite_codes_owner_all` already grants owner `DELETE` — no new API route required (confirmed in investigation).
- Refresh the Invite Codes History list after successful delete (optimistic UI update or refetch).
- Add a simple confirm step (e.g. `window.confirm` or existing modal pattern already used for staff deactivation) before deleting, to avoid accidental clicks.

## Verification Plan
### Automated
- `npm run build` to confirm no type/syntax errors.

### Manual (Ayush)
- Run the new migration SQL in Supabase SQL Editor.
- Generate several invite codes and mark/simulate enough as "used" to cross the 10-threshold; confirm oldest 10 auto-delete.
- On the Staff Management page, manually delete an individual invite code (both a used one and a pending one) and confirm it disappears from the UI and the DB.
