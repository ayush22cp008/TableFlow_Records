# TableFlow — Chat 19 — Node 12 Implementation Plan v1.0

**Node:** 12 — Push Notifications  
**Chat:** 19  
**Status:** IMPLEMENTATION PLAN — READY FOR APPROVAL  
**Design Basis:** `Chat19_Node12_Design_v1.1_Final.md`  
**Claude Review:** `Chat19_Node12_Design_v1.1_Final_claude_accept.md`  
**Source Repository:** https://github.com/ayush22cp008/TableFlow  
**Records Repository:** https://github.com/ayush22cp008/TableFlow_Records

---

## 1. Objective

Implement true browser/OS push notifications for the existing TableFlow notification events without creating a second business notification system.

The existing `notifications` table remains the single event source.

Target architecture:

```
existing Node 9 / Node 13 notification producer
        |
        v
notifications INSERT
        |
        +-------------------------------+
        |                               |
        v                               v
Supabase Realtime                Supabase Database Webhook
        |                               |
        v                               v
NotificationBell              /api/push/deliver
                                        |
                                        v
                                  resolve recipients
                                        |
                                        v
                                  push_subscriptions
                                        |
                                        v
                                  Web Push / VAPID
                                        |
                                        v
                                  public/sw.js
                                        |
                                        v
                                browser / OS notification
```

Implementation must preserve Node 9 and Node 13 business logic.

---

## 2. Implementation Principles

1. Source repository changes are limited to Node 12.
2. Existing notification producers remain unchanged.
3. Existing NotificationBell remains unchanged unless a minimal integration point is required.
4. Next.js Route Handlers are the server runtime.
5. `web-push` is the selected server library for Node runtime.
6. VAPID private key remains server-only.
7. Supabase Database Webhook is configured manually in hosted Supabase.
8. Client subscription data is protected by RLS.
9. No `push_delivery_log` table in v1.
10. Customer and Owner push remain out of scope.
11. Optional improvements are deferred.

---

## 3. Important Pre-Implementation Gate

### Gate A — Web Push Sanity Check

This is a **small implementation sanity check**, not a separate architecture spike.

Before full Node 12 implementation:

1. Add the verified `web-push` dependency.
2. Create/load VAPID configuration in a server-only location.
3. Verify VAPID key handling/signing from a Next.js Node Route Handler.
4. Confirm the package imports/builds in the project's Next.js environment.
5. Confirm no Edge runtime declaration is added to the push routes.

The gate passes when the server can successfully initialize `web-push` and produce a valid VAPID-authenticated push request path.

The current `web-push` package documents VAPID key generation, `setVapidDetails()`, and `sendNotification()` for server-side Web Push. citehttps://www.npmjs.com/package/web-push

Do not spend time creating an Edge/Deno implementation because the approved architecture is Next.js Node runtime.

---

## 4. Phase 1 — Database

### 4.1 New migration

Create one Node 12 migration under:

`supabase/migrations/`

Suggested name:

`20260923000001_node12_push_notifications.sql`

The migration should create `push_subscriptions`.

### 4.2 Table

Fields:

```sql
id uuid primary key default gen_random_uuid(),
user_id uuid not null references profiles(id) on delete cascade,
endpoint text not null unique,
p256dh text not null,
auth text not null,
user_agent text null,
created_at timestamptz not null default now(),
updated_at timestamptz not null default now(),
last_success_at timestamptz null,
last_failure_at timestamptz null,
failure_reason text null,
is_active boolean not null default true
```

Add an index on:

`user_id`

### 4.3 RLS

Enable RLS.

Policies must enforce:

`auth.uid() = user_id`

Required operations:

- SELECT own subscriptions
- INSERT own subscriptions
- UPDATE own subscriptions
- DELETE own subscriptions

No user may create or modify another user's subscription.

### 4.4 Trigger/update behavior

Add only the minimum timestamp-maintenance mechanism needed for `updated_at`.

