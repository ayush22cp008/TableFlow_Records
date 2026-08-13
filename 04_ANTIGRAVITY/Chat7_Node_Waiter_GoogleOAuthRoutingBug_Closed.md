# Waiter Google OAuth Routing — Live Verified & Closed

**Status:** ✅ Confirmed fixed by Ayush via live manual test (post-push, Vercel deploy).

- Google Sign-In with waiter invite code → correctly lands on `/dashboard/waiter`. Fixed.
- Manual signup with waiter invite code → still correctly lands on `/dashboard/waiter`. No regression.

Both `app/auth/select-role/page.tsx` and `app/auth/callback/route.ts` waiter branches confirmed working live.

**Pushed to GitHub:** Yes (per standing protocol, pushed before this live test was possible).

## Remaining before Waiter Dashboard (Node 3) can be fully locked

1. Tables R/W access — confirm Waiter role can actually reserve/edit/clear tables via `/dashboard/tables`, not just see the Navbar link.
2. Reservations R/W access — same check.
3. Menu R-only access — confirm Waiter can view Menu but not edit.

Once these are tested, Node 3 (Waiter Dashboard) can be marked ✅ LOCKED.
