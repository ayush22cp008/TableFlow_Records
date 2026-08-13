# Investigation: User Delete Failure

## 1. Underlying Error from Logs
An examination of `supabase_logs.json` reveals the exact Postgres error when attempting to delete the user. The error is:
```json
"event_message": "update or delete on table \"users\" violates foreign key constraint \"profiles_id_fkey\" on table \"profiles\""
```
This confirms that the deletion is blocked because the `profiles_id_fkey` constraint on the `profiles` table is restricting the deletion of the referenced row in `auth.users`.

## 2. Foreign Keys Referencing `auth.users(id)`
A review of the schema (`profiles.sql` and `supabase/migrations/`) confirms that **only one table** directly references `auth.users(id)`:
- **`profiles.id`**: References `auth.users(id)`. 
  *Note on Cascade Behavior:* While the `profiles.sql` file currently contains an `ALTER TABLE` statement attempting to add `ON DELETE CASCADE` to `profiles_id_fkey`, the error in the logs proves that this constraint on the live database is currently set to `RESTRICT` (or lacks the cascade directive), making it the primary blocker.

## 3. Other Tables Referencing Users
Instead of referencing `auth.users(id)`, all other user-related tables in the schema reference **`profiles(id)`**. Their behaviors are as follows:

- **`waitlist.customer_id`**: References `profiles(id)` `ON DELETE SET NULL`.
- **`orders.customer_id`**: References `profiles(id)` `ON DELETE SET NULL`.
- **`feedback.customer_id`**: References `profiles(id)` `ON DELETE SET NULL`.
- **`invite_codes.created_by`**: References `profiles(id)` **without any cascade rule** (defaults to `RESTRICT`).

## 4. Conclusion & Risks
The immediate cause of the failure is that the `profiles` table's foreign key (`profiles_id_fkey`) to `auth.users` lacks `ON DELETE CASCADE` in the live database.

**Secondary Risk Discovered:**
Even if we add `ON DELETE CASCADE` to `profiles_id_fkey`, deleting an `owner` or `manager` who has generated invite codes will **still fail**. When `auth.users` cascades to delete the `profiles` row, Postgres will hit the `invite_codes.created_by` constraint, which defaults to `RESTRICT` and will block the `profiles` deletion, thereby rolling back the entire `auth.users` deletion. 

To fully fix user deletion, we must address both the `profiles -> auth.users` constraint and the `invite_codes -> profiles` constraint.