Do not introduce an unrelated generic audit system.

### 4.5 Migration execution rule

Antigravity prepares/reviews the migration.

Ayush runs the migration manually in Supabase SQL Editor.

After execution:

- verify table exists;
- verify RLS enabled;
- verify policies;
- verify unique endpoint constraint;
- verify user_id index.

---

## 5. Phase 2 — Package and Environment

### 5.1 Dependency

Add:

`web-push`

to `package.json`.

No Firebase Messaging, Workbox, or Edge-specific push package is required.

### 5.2 Server-only environment variables

Required configuration:

- `VAPID_SUBJECT`
- `VAPID_PUBLIC_KEY`
- `VAPID_PRIVATE_KEY`
- webhook-delivery secret/header configuration

Rules:

- private key never shipped to the browser;
- private key never appears in client code;
- private key never appears in logs;
- public VAPID key may be exposed to the client;
- webhook secret must remain server-side.

### 5.3 VAPID key generation

Generate the key pair once and persist it securely.

Do not generate a new VAPID key pair on every deployment or request.

---

## 6. Phase 3 — Subscription Registration API

Create:

`app/api/push/subscribe/route.ts`

### Responsibilities

1. Authenticate the caller using the existing TableFlow auth/session mechanism.
2. Reject unauthenticated requests.
3. Validate incoming PushSubscription shape:
   - endpoint
   - keys.p256dh
   - keys.auth
4. Determine `user_id` from authenticated session.
5. Never accept arbitrary `user_id` from the client as the owner.
6. Upsert by unique endpoint.
7. Ensure the endpoint belongs to the authenticated user.
8. Set/restore `is_active = true`.
9. Update `updated_at`.
10. Return a minimal success response.

### Endpoint registration behavior

Same endpoint + same user:

- update/reuse existing row.

Same endpoint + different user:

- reject or safely reconcile server-side; never silently transfer ownership.

Malformed subscription:

- reject with validation error.

---

## 7. Phase 4 — Service Worker

Create:

`public/sw.js`

### Responsibilities

- receive `push`;
- parse payload;
- display visible notification;
- handle `notificationclick`;
- focus an existing TableFlow window where possible;
- otherwise open the target URL.

### Security

Service Worker must contain:

- no VAPID private key;
- no service-role key;
- no Supabase secret;
- no webhook secret.

### Payload handling

Service Worker should validate the basic expected payload shape before displaying the notification.

---

## 8. Phase 5 — Client Push Enablement

Add a small client-side push control appropriate for the existing staff UI.

Target audience:

- Manager
- Cook
- Waiter

Do not expose Node 12 push enablement to Customer or Owner in Phase 1.

### Client flow

1. Check `window.isSecureContext`.
2. Check Service Worker support.
3. Check Push API support.
4. Register `/sw.js`.
5. Request Notification permission after explicit user interaction.
6. Convert the VAPID public key to the format required by `PushManager.subscribe()`.
7. Subscribe with `userVisibleOnly: true`.
8. Send the subscription to `/api/push/subscribe`.
9. Display enabled/disabled state.
10. Handle denied permission without breaking the application.

Do not request notification permission automatically on page load.

---

## 9. Phase 6 — Push Delivery API

Create:

`app/api/push/deliver/route.ts`

### Responsibilities

1. Authenticate the incoming Database Webhook request.
2. Reject unauthorized requests.
3. Parse the inserted notification record.
4. Validate required fields.
5. Resolve recipients.
6. Load active subscriptions.
7. Construct push payload.
8. Call `webpush.sendNotification()`.
9. Update subscription success/failure health fields.
10. Deactivate permanently invalid subscriptions.
11. Return an appropriate server response.

### Runtime

Use the normal Node.js Route Handler runtime.

Do not add:

`export const runtime = 'edge'`

to this route.

---

## 10. Phase 7 — Recipient Resolution

Use the exact semantics of the existing `notifications` table.

### Direct recipient

