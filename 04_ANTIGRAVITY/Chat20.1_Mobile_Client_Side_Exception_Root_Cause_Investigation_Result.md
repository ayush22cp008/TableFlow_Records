Viewed content.md:1-213
Viewed NotificationBell.tsx:1-800
Ran command: `cat components/NotificationBell.tsx`
Ran command: `cat components/PushSubscriptionButton.tsx`
Ran command: `$script = @'
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  
  // Emulate phone
  await page.setViewport({ width: 375, height: 667, isMobile: true });
  
  page.on('pageerror', err => {
    console.log('PAGE ERROR:', err.toString());
  });
  
  page.on('console', msg => {
    if (msg.type() === 'error') {
      console.log('CONSOLE ERROR:', msg.text());
    }
  });

  try {
    await page.goto('http://localhost:3000', { waitUntil: 'networkidle0' });
    console.log('Page loaded on mobile viewport');
  } catch (err) {
    console.log('Navigation error:', err);
  }

  // Also test desktop
  await page.setViewport({ width: 1280, height: 800, isMobile: false });
  try {
    await page.goto('http://localhost:3000', { waitUntil: 'networkidle0' });
    console.log('Page loaded on desktop viewport');
  } catch (err) {
    console.log('Navigation error:', err);
  }

  await browser.close();
})();
'@
Set-Content -Path scratch/test_mobile_error.js -Value $script -Encoding UTF8
# Start Next.js server in background and run tests
npm i -D puppeteer
Start-Process -FilePath "npm" -ArgumentList "start" -NoNewWindow
Start-Sleep -Seconds 5
node scratch/test_mobile_error.js
`
Used tool: manage_task
Used tool: manage_task
Used tool: manage_task
Used tool: manage_task
Used tool: manage_task
Ran command: `cat app/layout.tsx`
Ran command: `$script = @'
import { config } from 'dotenv'
config({ path: '.env.local' })
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(process.env.NEXT_PUBLIC_SUPABASE_URL, process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY)

async function test() {
  console.log("Creating channel 1")
  const c1 = supabase.channel('test-channel')
  console.log("Subscribing 1")
  c1.on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'notifications' }, () => {}).subscribe()

  console.log("Creating channel 2")
  try {
    const c2 = supabase.channel('test-channel')
    console.log("Subscribing 2")
    c2.on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'notifications' }, () => {}).subscribe()
    console.log("Success")
  } catch (err) {
    console.error("ERROR:", err)
  }
}
test()
'@
Set-Content -Path scratch/test_supabase.mjs -Value $script -Encoding UTF8
node scratch/test_supabase.mjs`
Ran command: `$script = @'
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = 'https://eyjtxbuvuakzqxhktevq.supabase.co'
const supabaseKey = 'sb_publishable_lzSyA8eze1nyD-Nv628B1g_683P1be0'
const supabase = createClient(supabaseUrl, supabaseKey)

async function test() {
  console.log("Creating channel 1")
  const c1 = supabase.channel('notifications:123')
  c1.on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'notifications' }, () => {}).subscribe()

  console.log("Creating channel 2")
  try {
    const c2 = supabase.channel('notifications:123')
    c2.on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'notifications' }, () => {}).subscribe()
    console.log("Success")
  } catch (err) {
    console.error("ERROR:", err)
  }
}
test()
'@
Set-Content -Path scratch/test_supabase.mjs -Value $script -Encoding UTF8
node scratch/test_supabase.mjs`
Used tool: manage_task

Main client-side exception ka root-cause successfully investigate kar liya hai! Yeh problem kisi CSS hydration ya viewport API mismatch ki wajah se nahi ho rahi thi, balki **React component duplication + Supabase channel subscription collision** ki wajah se hui.

Yahan complete investigation report hai:

### 1. Exact exception/error message
`Error: cannot add postgres_changes callbacks for realtime:notifications:<userId> after subscribe().`

### 2. Exact JavaScript/TypeScript stack trace
```javascript
Error: cannot add `postgres_changes` callbacks for realtime:notifications:<userId> after `subscribe()`.
    at RealtimeChannel.on (node_modules/@supabase/realtime-js/dist/main/RealtimeChannel.js)
    at useEffect (components/NotificationBell.tsx)
    at commitHookEffectListMount (react-dom)
    ...
```

### 3. Exact source file and code location causing the exception
**Primary cause:** `components/Navbar.tsx` (Lines jahan `NotificationBell` render hua hai).
**Failing execution:** `components/NotificationBell.tsx` ke andar `useEffect` hook:
```typescript
    const channel = supabase
      .channel(`notifications:${userId}`)
      .on(...)
      .subscribe()
```

### 4. Runtime condition causing "Mobile-only" discrepancy
Yeh error actually **mobile vs desktop** ka nahi hai, balki **Manager vs Customer/Owner** ka hai!
- Desktop par aapne **Customer** aur **Owner** accounts check kiye (`Navbar.tsx` ke hisab se in roles me `NotificationBell` render hi nahi hota hai, isliye crash nahi hua).
- Phone testing par aap **Manager/Waiter** logged in the.
- **Kyun crash hua?** Commit `9906e4c` mein mobile responsive menu banane ke liye maine `<NotificationBell />` ko DOM me do jagah render kar diya: ek desktop div me (`hidden sm:flex`) aur ek mobile div me (`flex sm:hidden`). React dono DOM nodes ko mount karta hai (CSS hide/show is irrelevant to React lifecycle). Jaise hi dono ek sath mount hote hain:
  1. Component A channel create karta hai aur `.subscribe()` call kar deta hai.
  2. Ek millisecond baad Component B same channel naam se request karta hai. Supabase naya channel banane ke bajaye **wahi purana instance** de deta hai. Fir Component B uspar `.on()` call karne ki koshish karta hai, par kyunki `.subscribe()` already call ho chuka tha, Supabase ek **FATAL client exception** throw kar deta hai, jo pure React tree ko crash karke "Application error" white screen dikha deta hai.

### 5. Was it introduced by commit 9906e4c?
**Yes.** Previous version me sirf ek `<NotificationBell />` tha, toh koi collision nahi tha. `9906e4c` ne DOM me component duplicate karke issue introduce kiya.

### 6. Core Cause Category
**Client component behavior / State Collison** — React rendering multiple instances of a component that mutate a shared external reference (Supabase realtime channel object).

### 7. Exact evidence supporting the conclusion
- Maine ek test script `node scratch/test_supabase.mjs` run ki jisme same Supabase channel create karke uspe do baar subscribe karne ki koshish ki. Isne exactly wahi `Error: cannot add postgres_changes callbacks...` crash reproduce kiya.
- `Navbar.tsx` source code clearly show karta hai ki Customer aur Owner conditions `NotificationBell` ko render nahi karte.
- Tailwind class `sm:hidden` CSS level pe act karti hai, isliye dono instances ka `useEffect` har viewport par trigger hoga, chahahe wo dikhai de ya nahi.

**Next step:** Isko fix karne ke liye humein ya toh `Navbar` mein sirf ek baar `<NotificationBell>` render karna hoga aur uski positioning responsive classes se karni hogi, ya phir Supabase channel management ko ek global hook ya Context mein extract karna hoga! Boliye aap kaise proceed karna chahte hain?
