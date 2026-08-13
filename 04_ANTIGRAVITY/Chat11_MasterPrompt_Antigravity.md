# Antigravity-Side Master Prompt (Chat 11)

**Role:** Antigravity (IDE Execution & Implementation)

## 1. General Project Setup & Workflow Rules
We strictly follow Ayush's 3-tool workflow:
- **Roles:** Claude (Architecture/Decisions), Antigravity (Code Execution/Terminal/Builds), Ayush (Manual UI testing/Final Decisions).
- **Google Drive Bridge:** All instructions, master prompts, logs, and artifacts go to `G:\My Drive\TableFlow_Staff_Role_System`. No source code goes here.
- **Master Prompts:** Saved in Drive. Cross-reference Chat/Node numbers.
- **Node-Map:** Must be included (see below).
- **Chat Numbering:** Kept in sync between Claude and Antigravity.
- **Evidence Rule:** No fix proposed without proof.
- **Engineering Discipline:** Separate investigation from fixes. Manual DB changes must be scripted. No dual-updating of derived state. Set permissions first.
- **File Creation:** Ask permission before creating unnecessary files. Provide full local paths.
- **GitHub Push:** Triggered manually by Ayush. Use the reusable push prompt.
- **Communication:** Match Ayush's Hinglish register.

## 2. Node-Map Status
- ✅ **Node 1 (Permission Matrix):** LOCKED. (Includes Waiter scope override from Chat 7).
- ✅ **Node 2b (Staff Roles & Invites):** LOCKED.
- ✅ **Node 3 (Manager Dashboard):** LOCKED.
- ✅ **Node 4 (Reservation Lifecycle):** LOCKED.
- ✅ **Node 5 (Owner Staff Management):** LOCKED. (Includes Active/Inactive real-time status and Role-gate deactivated staff enforcement).
- ⬜ **Next Node:** NOT STARTED. Awaiting Claude/Ayush instructions.

## 3. Execution & Environment Status
- **Current Branch:** `main`
- **Last Build:** `npm run build` completed successfully (Next.js 14).
- **Uncommitted Changes:** None. The latest fix (Role-gate enforcement & Staff re-activation) was successfully committed and pushed to GitHub (Hash: `c49bdae`).
- **Recent Files Touched:**
  - `middleware.ts` (Added is_active check & role downgrade to customer)
  - `app/api/staff/deactivate/route.ts` (Removed permanent ban, added role downgrade)
  - `app/api/auth/staff-signup/route.ts` (Added reactivation logic for existing profiles)
  - `supabase/migrations/20260809000007_enforce_active_role.sql` (Enforced is_active in RLS)
- **Environment:** Supabase SQL Editor must be manually synced by Ayush for any new `.sql` migrations.

## 4. Next Task
Waiting for the next `Instruction` file from Claude via Google Drive to begin investigation or execution.
