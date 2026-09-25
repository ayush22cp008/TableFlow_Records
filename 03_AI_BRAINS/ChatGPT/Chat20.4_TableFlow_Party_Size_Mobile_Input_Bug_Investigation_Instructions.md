# Chat 20.4 — TableFlow Party Size Mobile Input Bug Investigation

Date: 2026-09-25
Source repository: https://github.com/ayush22cp008/TableFlow
Records repository: https://github.com/ayush22cp008/TableFlow_Records
Scope: Root-cause investigation only
Primary source file: app/order/cart/page.tsx
Implementation status: Do NOT modify source code during this investigation

## 1. Reported Problem

On the TableFlow customer Cart page, the Party Size field displays 1.

On a phone, the user reports that the value does not change from 1 when attempting to enter 2, 3, 4, 5, etc.

Desktop/laptop behavior has not yet been established as failing. The investigation must determine whether the problem is:

- mobile-only
- browser-specific
- route/state-specific
- caused by the controlled React input
- caused by the max/min constraints
- caused by reservation state
- caused by another client-side interaction

Do not assume the root cause from the screenshot alone.

## 2. Current Source Evidence

The current Cart page contains:

State:

const [partySize, setPartySize] = useState(1)

Maximum capacity:

const [maxCapacity, setMaxCapacity] = useState(4)

The page fetches table capacities:

supabase.from('restaurant_tables').select('capacity').then(({ data }) => {
  if (data && data.length > 0) {
    setMaxCapacity(Math.max(...data.map(t => t.capacity)))
  }
})

The Party Size input is:

<input
  type="number"
  value={partySize}
  min={1}
  max={maxCapacity}
  disabled={!!verifiedReservation}
  onChange={(e) => setPartySize(Math.max(1, parseInt(e.target.value) || 1))}
  ...
/>

The value is therefore a controlled React input.

Important behavior in the current handler:

1. The browser supplies e.target.value as text.
2. parseInt() converts that text to an integer.
3. Math.max(1, parsedValue || 1) prevents values below 1 and converts an empty/non-numeric value back to 1.
4. React state then controls the displayed value.

This source code shows a plausible interaction point, but it does NOT prove that this is the root cause.

## 3. Separate Reservation Behavior

The Reservation page also has a partySize state:

const [partySize, setPartySize] = useState('2')

This is a different page and a different implementation.

Do not change or conflate the Reservation page with the Cart Party Size bug unless investigation evidence demonstrates a shared cause.

## 4. Business Purpose of Party Size

Party Size is used by Cart order placement for table allocation.

The normal path checks available tables and compares remaining capacity against partySize.

The placed order RPC receives party size through:

p_party_size: verifiedReservation ? verifiedReservation.party_size : partySize

The waitlist path also stores party_size.

Therefore the investigation must preserve the requirement that Party Size remains a numeric value used for seating/waitlist allocation.

Do not change table-allocation business rules during investigation.

## 5. Primary Investigation Questions

The final investigation must answer:

1. Can the mobile user change 1 to 2?
2. If not, what exact browser behavior is observed?
3. Does the input emit the expected input/change event?
4. What is e.target.value immediately after a key/tap input?
5. Does setPartySize receive the new value?
6. Does React state change and then immediately revert to 1?
7. Is the input disabled because verifiedReservation is truthy?
8. What is the runtime value of maxCapacity?
9. Does max={maxCapacity} constrain the input unexpectedly?
10. Does the behavior differ across Chrome/Brave/mobile browser and desktop browser?
11. Does the issue occur after entering through the reservation-code flow?
12. Does the issue occur on a fresh cart with no verified reservation?
13. Is there any form-level event, rerender, or effect that resets partySize?
14. Is browser autofill/input sanitization involved?
15. Is this a React controlled-input edge case rather than a CSS issue?

## 6. Reproduction Procedure

Reproduce on an actual phone whenever possible.

Start from:

- Customer login
- /order
- Add at least one menu item to cart
- Open /order/cart
- Do not enter a reservation code

Then:

1. Focus Party Size.
2. Try replacing 1 with 2.
3. Try replacing 1 with 3.
4. Try 4.
5. Try 5.
6. Try using keyboard/backspace.
7. Try select-all then type 2.
8. Try browser increment controls if available.
9. Try paste a value such as 3.
10. Try entering 12.
11. Observe whether the value changes temporarily and then returns to 1.

Record exact behavior for every attempt.

## 7. Important Constraint Test

Repeat the test when a valid reservation code has NOT been entered.

Then, if a valid reservation can be safely tested, repeat after applying a reservation code.

Because the input contains:

disabled={!!verifiedReservation}

a verified reservation intentionally disables manual Party Size editing.

Do not classify disabled behavior as a bug unless verifiedReservation is unexpectedly non-null.

## 8. Browser and Device Matrix

At minimum compare:

Phone:
- Chrome Android
- the user's normal mobile browser

Desktop:
- Chrome or Chromium browser

Where possible, use Android remote debugging to inspect the real phone tab.

Record:

- browser name/version
- Android/OS version
- viewport width
- whether physical or virtual keyboard is open
- logged-in role
- route
- reservation-code state
- maxCapacity value

## 9. Runtime Instrumentation

During investigation only, use temporary diagnostics if needed without committing them to the source repository.

Inspect:

e.target.value
typeof e.target.value
partySize before and after change
maxCapacity
verifiedReservation

Determine whether the sequence is:

Expected:
1 -> input event with "2" -> setPartySize(2) -> rendered 2

or:
1 -> input event -> setPartySize(2) -> rerender -> 1

or:
1 -> browser rejects/normalizes input before React receives it

or another sequence.

Capture console evidence where available.

## 10. React Controlled Input Investigation

Specifically investigate whether this handler causes the problem:

onChange={(e) => setPartySize(Math.max(1, parseInt(e.target.value) || 1))}

Test the following scenarios conceptually and with runtime evidence:

- entering a valid single digit such as 2
- clearing the field
- entering an invalid/temporary empty value
- replacing the value by selection
- entering multi-digit values
- pressing backspace

Important question:

If clearing/replacing the field briefly produces an empty string, does the handler immediately convert that state back to 1 and make replacement difficult on the mobile keyboard?

This is a hypothesis to test, not a confirmed cause.

## 11. HTML Number Input Investigation

Determine whether mobile browser behavior for:

type="number"
min="1"
max="<runtime maxCapacity>"

interacts with the React controlled value.

Check:

- native number-input editing behavior
- keyboard behavior
- browser validation
- min/max enforcement
- replacement/select-all behavior
- whether a value above max is accepted temporarily
- whether a value below min is normalized

Do not assume that HTML min/max alone causes the issue.

## 12. maxCapacity Investigation

Determine the actual runtime maxCapacity on the affected account/session.

The initial state is 4 but it can be replaced by the largest restaurant table capacity returned from Supabase.

Verify:

- table query succeeds
- data is non-empty
- capacities are numeric
- Math.max(...) returns a sensible number
- maxCapacity is not 1
- maxCapacity is not 0/NaN/undefined

If maxCapacity unexpectedly becomes 1, determine why, but do not modify the database during investigation.

Compare the same value on desktop and mobile sessions.

## 13. Reservation-Code State Investigation

Check whether verifiedReservation is unexpectedly populated on the failing mobile session.

If verifiedReservation is truthy, the field is intentionally disabled.

Test a fresh cart session with:

reservationCode = ''
verifiedReservation = null

and verify Party Size editing.

## 14. Rerender / Effect Investigation

Inspect whether any React effect or parent component resets partySize after input.

Search the route/component for every:

setPartySize(
partySize

Determine every place that can write to partySize.

Known writes include:

- initial state 1
- reservation verification setting partySize from reservation data
- Party Size onChange handler

Confirm there is no additional reset triggered by:

- cart reload
- auth state change
- table-capacity query
- waitlist check
- navigation
- Notification/Navbar rendering

## 15. Mobile Layout vs Functional Input

Separate functional behavior from visual behavior.

The input already has:

w-full
px-4
py-2.5

and is inside a responsive max-width Cart container.

Do not conclude this is a CSS responsiveness problem merely because it occurs on a phone.

The investigation must determine whether the value/state actually fails to change.

## 16. Regression / Workflow Boundaries

Do NOT modify:

- NotificationBell
- Navbar
- order placement business rules
- table allocation rules
- reservation workflow
- waitlist workflow
- database schema
- Supabase RLS
- authentication
- Node 9
- Node 12
- Node 13

Do not implement a fix in this investigation.

## 17. Required Final Investigation Result

Create:

04_ANTIGRAVITY/Chat20.4_TableFlow_Party_Size_Mobile_Input_Bug_Investigation_Result.md

The result must contain:

### Incident
Exact user-visible behavior.

### Reproduction
Exact route, account role, browser, device, and steps.

### Desktop comparison
Whether the same behavior occurs on laptop/desktop.

### Exact runtime behavior
What input/change events occur and what values are observed.

### Root cause
One precise technical cause supported by evidence.

### Source location
Exact file and relevant code.

### Contributing conditions
Browser/device/state conditions that explain the behavior, if applicable.

### Business impact
Effect on table allocation/order placement/waitlist, if any.

### Fix recommendation
Describe the required implementation direction without implementing it.

### Regression tests
Tests required after implementation.

## 18. Antigravity Execution Sequence

Use this sequence:

Read investigation instructions
  -> inspect current main
  -> reproduce on phone
  -> compare desktop
  -> capture exact runtime behavior
  -> inspect Party Size state/writes
  -> inspect maxCapacity and verifiedReservation
  -> test controlled-input behavior
  -> test number-input behavior
  -> determine root cause
  -> write result report
  -> STOP

Do not commit source-code changes for Chat 20.4 investigation.

## 19. Success Criteria

The investigation is complete only when it answers:

Why does the Party Size control remain at 1 on the affected phone, and what exact code/runtime condition prevents it from changing to 2, 3, 4, 5, etc.?

The final report must distinguish confirmed evidence from hypotheses.

Party Size implementation comes only after the root cause is documented.