When `recipient_id` is set:

- resolve exactly that profile;
- require active eligibility;
- load that user's active subscriptions.

### Role recipient

When `recipient_role` is set:

- select profiles matching the role;
- require `is_active = true`;
- load each eligible user's active subscriptions.

### Important

Do not convert:

`recipient_id`

into role-wide delivery.

Do not expand the notification audience beyond the source row's existing semantics.

---

## 11. Phase 8 — Notification Payload Mapping

The delivery endpoint must map existing notification types to notification metadata.

Minimum fields:

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

### Route rules

Use only routes verified to exist in the source.

Known staff destinations:

- Manager → `/dashboard/manager`
- Cook → `/dashboard/cook`
- Waiter → `/dashboard/waiter`

Before finalizing the mapping, verify all notification types against their actual source consumers.

No invented route may be shipped.

---

## 12. Phase 9 — Duplicate Webhook Protection

Because the same Database Webhook event may be delivered again, the delivery implementation must avoid unnecessary duplicate OS notifications.

### v1 approach

Use the smallest mechanism that can safely recognize an already-processed:

`notificationId + subscription endpoint`

combination without creating a full delivery-log subsystem.

The implementation planner must finalize the exact mechanism before coding.

Do not create:

`push_delivery_log`

for v1.

The deduplication mechanism must be:

- race-safe;
- compatible with multiple subscriptions;
- compatible with webhook retries;
- unable to authorize arbitrary clients to mark notifications as delivered.

---

## 13. Phase 10 — Invalid Subscription Cleanup

When Web Push returns a permanent subscription-invalid response:

- set `is_active = false`;
- set `last_failure_at`;
- save concise `failure_reason`.

For successful delivery:

- set `last_success_at`;
- clear stale failure state where appropriate;
- update `updated_at`.

For temporary failures:

- do not immediately deactivate the subscription.

Do not build a background cleanup service for v1.

---

## 14. Phase 11 — Supabase Database Webhook

This is a manual hosted-Supabase configuration step.

Configure:

- source table: `public.notifications`
- event: INSERT
- target: production `/api/push/deliver`
- secure webhook authentication

### Required verification

After configuration, Ayush must verify in the live Supabase environment:

1. webhook exists;
2. endpoint is correct;
3. payload matches the handler;
4. authentication works;
5. duplicate/retry behavior can occur and is handled;
6. failed delivery responses do not break the database event transaction.

No repository file is treated as proof that the hosted webhook exists.

---

## 15. Phase 12 — Existing Node 9 / Node 13 Compatibility

### Node 9

Do not modify:

- notification event triggers;
- NotificationBell;
- notification_reads;
- Realtime notification flow.

Node 12 reads the events those systems already produce.

### Node 13

Do not modify:

- claim RPCs;
- claim columns;
- claim ownership;
- Manager claim notification semantics.

When `order_claimed` is inserted, Node 12 consumes it exactly like any other notification event.

---

## 16. Phase 13 — Build/Static Verification

Antigravity checks:

- TypeScript compilation;
- Next.js build;
- import compatibility;
- server/client boundary correctness;
- Service Worker file availability;
- absence of server secrets in client bundles;
- API route syntax;
- migration syntax as far as local tooling allows;
- no accidental Edge runtime on push routes.

Antigravity does not claim browser push success from build results alone.

---

## 17. Phase 14 — Manual Browser Testing

Ayush manually verifies browser behavior.

### Permission

- Enable Push works.
- Deny permission does not break TableFlow.
- Re-enable after appropriate browser permission changes works.

### Subscription

- subscription row is created;
- duplicate enable does not create duplicate endpoint rows;
- multiple devices can create multiple subscriptions;
- another user cannot manage someone else's subscription.

### Delivery

Verify:

- Manager receives `order_placed`;
- Cook receives `order_preparing`;
- Waiter receives `order_ready`;
- Manager receives `order_claimed`;
- other supported event types route according to their actual recipients.

