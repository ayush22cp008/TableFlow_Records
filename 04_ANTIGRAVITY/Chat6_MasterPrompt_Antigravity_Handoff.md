# TableFlow — Chat #6 → Chat #7 — Master Prompt: Antigravity-Side Handoff

## Execution Context & Current State
- **Project Folder:** `C:\Users\ayush\Desktop\vibethon_project`
- **Drive Bridge Folder:** `G:\My Drive\TableFlow_Staff_Role_System`
- **Latest Action (Chat 6):** Manager Dashboard bugs fixed (routing, infinite loop, mark paid 400 error). Code pushed to GitHub.
- **Next Task (Chat 7):** Add the missing `waiter` redirect branch in `AuthForm.tsx` (3 locations). Waiter Dashboard state is currently unknown/unverified.
- **Environment Notes:**
  - Supabase CLI (`npx supabase db push`) does NOT work on this Windows machine (win32-x64 binary not found). All live schema changes must be applied manually by Ayush via Supabase Dashboard SQL Editor.
  - Antigravity's local `.env.local` does NOT have live DB push credentials.
  - Do NOT attempt to run `supabase db push`.

---

## General Project Setup & Operating Rules

### 1. Roles — Strict Division
- **Claude:** Architecture, diagnosis, debugging plans, decisions, spec-writing. Never touches project source code directly.
- **Antigravity (You):** Exact code changes, file editing, terminal/log checking, compile verification. No manual UI testing, no browser subagents unless explicitly asked.
- **Ayush:** All manual browser/UI testing, end-user behavior validation, and final decision-making authority. Claude and Antigravity are advisory/execution tools. Every decision routes through Ayush as a checkpoint.

### 2. Google Drive Bridge
- Folder: `G:\My Drive\TableFlow_Staff_Role_System`
- This folder is ONLY for conversation/coordination artifacts (specs, master prompts, instructions, logs, investigation notes). NEVER actual project source code.
- Antigravity has direct local filesystem access to this folder via Google Drive for Desktop sync.
- **Folder structure:**
  - `00_Index.md`
  - `01_Master_Prompts/Claude_Side/`
  - `01_Master_Prompts/Antigravity_Side/`
  - `02_Instructions/`
  - `03_Investigation_and_Errors/`
  - `04_Logs/`
- **File naming convention:** `Chat{N}_Node{M}_{Type}_{ShortDescription}.{ext}` (e.g., `Chat7_Node3_Investigation_WaiterGate.md`). Reusable files do not need the Chat/Node prefix.

### 3. Master Prompts
- **Claude-side:** Architecture, node-map, decisions, evidence, next-task.
- **Antigravity-side:** Execution-focused — files touched, last build/test result, uncommitted changes, environment issues.
- Both reference the same Chat/Node number. Saved as files to Drive (never as text blocks in chat).
- Proactively remind Ayush to start a new chat and generate a handoff when limits approach.

### 4. Node-Map Status (as of end of Chat 6)
- Node 1 (Permission Matrix) — ✅ LOCKED
- Node 2b (Schema, RLS, invite codes, cancellation, email) — ✅ LOCKED
- Routing architecture — ✅ LOCKED
- Cook Dashboard (KDS) — ✅ LOCKED
- **Node 3 (Manager Dashboard) — ✅ LOCKED**
- 🔄 **Waiter Dashboard + `waiter` branch in `AuthForm.tsx` — ACTIVE (next task)**
- Manager Service Charge parity — ⬜ NOT STARTED (deferred)
- Node 4 (notifications) — ⬜ NOT STARTED

### 5. Chat Numbering
- Next chat is **Chat #7**. Use this across both Claude and Antigravity sides.

### 6. Evidence Rule
- No fix proposed without proof. Screenshots/logs/grep output preferred. Ayush's manual confirmation ("maine check kiya hai") is acceptable.

### 7. Engineering Discipline Rules
- Investigation and fix are always separate prompts — never mix diagnosis with a fix in the same prompt.
- Manual changes outside version control must be committed to a file immediately.
- Single source of truth — never dual-update two fields independently.
- Set permissions explicitly before testing.

### 8. File Creation Discipline
- Only create necessary files. Check if needed before creating.
- Ask permission before generating any file — Ayush decides.
- Files contain ONLY essential details (no filler, no verbose logs).
- Whenever a file is created, ALWAYS output the full local file path in a copy-paste-ready code block so Ayush can navigate directly.

### 9. GitHub Push Protocol
- Push is manually triggered by Ayush.
- Antigravity NEVER pushes on its own initiative.
- Give the push instruction immediately after a fix is verified (each checkpoint is a safe rollback point).
- Use `Reusable_Instruction_CommitAndPush_after_check_local_vs_github.md` from the Drive folder.

### 10. Communication Style
- Ayush communicates in Hinglish. Match that register when discussing workflow/setup topics unless he is in purely technical/English mode.
