# TableFlow — Chat 19 — Node 12 Push Notifications Design v1.1

**Node:** 12 — Push Notifications  
**Chat:** 19  
**Status:** DESIGN v1.1 — CLAUDE REVIEW CHANGES APPLIED  
**Implementation:** NOT STARTED  
**Source Repository:** https://github.com/ayush22cp008/TableFlow  
**Records Repository:** https://github.com/ayush22cp008/TableFlow_Records

## 1. Design Decision

Claude independently reviewed the Node 12 design against the current TableFlow source and returned:

**FITS WITH CHANGES**  
**APPROVED AFTER DESIGN CHANGES**

This v1.1 applies only the important changes required for implementation readiness.

Optional/future improvements from the Claude review are intentionally not included in Node 12 v1.

### Final architectural direction

Use the existing TableFlow notification system as the single business-event source:

`notifications INSERT`
→ Supabase Database Webhook
→ Next.js server route
→ Web Push/VAPID
→ browser Service Worker
→ OS/browser notification

The existing in-app NotificationBell remains independent:

`notifications INSERT`
→ Supabase Realtime
→ NotificationBell

No second business notification engine is introduced.

---

## 2. Important Changes Applied From Claude Review

### Change 1 — Use Next.js server routes by default

The current TableFlow application is already built around Next.js Route Handlers and server-side Supabase access.

Therefore Node 12 will use Next.js API routes for:

- push subscription registration
- push delivery

Planned new routes:

`app/api/push/subscribe/route.ts`  
`app/api/push/deliver/route.ts`

This avoids introducing a second runtime unless a concrete requirement appears later.

### Change 2 — Web Push runtime must be verified before implementation

The server-side push implementation must be proven compatible with the selected Next.js runtime before normal Node 12 coding begins.

The design therefore has a mandatory runtime spike/checkpoint:

1. verify the selected Web Push/VAPID library runs correctly in the existing Next.js server environment;
2. verify VAPID signing and Web Push dispatch work;
3. only then continue to full Node 12 implementation.

A Node-only library must not be placed into an Edge runtime accidentally.

### Change 3 — Supabase Database Webhook is a hosted configuration step

The Database Webhook is part of the deployment/infrastructure configuration, not the GitHub migration history.

Node 12 must explicitly document and test:

- webhook source table: `notifications`
- event: INSERT
- target: `/api/push/deliver`
- authentication/secret used to protect the delivery endpoint
- actual webhook payload received by the endpoint
- retry/duplicate-delivery behavior

Repository source alone must not be treated as proof that the webhook exists in the live Supabase project.

### Change 4 — Subscription RLS follows the existing ownership model

The new `push_subscriptions` table will use the existing TableFlow security pattern.

Users may only manage subscriptions where:

`auth.uid() = user_id`

Server-side privileged delivery uses the existing server-only admin-client pattern.

No client may register or modify a subscription belonging to another user.

---

## 3. Node 12 Scope

### Included

- browser Web Push
- Notifications API
- Service Worker
- explicit Enable Push permission flow
- push subscription persistence
- multiple subscriptions per user/device/browser
- recipient resolution using existing notification semantics
- VAPID-based server delivery
- invalid subscription deactivation
- notification click handling
- minimal manifest/PWA support required for target browsers
- webhook → delivery route integration
- runtime verification
- duplicate/retry handling required for safe delivery

### Explicitly excluded

- new business notification events
- changes to Node 9 event generation
- changes to Node 13 claiming
- Customer push
- Owner push
- replacing NotificationBell
- email replacement
- redesign of restaurant order workflows
- `push_delivery_log` in v1
- additional observability systems not required for correct delivery

---

## 4. Existing Notification Event Source

Node 12 reuses the existing `notifications` table.

Current notification semantics:

- `recipient_id` = one exact user
- `recipient_role` = role-based audience

Role-based delivery must only target active eligible profiles:

`profiles.is_active = true`

Node 13's `order_claimed` notifications use `recipient_id` for the specific Manager. Node 12 must preserve that behavior exactly.

No Node 9 trigger or Node 13 RPC is rewritten for push.

---

## 5. Push Subscription Data Model

Create:

`push_subscriptions`

Suggested schema:

- `id uuid primary key`
- `user_id uuid not null references profiles(id) on delete cascade`
- `endpoint text not null unique`
- `p256dh text not null`
- `auth text not null`
- `user_agent text null`
- `created_at timestamptz not null default now()`
- `updated_at timestamptz not null default now()`
- `last_success_at timestamptz null`
- `last_failure_at timestamptz null`
- `failure_reason text null`
- `is_active boolean not null default true`

Required supporting index:

