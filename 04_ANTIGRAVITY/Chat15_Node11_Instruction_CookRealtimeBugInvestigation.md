# Instruction — Node 11 Bug Investigation: Cook Dashboard Realtime Dies on Tab Switch

**Type:** Investigation only. Do NOT write/modify any code. Report findings only.

## Bug report (from Ayush, manual testing)

- Cook Dashboard (`app/dashboard/cook/page.tsx`) auto-updates correctly while the tab stays active/focused.
- After switching to another app/tab and coming back, the realtime updates stop working — manual refresh is required again.
- Manager Dashboard and Waiter Dashboard do NOT show this issue (per Ayush's testing so far) — only Cook Dashboard.

## What to investigate and report

### 1. Compare subscription setup across all 3 files
Show the exact `useEffect`/subscription setup code (channel creation, `.subscribe()`, cleanup/unsubscribe on unmount) side-by-side for:
- `app/dashboard/manager/page.tsx` (`manager_orders_realtime`)
- `app/dashboard/cook/page.tsx` (`cook_orders_realtime`)
- `app/dashboard/waiter/page.tsx` (`waiter_orders_realtime`)

Report any differences in: dependency array, cleanup function, where the subscription is set up (top-level component vs nested), any conditional logic wrapping the subscription.

### 2. Check for visibility/focus-related code
- Does `cook/page.tsx` have any `document.visibilitychange`, `window.onfocus`, or similar browser tab-visibility handling? Compare to manager/waiter — do they have it and cook doesn't, or does none of them have it?
- Does Cook dashboard have any other `useEffect` that might be tearing down/re-running the subscription unexpectedly (e.g. a state variable in the dependency array that changes when tab loses focus)?

### 3. Check Supabase client-level reconnection behavior
- Is there a shared Supabase client instance used across all pages, or is each page creating its own?
- Report the Supabase JS client version in use (`package.json`) — some versions have known behavior differences around realtime reconnection after socket disconnect.

### 4. Any console errors
- If reproducible, note whether the browser console shows any WebSocket disconnect/error messages when this happens (Ayush can check this manually if asked — don't fabricate, just flag if this needs manual browser console check).

## Explicitly out of scope
- No fixes. Report only what exists today and the exact diff between the 3 files.

## Output

Save findings to: `03_Investigation_and_Errors/Chat15_Node11_Investigation_CookRealtimeTabSwitchBug.md`
