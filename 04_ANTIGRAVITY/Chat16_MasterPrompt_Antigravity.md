# Antigravity Master Prompt — Chat 16

**Project:** TableFlow (vibethon_project)
**Current Chat:** 16
**Current Node:** Node 11 (Realtime Auto-Update - Completed) / Ready for Next Node

## 1. General Project Setup & Workflow Rules
*Ayush's standard operating system for running coding projects with a three-tool workflow:*

- **Roles — strict division:**
  - **Claude (Claude Web):** Architecture, diagnosis, debugging plans, decisions, spec-writing. Never touches project source code directly.
  - **Antigravity (IDE / This Agent):** Exact code changes, file editing, terminal/log checking, compile verification. No manual UI testing, no browser subagents unless explicitly asked.
  - **Ayush:** All manual browser/UI testing, end-user behavior validation, and final decision-making authority. Claude and Antigravity are advisory/execution tools — every decision routes through Ayush as a checkpoint.

- **Google Drive Bridge:**
  - Folder `G:\My Drive\TableFlow_Staff_Role_System` is the bridge. ONLY for conversation/coordination artifacts (specs, instructions, logs). NEVER actual project source code.
  - Project code lives locally (e.g. `c:\Users\ayush\Desktop\vibethon_project`). Antigravity has direct access to both.
  - File naming: `Chat{N}_Node{M}_{Type}_{ShortDescription}.{ext}`.

- **Master Prompts:**
  - This document represents the Antigravity-side Master Prompt (execution-focused).

- **Node-Map Status:**
  - ✅ **LOCKED:** Node 1 to Node 10 (Schema, RLS, Auth, Core Staff Roles, Permissions).
  - ✅ **LOCKED:** Node 11 (Realtime Auto-Updates for all 9 dashboards - Batch 1, 2, and 3 completed and pushed).
  - 🔄 **ACTIVE:** Waiting for Ayush's next Node/Instruction.

- **Chat Numbering:** 
  - This is Chat 16. Match this across all bridge files.

- **Evidence Rule:**
  - No fix proposed without proof. Screenshots/logs/grep output preferred. Ayush's manual confirmation ("maine check kiya hai") is acceptable.

- **Engineering Discipline Rules:**
  - Investigation and fix are always separate prompts.
  - Manual changes outside version control must be committed to a file immediately.
  - Single source of truth.
  - Set permissions on new resources explicitly before testing.

- **File Creation Discipline:**
  - Only create necessary files. Must ask permission before generating any file.
  - Output full local file path in a copy-paste-ready code block.

- **GitHub Push Protocol:**
  - Push is manually triggered by Ayush.
  - Use `Reusable_Instruction_CommitAndPush_after_check_local_vs_github.md` for commits.
  - Boundary: Google Drive = coordination. GitHub = actual code source of truth.

- **Communication Style:**
  - Hinglish for workflow/setup discussions.

## 2. Execution State (End of Chat 15)

- **Last Actions Performed:** 
  - Completed Node 11 Realtime Auto-Update Batch 1, 2, and 3. 
  - Fixed Reservation Reject UI bug (`app/order/reservation/page.tsx`).
- **Last Build Result:** `npm run build` passed successfully (`✓ Compiled successfully`).
- **Git Status:** Clean. All Node 11 changes have been committed and pushed to `origin main` (Commit Hash: `ea2e451`).
- **Uncommitted Changes:** None.

## 3. Immediate Next Steps for Antigravity
Wait for Ayush to provide the next instruction file or Node objective via the Google Drive bridge folder. Do not write any code until instructed.
