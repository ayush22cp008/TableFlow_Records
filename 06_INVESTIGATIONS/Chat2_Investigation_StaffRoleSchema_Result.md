# Result — Investigation: Staff Role Schema

Chat #2 | Node 2

Based on the investigation of the database schema (specifically from the `profiles.sql` file in the project root), here is the report on the `profiles`/`users` table's role structure:

**1. Exact Table Name:**
`profiles` (mapped 1:1 with `auth.users`)

**2. Exact Column Name for Role/Type:**
`role`

**3. Exact Column Type and Definition:**
Type: `text`
Definition: 
```sql
alter table profiles
  add column if not exists role text not null default 'customer'
  check (role in ('customer', 'owner'));
```
*(It uses a CHECK constraint restricting it strictly to 'customer' or 'owner')*

**4. RLS Policies Referencing This Column:**
The `role` column is the backbone of the entire RLS system. It is heavily utilized via a helper function:
```sql
create or replace function is_owner()
returns boolean language sql security definer stable as $$
  select exists (
    select 1 from profiles where id = auth.uid() and role = 'owner'
  );
$$;
```
This `is_owner()` function is used in almost all RLS policies:
- **`profiles` table:**
  - `profiles_owner_read` (select): `using (is_owner());`
- **`menu_items` table:**
  - `menu_public_read` (select): `using (is_available = true and is_active = true or is_owner());`
  - `menu_owner_write` (all): `using (is_owner()) with check (is_owner());`
- **`restaurant_tables` table:**
  - `tables_owner_write` (all): `using (is_owner()) with check (is_owner());`
- **`waitlist` table:**
  - `waitlist_own_read` (select): `using (customer_id = auth.uid() or is_owner());`
  - `waitlist_own_insert` (insert): `with check (customer_id = auth.uid() or is_owner());`
  - `waitlist_owner_update` (update): `using (is_owner());`
- **`orders` table:**
  - `orders_own_read` (select): `using (customer_id = auth.uid() or is_owner());`
  - `orders_own_insert` (insert): `with check (customer_id = auth.uid() or is_owner());`
  - `orders_owner_update` (update): `using (is_owner() or customer_id = auth.uid());`
- **`order_items` table:**
  - `order_items_read` (select) & `order_items_insert` (insert): Allow if the parent order belongs to `auth.uid()` OR `is_owner()` is true.
- **`feedback` table:**
  - `feedback_read` (select): `using (customer_id = auth.uid() or is_owner());`

**5. Foreign Keys / Triggers Tied to This Column:**
There is a trigger on `auth.users` that automatically populates this column upon signup:
```sql
-- Trigger: on_auth_user_created -> executes handle_new_user()
insert into public.profiles (id, email, role)
values (
  new.id, 
  new.email,
  coalesce(new.raw_user_meta_data->>'role', 'customer')
)
```

**6. Sample of Current Distinct Values:**
Querying `SELECT DISTINCT role FROM profiles;` against the live DB returned an empty array `[]`. 
*(Note: There are currently 0 users populated in the profiles table in the active testing environment).*
