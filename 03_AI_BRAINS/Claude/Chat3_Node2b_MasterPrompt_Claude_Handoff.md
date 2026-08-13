# TableFlow Staff Role System — Chat #3 Handoff (start Chat #4 with this)

## Project
TableFlow Staff Role System. Bridge folder: `TableFlow_Staff_Role_System` (flat structure, no subfolders — all files at root).

## Node-Map

- ✅ **LOCKED — Node 1:** Permission Matrix (roles: customer/waiter/cook/manager/owner, dashboard scope per role, order status flow placed→preparing→ready→served→billed).
- ✅ **LOCKED — Node 2a:** Staff invite-code design (7-char code, role tag at random position, editable-while-unused name/email, 30-min expiry).
- ✅ **LOCKED/BUILT — Node 2b, Part 1 — Core schema + RLS + cancellation:**
  - `profiles.role` extended (waiter/cook/manager added), `invite_codes` table with `status` enum (unused/used/expired), pg_cron 2-step expiry (mark expired → batch delete).
  - Full RLS overhaul: paired-transition policies per role (no cross-product bugs) for orders/tables/menu.
  - Order cancellation system: single-order cancel (all 4 roles, reason mandatory, only from placed/preparing/ready) + Owner-only Bulk Emergency Stop (select-specific + cancel-all modes, category dropdown + optional detail, atomic RPC `cancel_active_orders`).
  - Build verified, pushed to GitHub. 3 post-deploy bugs found in manual testing (signup role missing, bulk-cancel silent fail, checkbox click-blocked by modal overlay) — all 3 fixed and build-verified.
- 🔄 **ACTIVE — Node 2b, Part 2 — Invite Email Delivery Redesign:**
  - **Supersedes** original manual-share assumption from Node 2a. New flow: Owner generates code (no email sent) → staff enters email at signup → system finds matching unused/non-expired invite_codes row → emails the code to that address → staff enters code + password → 3-layer validation (code/role/email, unchanged) → account created.
  - Replaces the Supabase OTP step for staff signup ONLY — Owner/Customer signup untouched.
  - Decisions locked: Resend npm SDK (not raw fetch) via new API route, new Resend API key needed (separate from existing Supabase SMTP creds), Admin SDK `createUser()` with `email_confirm:true` for staff account creation (bypasses Supabase's native confirmation email entirely — cleaner than post-hoc confirm).
  - **Last instruction sent, NOT YET confirmed complete:** `Chat3_Node2b_Instruction_ProceedInviteEmailRedesign.md` — Antigravity needs to build: `app/api/send-invite/route.ts` (lookup + Resend send), a server-side account-creation route (Admin SDK createUser + re-validate 3-layer + mark invite used), and redesign `AuthForm.tsx` staff steps (role → email/send-invite → code+password/create-account).

## Parked (not in current scope)
- `Chat3_ParkedIssue_OwnerHardcodedInviteCode.md` — Owner signup uses a hardcoded client-visible string `'TableFlow12'` in AuthForm.tsx (security gap). Explicitly deferred by Ayush to a future node.

## Also locked, not yet executed
- `Chat3_Decision_ReadmeStructure_v2.md` — README rebuild happens at project END: two sections (VibeAthon Submission vs Post-Hackathon Updates), split via `git log` at the 30 July 11:59 PM IST cutoff, cross-checked against Drive decision docs.

## Next action for Chat #4
1. Check Drive for `Chat3_Node2b_Evidence_InviteEmailRedesign.md` (or similarly named) — if Antigravity has reported back, review build evidence + confirm Service Role key stayed server-side only, per evidence rule.
2. If not yet done, resend `Chat3_Node2b_Instruction_ProceedInviteEmailRedesign.md` to Antigravity.
3. Once verified, full manual testing checklist for Node 2b: invite generation, email delivery (check inbox), 3-layer validation (wrong role/email/expired code all reject), role-based dashboard access, order status transitions, single + bulk cancellation, table seat release.
4. Commit + push once testing passes (use `Chat3_Node2b_Instruction_CommitAndPush.md` pattern).
5. After Node 2b fully closes: next node is the customer-side Table Status Board (mentioned earlier, not yet scoped) — or continue further Staff Role System refinements per Ayush's direction.

## Standing rules for Chat #4 (unchanged)
Three-role division, Drive bridge = coordination only, investigation/fix always separate prompts, evidence required before merge, Ayush does all manual testing, push only on Ayush's explicit go-ahead.
