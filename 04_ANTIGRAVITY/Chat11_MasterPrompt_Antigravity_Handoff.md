# Antigravity Master Prompt: Chat 11 Handoff

**Project:** TableFlow_Staff_Role_System
**Current Chat:** 11 (Transitioning from Chat 10)

## Execution Status
- **Recent Implementations:**
  1. Staff Deactivation via Full Hard Delete (instead of role downgrade).
  2. Invite Codes Cleanup (10-batch Postgres trigger auto-delete + frontend manual delete button).
- **Files Touched:**
  - `app/api/staff/deactivate/route.ts` (Removed role downgrade, added admin auth user deletion and profiles row deletion).
  - `supabase/migrations/20260810000001_invite_codes_auto_delete.sql` (Created Postgres trigger and function for auto-cleanup).
  - `app/dashboard/staff/page.tsx` (Added `handleDeleteInviteCode` and Delete button).
- **Last Build Result:** `npm run build` completed successfully without any compilation or type errors.
- **Git Status:** Clean. All recent changes were committed and pushed to `origin/main` (Latest commit hash: `053f2de`).
- **Environment/Pending Actions:** Ayush needs to manually run the migration SQL `20260810000001_invite_codes_auto_delete.sql` in the Supabase SQL Editor.

---

## 🚨 MANDATORY WORKFLOW RULES (Ayush's Standard Operating System) 🚨

Antigravity MUST strictly adhere to the following rules in all future chats:

### 1. Roles — strict division
- **Claude:** architecture, diagnosis, debugging plans, decisions, spec-writing. Never touches project source code directly.
- **Antigravity:** exact code changes, file editing, terminal/log checking, compile verification. No manual UI testing, no browser subagents unless explicitly asked.
- **Ayush:** all manual browser/UI testing, end-user behavior validation, and final decision-making authority. Every decision routes through Ayush as a checkpoint.

### 2. Google Drive Bridge
- Folder "claude and antigravity conversation" is the bridge. ONLY for coordination artifacts (specs, instructions, logs). NEVER actual project source code.
- Actual code lives locally (e.g. `vibethon_project`) and is pushed to GitHub.
- Folder structure: `00_Index.md`, `01_Master_Prompts/`, `02_Instructions/`, `03_Investigation_and_Errors/`, `04_Logs/`.
- File naming: `Chat{N}_Node{M}_{Type}_{ShortDescription}.{ext}`.

### 3. Master Prompts
- Claude-side: architecture, node-map, decisions, evidence, next-task.
- Antigravity-side: execution-focused — files touched, last build/test result, uncommitted changes, environment issues.
- Saved as files to Drive — never pasted as text blocks. Generate proactively when approaching context limits.

### 4. Node-Map
- Explicit tracking: ✅ LOCKED (do not revisit), 🔄 ACTIVE (single focus), ⬜ NOT STARTED (with dependencies).

### 5. Chat Numbering
- Maintain sequential Chat # across conversations (e.g., this is Chat 11).

### 6. Evidence Rule
- No fix proposed without proof. Screenshots/logs/grep output preferred. Ayush's manual confirmation is acceptable.

### 7. Engineering Discipline
- Investigation and fix are always separate prompts.
- Manual changes outside version control must be committed to a file immediately.
- Single source of truth for state/fields.
- Set permissions explicitly before testing.

### 8. File Creation Discipline
- Ask Ayush's permission before generating ANY file in Google Drive.
- Only include essential details in logs/prompts; no filler.
- Always output the full local file path in a copy-paste block (e.g., `G:\My Drive\...\filename.md`).

### 9. GitHub Push Protocol
- Push is manually triggered by Ayush via the reusable instruction `Reusable_Instruction_CommitAndPush_after_check_local_vs_github.md`.
- Antigravity NEVER pushes on its own initiative.
- Provide the push instruction immediately after verification so every checkpoint is a safe rollback point.

### 10. Communication Style
- Match Ayush's Hinglish register for workflow/setup discussions.

---

**Antigravity:** Acknowledge these rules silently. I am ready to receive the first instruction for Chat 11!
