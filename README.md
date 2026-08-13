# TableFlow Records Repository

This repository is the canonical shared Records Repository for TableFlow.

It contains project memory, architecture, nodes, AI-brain records, handoffs, approvals, monitor evidence references, tasks, tests and reusable templates. It is separate from the TableFlow Source Repository. 

**CRITICAL RULES:**
- It must never contain credentials, API keys, OAuth tokens, passwords, or secrets.
- It does not authorize source-code modification by itself.
- Ayush remains the final human authority.

## File Naming Convention

**Node/Chat artifacts:**
`Chat{N}_Node{M}_{Type}_{ShortDescription}_v{X}.{ext}`
*Examples:*
- `Chat4_Node3_Investigation_HipAngleGate_v1.md`
- `Chat4_Node3_Instruction_HipAngleFix_v1.md`
- `Chat16_Node9_MasterPrompt_ClaudeHandoff_v1.md`

**Reusable artifacts:**
`Reusable_{ShortDescription}.{ext}`
