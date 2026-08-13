Chat #3 | Node 2b | Investigation Only — Reservation Code Expiry Mechanism

Task: Find exactly how the existing reservation-request 30-min expiry works, so the new invite_codes table can reuse the same mechanism instead of building a parallel one.

Investigate:
1. Table/column — which table stores reservation requests, which column holds expiry (expires_at? created_at + interval?)
2. Mechanism type — pg_cron job, Supabase Edge Function on schedule, read-time trigger check, or client-side filter (excludes expired but doesn't delete)?
3. Deletion logic — does it actually DELETE expired rows, or just mark/filter? Get the exact SQL/function/trigger definition.
4. Location — file path (migration/Edge Function) or Supabase dashboard location (SQL editor, Database > Functions, Database > Cron)
5. Frequency — if scheduled, how often does cleanup run?

Output: report findings only, to 03_Investigation_and_Errors/Chat3_Node2b_Investigation_ReservationExpiryMechanism.md
Include: table/column names, exact mechanism with code/SQL snippet copied verbatim, file path or dashboard location, and confirmation of whether it's directly reusable for invite_codes or needs adaptation.

No fixes, no schema changes, no new code — investigation and evidence only.
