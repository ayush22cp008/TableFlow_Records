# Instruction — Node 11 Fix: Reservation Rejected State Handling

**Type:** Fix. Implement code changes as specified below.

## Root cause (confirmed via investigation)

`app/order/reservation/page.tsx` — `checkReservation` function only treats `['pending', 'approved', 'arrived']` as valid statuses:
```tsx
if (data && ['pending', 'approved', 'arrived'].includes(data.status)) {
  setActiveReservation(data)
}
```
`'rejected'` is missing. This causes two symptoms from the same root cause:
1. Realtime reject event arrives correctly but is silently ignored (no `else` branch), so UI doesn't update.
2. On page refresh, the same filter causes `activeReservation` to stay `null`, so the form renders again — allowing a duplicate request even though a rejected request already exists.

## What to do

### 1. Include `'rejected'` in the status handling
Update the condition to include `'rejected'`:
```tsx
if (data && ['pending', 'approved', 'arrived', 'rejected'].includes(data.status)) {
  setActiveReservation(data)
}
```

### 2. Add a "Rejected" UI state
In the render logic (wherever `activeReservation` is used to decide what to show), add a case for `activeReservation.status === 'rejected'`:
- Show a clear message: reservation request was declined.
- Show a button/CTA to submit a new request (this should let the customer go back to the request form — e.g. a "Request Again" button that either clears `activeReservation` locally or navigates to the form state).
- Do not auto-show the form directly — the customer must explicitly choose to request again, per Ayush's decision.

### 3. Verify realtime callback and initial fetch use the same logic
Both `checkReservation` (initial load) and the realtime `.on('postgres_changes', ...)` callback should hit the same updated condition — confirm no separate/duplicate status-check exists elsewhere in the file that also needs the same fix.

## Constraints (per project rules)

- Only touch `app/order/reservation/page.tsx`. No other files.
- No DB/schema changes — this is a frontend state-handling fix.
- No changes to `rejectRequest`/`approveRequest` in `app/dashboard/tables/page.tsx` — those already work correctly per investigation.
- After implementation: report files modified, confirm build passes, confirm no unexpected files touched.

## Testing note

Ayush will manually verify: Manager rejects a reservation → customer side should show the rejected message without refresh → customer can click "Request Again" to submit a new one → refreshing the page after a reject should also correctly show the rejected state, not the form.

## Output

Report back with: file modified, build result, and a short description of the UI added for the rejected state.