### Background

Test with:

- active tab;
- background tab;
- closed tab where browser supports background push.

### Click

Click notification and verify:

- correct route;
- existing TableFlow tab is focused where possible.

### Failure

Verify:

- invalid subscription becomes inactive;
- temporary error does not incorrectly deactivate;
- webhook retry does not create unacceptable duplicate notifications.

---

## 18. Browser Acceptance Matrix

Test at minimum:

### Desktop

- Chrome
- Edge
- Safari/macOS where supported

### Mobile

- Android Chrome
- iPhone/iPad Home Screen web app where supported

Browser-specific limitations must be recorded as actual observed behavior, not assumed from general documentation.

---

## 19. Final Acceptance Criteria

Node 12 cannot be marked complete until:

- [ ] `push_subscriptions` migration applied successfully.
- [ ] RLS prevents cross-user subscription access.
- [ ] `web-push` server dependency works in the normal Next.js Node runtime.
- [ ] VAPID configuration works.
- [ ] browser can create a PushSubscription.
- [ ] authenticated subscription endpoint persists it correctly.
- [ ] Service Worker receives a real push.
- [ ] OS/browser displays the notification.
- [ ] direct-recipient notifications remain direct-recipient.
- [ ] role notifications target only active eligible users.
- [ ] invalid subscriptions are deactivated.
- [ ] temporary failures do not incorrectly deactivate subscriptions.
- [ ] duplicate webhook invocation is safely handled.
- [ ] notification click reaches a valid route.
- [ ] Node 9 NotificationBell still works.
- [ ] Node 9 notification triggers still work.
- [ ] Node 13 order-claiming still works.
- [ ] Node 13 `order_claimed` notification still reaches Manager.
- [ ] Customer/Owner push has not been unintentionally added.

---

## 20. Rollout Order

Implementation must proceed in this order:

```
1. Database migration
2. web-push dependency + VAPID configuration
3. Gate A sanity check
4. Subscription API
5. Service Worker
6. Client Enable Push control
7. Delivery API
8. Recipient/payload mapping
9. Duplicate protection
10. Invalid-subscription handling
11. Supabase Database Webhook configuration
12. Build verification
13. Manual browser E2E testing
14. Node 9 / Node 13 regression verification
15. Final Node 12 closure
```

Do not configure the production webhook to a half-built endpoint.

---

## 21. Rollback Considerations

Node 12 must be removable without breaking the existing notification system.

Rollback target:

- remove/disable Database Webhook;
- remove push client integration;
- disable push delivery endpoint;
- preserve `notifications`, Node 9 Realtime, NotificationBell, and Node 13 notification producers.

The existing in-app notification path must remain functional throughout rollback.

---

## 22. Explicit Non-Goals

Do not add:

- Customer push;
- Owner push;
- Firebase;
- Workbox;
- Edge Functions;
- `push_delivery_log`;
- new business notification events;
- replacement NotificationBell;
- email notification replacement;
- broad PWA framework dependencies;
- unrelated notification UI redesign.

---

## 23. Checkpoint Rules

### Before implementation

- Design v1.1 accepted by Claude.
- This implementation plan completed.
- No code changes yet.

### During implementation

At each major checkpoint:

- Antigravity reports source/build state.
- Ayush manually verifies browser behavior where applicable.
- No GitHub push occurs without explicit approval.

### Database

All migrations are run manually through Supabase SQL Editor.

### Completion

Node 12 is only considered complete after source, database, webhook, browser, and regression verification all pass.

---

## 24. Final Status

**Node 12 Implementation Plan v1.0 — READY FOR IMPLEMENTATION APPROVAL**

Important Claude-required change incorporated:

> Gate A is now a concrete, low-effort `web-push` + VAPID sanity check in a normal Next.js Node Route Handler, not an Edge/Deno architecture spike.

Optional `push_delivery_log` work remains intentionally deferred.

Implementation has **not started**.
