# Reusable Prompt — Commit & Push to GitHub

Use this anytime local code needs to be pushed live. Generic — works for any project, no build command assumed.

## Prompt to give Antigravity

```
Check git status and git diff against the current GitHub branch to see exactly what's changed locally vs what's on GitHub.
Stage and commit all changed files with a short, clear message describing what changed (based on the diff).
Push to GitHub (origin main, or current branch).
Report back: files changed, commit message used, push success/failure, and the commit hash.
```

## Notes
- No build/compile step here — verification that the fix actually works already happened earlier (per evidence rule) before this prompt is given.
- If deployment is connected to GitHub (e.g. Vercel), deployment will trigger automatically after push — check deployment status separately if needed.

