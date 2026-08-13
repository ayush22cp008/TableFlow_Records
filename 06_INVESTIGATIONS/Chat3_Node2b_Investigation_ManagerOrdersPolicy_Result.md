# Investigation Result: Manager Orders Policy (Status Values)

Chat #3 | Node 2b

## 1. Actual Status Enum Values in Use
According to the TypeScript schema definition (`types/index.ts`, which serves as the frontend's source of truth for the database schema), the exact enum values for `orders.status` are:

```typescript
export type OrderStatus = 'placed' | 'preparing' | 'ready' | 'served' | 'billed' | 'cancelled'
```

- **"Completed/Paid"** maps exactly to the `'billed'` status.
- **"Cancellation"** maps exactly to the `'cancelled'` status.

## 2. Manager Order Transitions
Based on the Permission Matrix and the actual status enum values, the valid state transitions for a Manager should be:

1. **Intake (Accepting Order):** `placed` -> `preparing`
2. **Billing (Closure):** `served` -> `billed`
3. **Cancellation (Edge case):** 
   - The Permission Matrix does not explicitly scope cancellation.
   - Logically, a Manager should be able to cancel an order *before* it has been cooked. Thus, valid source states for `cancelled` should be `placed` and `preparing`. 
   - (A Manager should not simply "cancel" an order that is already `ready` or `served` without a different flow like comping/voiding, but for this strict DB policy, restricting cancellation from `placed` and `preparing` is the safest approach).

## Next Step
Now that the exact values (`billed`, `cancelled`, `placed`, `preparing`, `served`) are confirmed, we can write the corrected SQL policy using explicit `OLD.status` and `status` checks to guarantee that no cross-product transitions (e.g. `served` -> `preparing`) are permitted.

Please provide the go-ahead, and I will apply the corrected, paired policies!