`push_subscriptions(user_id)`

### Why endpoint uniqueness is retained

A browser/device subscription endpoint represents the concrete push destination.

The same endpoint should not create multiple active rows.

A user may still have multiple different endpoints for multiple browsers/devices.

---

## 6. Subscription RLS

Authenticated users may:

- SELECT their own subscriptions
- INSERT only rows with their own `user_id`
- UPDATE their own subscriptions
- DELETE their own subscriptions

The policy model should follow existing TableFlow patterns using:

`auth.uid() = user_id`

No role-specific permission is required merely to create a personal push subscription.

Server-side delivery uses the existing privileged admin client and therefore does not rely on client RLS for recipient fan-out.

---

## 7. Subscription Registration Flow

User-facing flow:

1. User opens a supported staff portal.
2. User clicks **Enable Push**.
3. Client checks browser support.
4. Service Worker is registered.
5. Notification permission is requested from the explicit user gesture.
6. Client obtains the application VAPID public key.
7. Client creates a PushSubscription.
8. Client sends subscription data to `/api/push/subscribe`.
9. Protected server route authenticates the user.
10. Server validates the subscription payload and stores/updates the user's subscription.
11. UI displays the current push-enabled state.

Permission denial must not break TableFlow.

The existing in-app notification experience continues to work.

---

## 8. Delivery Architecture

### Final flow

```
Node 9 trigger / Node 13 RPC
        |
        v
notifications INSERT
        |
        +------------------------------+
        |                              |
        v                              v
Supabase Realtime                 Database Webhook
        |                              |
        v                              v
NotificationBell             /api/push/deliver
                                       |
                                       v
                              resolve recipients
                                       |
                                       v
                              push_subscriptions
                                       |
                                       v
                                  VAPID/Web Push
                                       |
                                       v
                                 Service Worker
                                       |
                                       v
                              OS/browser notification
```

The two paths are intentionally separate:

- Realtime is for the existing in-app experience.
- Web Push is for background/browser/OS notification delivery.

---

## 9. Delivery Endpoint Security

The delivery route is server-to-server infrastructure.

It must not be an unauthenticated public endpoint.

The implementation must establish a trusted webhook authentication mechanism so arbitrary internet clients cannot invoke push delivery.

The route must:

- validate the incoming webhook request;
- reject unauthorized requests;
- safely parse the notification record;
- resolve recipients server-side;
- never trust client-supplied recipient authorization;
- never expose the VAPID private key;
- use the server-only Supabase admin client for privileged subscription lookup;
- avoid returning sensitive subscription data.

The VAPID private key must remain server-only in deployment environment variables.

---

## 10. Recipient Resolution

### Direct notification

When:

`recipient_id IS NOT NULL`

resolve exactly that user.

Do not broaden an exact-user notification into role-wide delivery.

### Role notification

When:

`recipient_role IS NOT NULL`

resolve all eligible active users matching that role.

At minimum:

`profiles.role = recipient_role`  
and  
`profiles.is_active = true`

Then load active push subscriptions for those users.

### Current Phase 1 audience

Push delivery follows the current notification audience represented by the application:

- Manager
- Cook
- Waiter

Customer and Owner push remain out of scope.

---

## 11. Service Worker

New file:

`public/sw.js`

Responsibilities:

- receive push messages;
- parse the payload;
- display the OS/browser notification;
- handle `notificationclick`;
- focus an existing TableFlow tab where possible;
- otherwise open the target TableFlow route.

The Service Worker must contain no secrets.

---

## 12. Push Payload

Minimum payload:

```json
{
  "notificationId": "uuid",
  "type": "order_placed",
  "title": "New Order",
  "body": "Order #1234 has been placed",
  "url": "/dashboard/manager",
  "orderId": "uuid",
  "reservationId": "uuid"
}
```

The exact payload must be derived from the notification row and validated server-side.

Do not invent unsupported routes.

Current known staff destinations include:

- `/dashboard/manager`
- `/dashboard/cook`
- `/dashboard/waiter`

The implementation must verify every final notification-type destination against the source repository before coding.

---

## 13. Invalid Subscription Handling

When the Web Push provider reports that a subscription is permanently invalid/expired:

1. mark the subscription `is_active = false`;
2. record `last_failure_at`;
3. store a concise `failure_reason`;
4. do not repeatedly attempt delivery to that inactive subscription.

Temporary failures must not automatically be treated as permanent unless the provider response clearly indicates permanent invalidation.

The in-app NotificationBell remains available as fallback.

---

## 14. Duplicate Webhook / Retry Safety

Database Webhooks may be delivered again because of retry or operational conditions.

