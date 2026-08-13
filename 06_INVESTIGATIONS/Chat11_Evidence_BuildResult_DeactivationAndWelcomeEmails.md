# Evidence: Build Result for Deactivation & Welcome Emails

**Context:** Implementation of Resend email logic for Staff Deactivation and Welcome Welcome emails (Chat 11).

## Build Command Output (`npm run build`)
```text
> tableflow@0.1.0 build
> next build

  ▲ Next.js 14.2.35
  - Environments: .env.local

   Creating an optimized production build ...
 ✓ Compiled successfully
   Linting and checking validity of types ...
   Collecting page data ...
   Generating static pages (30/30) ...
 ✓ Generating static pages (30/30)
   Finalizing page optimization ...
   Collecting build traces ...

Route (app)                              Size     First Load JS
┌ ○ /                                    175 B          96.2 kB
├ ○ /_not-found                          873 B          88.2 kB
├ ƒ /api/auth/staff-signup               0 B                0 B
├ ƒ /api/auth/verify-invite              0 B                0 B
├ ƒ /api/hello                           0 B                0 B
├ ƒ /api/insights                        0 B                0 B
├ ƒ /api/send-invite                     0 B                0 B
├ ƒ /api/staff/deactivate                0 B                0 B
├ ƒ /auth/callback                       0 B                0 B
├ ○ /auth/select-role                    1.83 kB         168 kB
├ ○ /dashboard                           2.5 kB          165 kB
├ ○ /dashboard/analytics                 3.23 kB         166 kB
├ ƒ /dashboard/billing/[orderId]         3.38 kB         166 kB
├ ○ /dashboard/cook                      2.82 kB         166 kB
├ ○ /dashboard/insights                  3.1 kB          166 kB
├ ○ /dashboard/manager                   3.84 kB         167 kB
├ ○ /dashboard/menu                      3.35 kB         166 kB
├ ○ /dashboard/orders                    4.92 kB         168 kB
├ ○ /dashboard/staff                     4.39 kB         167 kB
├ ○ /dashboard/tables                    4.87 kB         168 kB
├ ○ /dashboard/waiter                    2.87 kB         166 kB
├ ○ /login                               194 B           166 kB
├ ○ /order                               3.29 kB         166 kB
├ ○ /order/cart                          4.51 kB         167 kB
├ ○ /order/my-orders                     3.16 kB         166 kB
├ ○ /reserve                             2.43 kB         165 kB
├ ○ /reserve/status                      2.63 kB         165 kB
└ ○ /signup                              194 B           166 kB
+ First Load JS shared by all            87.3 kB
  ├ chunks/117-fc73271fc24037f1.js       31.7 kB
  ├ chunks/fd9d1056-5f55067fad1d0619.js  53.6 kB
  └ other shared chunks (total)          1.95 kB

ƒ Middleware                             84.2 kB

○  (Static)   prerendered as static content
ƒ  (Dynamic)  server-rendered on demand
```

**Status:** Clean. No compilation or type errors found. Ready for manual UI/Email testing by Ayush.
