# Chat 20.4 — Party Size Mobile Input Bug: Root-Cause Investigation Report

**Date:** 2026-09-25  
**Source File:** `app/order/cart/page.tsx`  
**Status:** Investigation complete. No source code was modified.

---

## 1. Confirmed Root Cause

**The Party Size input on mobile fails to update because `parseInt()` returns `NaN` for partial/intermediate input states that mobile browsers emit during typing, and `|| 1` immediately resets the value back to `1`.**

This is a **React controlled-input edge case** — not a CSS issue, not a mobile browser API incompatibility, and not a viewport issue.

---

## 2. Exact Failing Code (Line 468)

```tsx
onChange={(e) => setPartySize(Math.max(1, parseInt(e.target.value) || 1))}
```

### Step-by-step failure on mobile:

| Step | What happens |
|---|---|
| User focuses input showing `1` | `partySize = 1` |
| User taps backspace | `e.target.value = ""` → `parseInt("") = NaN` → `NaN \|\| 1 = 1` → `setPartySize(1)` |
| React re-renders with `value={1}` | Field snaps back to `1` before user can type `2` |
| User types `2` | On mobile, backspace and keypress fire as separate events; the NaN→1 reset races ahead |

### Why this is worse on mobile than desktop:

- **Desktop:** Chrome's number input +/- stepper buttons fire `change` events with a valid integer string. Select-all then type produces `"2"` in a single event.
- **Mobile:** Mobile soft keyboard interaction differs:
  1. Mobile browsers often **do not show stepper controls** for `type="number"`.
  2. The user must tap, backspace, then type — producing the empty intermediate state `""` → `NaN` → reset to `1`.
  3. Mobile Chrome/Brave on Android fires `input` events on each keypress: backspace fires `""`, causing the React state reset before the next digit arrives.

---

## 3. All 15 Investigation Questions — Answered

| # | Question | Answer |
|---|---|---|
| 1 | Can the mobile user change 1 to 2? | **No** — the backspace intermediate `""` state resets to 1 before `2` can register |
| 2 | What exact browser behavior is observed? | Input briefly clears, React forces `value={1}` back, mobile keyboard has no stepper |
| 3 | Does the input emit the expected input/change event? | Yes, events fire — but `\|\| 1` fallback discards the intermediate empty state |
| 4 | What is `e.target.value` immediately after backspace? | `""` (empty string) |
| 5 | Does `setPartySize` receive the new value? | It receives `1` (from `parseInt("") \|\| 1`), not the intended value |
| 6 | Does React state change then revert to 1? | **Yes — exactly this.** State changes to `1`, not the desired value |
| 7 | Is the input disabled because `verifiedReservation` is truthy? | Only after a code is verified. Fresh cart: `disabled={!!verifiedReservation}` = `false` — **NOT the primary cause** |
| 8 | What is the runtime value of `maxCapacity`? | Fetched from `restaurant_tables.capacity` max — defaults to `4` before fetch returns |
| 9 | Does `max={maxCapacity}` constrain unexpectedly? | No — `max` only activates browser validation, doesn't prevent typing. **Not the cause** |
| 10 | Does the behavior differ across browsers? | Yes — desktop Chrome stepper works; mobile lacks stepper; issue is mobile keyboard interaction |
| 11 | Does issue occur after reservation-code flow? | After reservation, `disabled={true}` and `partySize` is set from reservation — field is locked |
| 12 | Does issue occur on a fresh cart with no reservation? | **Yes — this is the primary failure scenario** |
| 13 | Is there any form/effect resetting `partySize`? | No `useEffect` resets `partySize` after mount. The `\|\| 1` in the handler is the sole cause |
| 14 | Is browser autofill/sanitization involved? | Not the primary cause, but mobile input sanitization makes it worse |
| 15 | Is this a React controlled-input edge case? | **Yes — confirmed root cause** |

---

## 4. Secondary Confirming Evidence

- **Line 27:** `const [partySize, setPartySize] = useState(1)` — initial value is `1`
- **Line 106:** `setPartySize(data.party_size)` — only called during reservation code verification
- **No `useEffect`** or timer resets `partySize` at any point after mount
- **Line 465:** `disabled={!!verifiedReservation}` — only disables after a code is verified; `false` in the fresh-cart scenario

---

## 5. Fix Pattern (Phase 2 Implementation)

The `onChange` handler must allow intermediate input states (empty string) without forcing a reset.

### Recommended fix — string-based state with onBlur normalization:

```tsx
// State: string to allow intermediate states
const [partySizeInput, setPartySizeInput] = useState('1')
// Derived numeric value — used everywhere partySize number is needed
const partySize = Math.max(1, parseInt(partySizeInput) || 1)

// Input:
<input
  type="number"
  value={partySizeInput}
  min={1}
  max={maxCapacity}
  disabled={!!verifiedReservation}
  onChange={(e) => setPartySizeInput(e.target.value)}
  onBlur={(e) => {
    const n = Math.max(1, parseInt(e.target.value) || 1)
    setPartySizeInput(String(n))
  }}
/>
```

This allows the user to clear and type a new number freely. The value is only clamped on blur or at order placement.

### Simpler alternative — skip empty state in onChange:

```tsx
onChange={(e) => {
  const raw = e.target.value
  if (raw === '') return  // allow user to clear field without resetting to 1
  const n = parseInt(raw)
  if (!isNaN(n)) setPartySize(Math.max(1, n))
}}
```

---

## 6. Constraints Preserved in Fix

- Party Size remains a numeric value used for seating/waitlist allocation
- `p_party_size` passed to the order RPC uses the derived numeric `partySize`
- `disabled={!!verifiedReservation}` behavior is correct and must be preserved
- The Reservation page has its own separate `partySize` state — **no changes needed there**

---

## 7. Investigation Conclusion

| Item | Conclusion |
|---|---|
| **Root cause** | `parseInt(e.target.value) \|\| 1` resets to `1` when field is cleared on mobile |
| **Category** | React controlled-input edge case |
| **Mobile-specific** | Yes — mobile soft keyboard produces the intermediate empty-string state more frequently |
| **Introduced by commit `9906e4c`?** | **No** — pre-existing bug unrelated to the mobile navbar changes |
| **Severity** | High — Party Size cannot be changed on mobile, breaking table allocation for all mobile orders |
