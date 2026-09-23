# Chat 19 — Node 12 — Next Action & Manual Test Checklist

**Current Antigravity report status:** CODE COMPLETE — AWAITING MANUAL WEBHOOK CONFIG & E2E TESTING

## 1. Current State

Antigravity reports that the Node 12 implementation is complete locally and has reached the manual rollout checkpoint.

Reported completed work includes:

- `web-push` installed
- VAPID helper created
- Gate A Node runtime sanity check completed
- `push_subscriptions` migration created and manually executed
- subscription API implemented
- push UI implemented inside the existing NotificationBell dropdown
- `public/sw.js` created
- push delivery API created
- recipient resolution implemented
- permanent 404/410 subscription invalidation implemented
- Node 9 NotificationBell/Realtime path preserved
- Node 13 claiming implementation preserved

## 2. Immediate Next Step

Do **not** mark Node 12 complete yet.

The next action is:

**Configure the live Supabase Database Webhook and perform manual browser E2E verification.**

## 3. Supabase Webhook Configuration

In the live Supabase project configure a Database Webhook:

- Table: `public.notifications`
- Event: INSERT
- Target: production TableFlow push-delivery endpoint
- Authentication: secure secret/header expected by `app/api/push/deliver/route.ts`

Before testing, confirm the production deployment contains the Node 12 code and the required server environment variables.

Required server-side configuration must include:

- VAPID subject
- VAPID public key
- VAPID private key
- webhook authentication secret

Never expose the VAPID private key or webhook secret to the browser.

## 4. Important Production Check

The Antigravity report says implementation is local and has not yet been pushed to the TableFlow source repository.

Therefore:

1. First complete local/preview verification that the implementation is usable.
2. Push to the source repository only after the planned verification checkpoint and explicit approval.
3. Confirm Vercel/production deployment contains the Node 12 routes and Service Worker.
4. Only then configure the production webhook to target the deployed delivery route.

Do not point the live webhook at a deployment that does not contain the delivery endpoint.

## 5. Manual E2E — First Test

Use an existing Manager account.

### Step A — Enable Push

Open:

`/dashboard/manager`

Open the NotificationBell/dropdown.

Use the new Push enable control.

Expected:

- browser permission prompt appears after the explicit user action;
- allow permission;
- subscription request succeeds;
- `push_subscriptions` contains the Manager's subscription;
- UI reports push enabled.

### Step B — Generate a Real Notification

Create a normal TableFlow customer order that produces the existing:

`order_placed`

notification for Manager.

Expected:

1. `notifications` receives the normal Node 9 event.
2. NotificationBell still receives it through Realtime.
3. Database Webhook receives the INSERT.
4. `/api/push/deliver` accepts the webhook.
5. Manager push subscription is resolved.
6. Browser displays an OS/browser notification.

## 6. Manual E2E — Node 13 Compatibility

After Manager `order_placed` succeeds, verify:

- Cook receives the appropriate `order_preparing` push.
- Waiter receives the appropriate `order_ready` push.
- Manager receives `order_claimed` push after a successful Cook/Waiter claim.

This confirms Node 12 consumes existing events without modifying Node 13.

## 7. Manual E2E — Permission Denied

On a test browser:

1. deny notification permission;
2. continue using TableFlow.

Expected:

- application remains fully functional;
- NotificationBell continues working;
- no uncaught client failure;
- no invalid subscription row is created.

## 8. Manual E2E — Multiple Subscriptions

Test with:

- two different browsers/devices for the same user.

Expected:

- two distinct valid endpoints may coexist;
- the same endpoint should not create duplicate rows;
- one notification may be delivered to each active subscription belonging to the intended user.

## 9. Manual E2E — Invalid Subscription

Use a subscription that returns a permanent 404/410 response when practical.

Expected:

- subscription becomes `is_active = false`;
- failure timestamp/reason is recorded;
- future delivery attempts skip that inactive subscription.

## 10. Manual E2E — Notification Click

Receive a real push.

Click it.

Expected:

- TableFlow opens/focuses the intended existing dashboard route;
- no nonexistent route is used.

## 11. Manual E2E — Background/Closed Tab

Test:

- active tab;
- background tab;
- closed tab where browser support allows.

Expected:

- Service Worker receives the push when supported;
- notification remains visible;
- clicking it opens/focuses TableFlow.

## 12. Manual E2E — Duplicate Webhook

Invoke or reproduce duplicate webhook delivery for the same notification/subscription combination.

Expected:

- duplicate webhook execution does not create an unacceptable duplicate OS notification.

Record the actual behavior.

## 13. Node 9 Regression

After push testing verify:

- NotificationBell still displays notifications;
- unread count still works;
- marking notification read still works;
- existing Realtime notification delivery still works.

No Node 9 regression should be introduced.

## 14. Node 13 Regression

Verify:

- Cook claiming still works;
- Waiter claiming still works;
- claim exclusivity still works;
- completion ownership still works;
- Manager still receives `order_claimed`;
- existing claim UI is unchanged.

## 15. Browser Matrix

At minimum, record observed results for:

- Desktop Chrome
- Desktop Edge
- Android Chrome
- Safari/macOS where supported
- iPhone/iPad Home Screen web app where supported

Do not claim browser compatibility solely from documentation.

## 16. Evidence To Record

For each successful checkpoint, capture:

- browser used;
- user role;
- action;
- expected result;
- actual result;
- screenshot where useful;
- relevant Supabase row/log evidence;
- timestamp.

## 17. Current Decision

**Node 12 is NOT closed yet.**

Current state:

`Code → Complete locally`  
`Database migration → Applied`  
`Webhook → Pending`  
`Browser E2E → Pending`  
`Regression → Pending`  
`Production verification → Pending`  
`Node 12 closure → Pending`

## 18. After All Tests Pass

Only after the manual E2E/regression evidence passes:

1. update the Node 12 implementation report;
2. record final source commit/deployment information;
3. document the live webhook configuration;
4. perform final production smoke test;
5. create the Node 12 closure record;
6. lock Node 12.