Node 12 must therefore explicitly test duplicate webhook invocation.

The implementation must prevent one webhook retry from unnecessarily producing duplicate push notifications.

For v1, this safety must be achieved with the smallest practical mechanism compatible with the chosen delivery flow.

A full `push_delivery_log` system is intentionally deferred.

The implementation plan must define the concrete deduplication mechanism before coding.

---

## 15. No Push Delivery Log in v1

Claude identified `push_delivery_log` as useful but optional for the first version.

For this project scope it is intentionally deferred.

Node 12 v1 only keeps minimal subscription health fields already present on `push_subscriptions`.

A future observability milestone may add detailed per-notification/per-subscription delivery history.

---

## 16. Existing Node Compatibility

### Node 9

Must remain unchanged.

Its existing:

- notification triggers
- notification schema
- Realtime publication
- NotificationBell behavior

continue to operate.

### Node 13

Must remain unchanged.

Its:

- claim RPCs
- claim columns
- claim ownership rules
- Manager `order_claimed` notification

remain the existing business logic.

Node 12 is a consumer of notification events, not a replacement for the producers.

---

## 17. Required Implementation Touchpoints

### New files

`app/api/push/subscribe/route.ts`  
Protected subscription registration endpoint.

`app/api/push/deliver/route.ts`  
Webhook receiver and Web Push delivery endpoint.

`public/sw.js`  
Service Worker.

Additional client component/hook may be introduced where the current source architecture makes the Enable Push control cleanest.

### Existing files likely to change

`app/layout.tsx`  
Service Worker registration / client integration as required.

`components/Navbar.tsx`  
Expose push enablement for the same staff audience as the existing notification experience.

`types/index.ts`  
Only where required for push subscription/payload types.

`package.json`  
Add only the verified Web Push dependency required by the selected Next.js runtime.

### Database

New migration under:

`supabase/migrations/`

for:

- `push_subscriptions`
- indexes
- RLS
- required supporting constraints

No changes to Node 9 or Node 13 business-event generation are planned.

---

## 18. Mandatory Runtime Checkpoint

Before full implementation:

### Gate A

Prove the selected Web Push/VAPID implementation works in the existing Next.js server runtime.

### Gate B

Prove a real browser PushSubscription can be registered and persisted.

### Gate C

Prove a test notification can travel:

`server route → Web Push → Service Worker → browser notification`

### Gate D

Only after A/B/C pass, continue with full notification-event integration.

If the selected runtime/library fails, revise the implementation approach before proceeding.

---

## 19. Mandatory Supabase Infrastructure Checkpoint

Before claiming Node 12 integration complete, verify manually in the live Supabase project:

- Database Webhook exists.
- Webhook watches `notifications` INSERT.
- Webhook target is the correct production delivery endpoint.
- Webhook authentication is configured.
- Actual webhook payload matches the delivery handler.
- Retry behavior is understood and tested.
- Duplicate delivery does not create unacceptable duplicate OS notifications.

GitHub source alone cannot prove these hosted configuration items.

---

## 20. Acceptance Tests

### Permission

- supported browser can enable push;
- permission denial leaves the application functional;
- repeated enable attempts do not create duplicate subscription rows.

### Subscription

- one browser subscription is persisted;
- multiple devices/browsers can coexist;
- re-registration of the same endpoint updates/reuses the existing row;
- user cannot write another user's subscription.

### Event delivery

- Manager receives push for `order_placed`;
- Cook receives appropriate push for `order_preparing`;
- Waiter receives appropriate push for `order_ready`;
- Manager receives `order_claimed`;
- other existing notification events follow their actual source recipients.

### Recipient isolation

- direct user notification reaches only the intended user;
- role notification reaches only active users with that role;
- deactivated users do not receive role-based push.

### Reliability

- invalid/expired subscription becomes inactive;
- temporary delivery failure does not incorrectly deactivate a subscription;
- duplicate webhook invocation does not create an unacceptable duplicate notification.

### Service Worker

- push appears when the TableFlow tab is closed/backgrounded where the browser supports it;
- notification click reaches a valid TableFlow route;
- an existing TableFlow tab is focused when appropriate.

### Compatibility

- desktop Chrome
- desktop Edge
- Android Chrome
- Safari/macOS where supported
- iOS/iPadOS Home Screen web app where supported

Browser support must be verified during implementation rather than assumed.

---

## 21. Implementation Readiness

Node 12 is now:

**DESIGN v1.1 — APPROVED AFTER CLAUDE CHANGES**

The important review findings have been incorporated.

Optional improvements are intentionally deferred.

### Next approved checkpoint

**Create the detailed Node 12 implementation plan.**

Implementation itself has not started.
