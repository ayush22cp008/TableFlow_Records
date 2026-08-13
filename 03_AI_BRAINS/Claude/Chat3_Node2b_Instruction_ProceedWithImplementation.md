Chat #3 | Node 2b | Instruction — Proceed with Implementation Plan

Approved: proceed with the full plan in Chat3_Node2b_Implementation_Plan.md as written.

pg_cron decision: Try scheduling via the standard migration file first. Only if it fails due to superuser/permission error, fall back to providing the SQL for manual execution in the Supabase dashboard — report the exact error before falling back, don't pre-emptively switch.

One check before merging RLS: confirm the Orders RLS policy grants customers INSERT only (own orders) and does NOT grant customers UPDATE on order status — customers must stay view-only on status per the locked Permission Matrix (Chat1_Node1_MasterPrompt_Claude_PermissionMatrix.md). Show the exact policy definitions for orders Insert/Update in your evidence report.

Proceed with schema migration, invite_codes table + cron, RLS overhaul, and frontend changes as scoped. Report back with build/compile evidence and the exact policy SQL per engineering discipline (evidence rule) before this gets merged.
