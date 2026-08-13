# Goal: Invite Codes — Batch Auto-Delete + Manual Delete

**Type:** Implementation Plan (Chat 10)

This plan implements the invite codes cleanup strategy as determined in the investigation. It adds a database-level trigger for auto-deleting batches of 10 used codes and a manual delete button on the frontend.

## Proposed Changes

### 1. Database Trigger (Batch-of-10 Auto-Delete)

#### [NEW] `supabase/migrations/20260810000001_invite_codes_auto_delete.sql`
Create a new Postgres migration file containing:
- A trigger function `auto_delete_used_invite_codes()` that checks `IF (SELECT count(*) FROM invite_codes WHERE status = 'used') >= 10`.
- If true, it deletes the 10 oldest used rows using `DELETE FROM invite_codes WHERE id IN (SELECT id FROM invite_codes WHERE status = 'used' ORDER BY created_at ASC LIMIT 10)`.
- A trigger `trigger_auto_delete_used_invite_codes` that fires `AFTER UPDATE ON invite_codes FOR EACH STATEMENT`.

> [!IMPORTANT]
> Because the Supabase CLI is unavailable, this file will need to be run manually by you in the Supabase SQL Editor.

### 2. Manual Delete Button

#### [MODIFY] `app/dashboard/staff/page.tsx`
- Add a new `handleDeleteInviteCode(codeId: string)` function that:
  - Uses `confirm()` to prevent accidental clicks.
  - Calls `supabase.from('invite_codes').delete().eq('id', codeId)` (RLS natively permits this for owners).
  - Refetches the invite codes history (`fetchData()`).
- Add a "Delete" button next to the "Edit" button in the Invite Codes History table for both `used` and `unused` codes, matching the style of the staff delete button (red text).

## Verification Plan

### Automated Tests
- Run `npm run build` to verify Next.js routing and UI syntax correctness.

### Manual Verification
- **Ayush will manually run the SQL migration** via the Supabase SQL Editor.
- Ayush will manually generate and "use" a few invite codes (simulating signups) to verify the auto-delete triggers at 10.
- Ayush will click "Delete" on an invite code from the dashboard UI to confirm it vanishes.
