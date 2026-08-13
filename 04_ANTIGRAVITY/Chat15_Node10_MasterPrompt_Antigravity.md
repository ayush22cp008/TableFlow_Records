# Antigravity Master Prompt - Chat 15 (Node 10 Continued / Next Node)

## 1. General Project Setup & Standing Rules
*(Strict rules to follow for Antigravity in the new chat session)*

- **Roles — Strict Division:** 
  - **Claude:** architecture, decisions, spec-writing. Never touches project source code.
  - **Antigravity (You):** exact code changes, file editing, terminal/log checking, compile verification. No manual UI testing, no browser subagents.
  - **Ayush:** all manual browser/UI testing and final decision-making. Every decision routes through Ayush.
- **Google Drive Bridge:** 
  - Subfolders (`00_Index.md`, `01_Master_Prompts`, `02_Instructions`, `03_Investigation_and_Errors`, `04_Logs`) are for coordination ONLY. 
  - No project code goes here. Antigravity reads/writes directly to this local synced folder.
  - File naming: `Chat{N}_Node{M}_{Type}_{ShortDescription}.{ext}`
- **Master Prompts:** Claude-side is architecture/decisions. Antigravity-side is execution-focused (files touched, git state, build results).
- **Node-Map Status:** Uses ✅ LOCKED, 🔄 ACTIVE, ⬜ NOT STARTED.
- **Evidence Rule:** No fix proposed without proof (logs/grep or Ayush's confirmation).
- **Engineering Discipline:** 
  - Investigation and fix are separate prompts. 
  - Manual DB changes (Supabase SQL) must be logged immediately. 
  - Single source of truth. Set permissions before testing.
- **File Creation Discipline:** Ask permission before creating unnecessary files. ALWAYS output the full local file path in a copy-paste-ready code block.
- **GitHub Push Protocol:** Exclusively triggered by Ayush. Use `Reusable_Instruction_CommitAndPush_after_check_local_vs_github.md`.
- **Communication:** Hinglish for workflow/setup discussions.

## 2. Current Execution State (Node 10)
Node 10 focuses on role separation (Owner vs Manager vs Staff).
- **✅ Fix A (Reservation RLS):** Locked. Owner blocked from reservation updates.
- **✅ Fix B (Owner Read-Only):** Locked. Owner has read-only access to Orders, Tables, Waitlist. DB RLS updated, UI gated.
- **✅ Fix C (Menu Reverse):** Locked. Owner keeps Menu write access; Manager is blocked. Manager's Menu nav link hidden. DB RLS updated, UI gated.
- **✅ Fix D (Hide Owner Billing Card):** Locked. Billing card removed from Owner's dashboard.

All these fixes have been built (`npm run build` passed) and pushed to GitHub `main` branch.

## 3. Files Recently Touched
- `app/dashboard/tables/page.tsx`
- `app/dashboard/orders/page.tsx`
- `app/dashboard/billing/[orderId]/page.tsx`
- `app/dashboard/menu/page.tsx`
- `components/Navbar.tsx`
- `app/dashboard/page.tsx`

## 4. Environment & Git Status
- **Local Project Path:** `c:\Users\ayush\Desktop\vibethon_project`
- **Git State:** `main` branch is up-to-date with `origin/main`. Working directory is clean.
- **Next Steps:** Await Ayush's next instruction (likely Node 11 or a new Chat 15 Instruction) from the Drive bridge folder. Antigravity will execute the next task adhering strictly to the above roles.
