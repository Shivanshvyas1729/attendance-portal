

```sql
-- =====================================================
-- 1️⃣ CLASSROOM SETTINGS TABLE
-- =====================================================

create table if not exists classroom_settings (
  class_name text primary key,
  code text not null,
  daily_limit int not null default 10,
  is_open boolean not null default false
);


-- =====================================================
-- 2️⃣ ROLL MAP TABLE
-- =====================================================

create table if not exists roll_map (
  class_name text
    references classroom_settings(class_name)
    on delete cascade,
  roll_number integer not null,
  name text not null,
  primary key (class_name, roll_number)
);


-- =====================================================
-- 3️⃣ ATTENDENCE TABLE
-- =====================================================

create table if not exists attendence (
  class_name text
    references classroom_settings(class_name)
    on delete cascade,
  roll_number integer not null,
  name text not null,
  attendance_date date not null default current_date,
  attendance_time time not null default current_time,
  primary key (class_name, roll_number, attendance_date)
);


-- =====================================================
-- 4️⃣ ENABLE ROW LEVEL SECURITY
-- =====================================================

alter table classroom_settings enable row level security;
alter table roll_map enable row level security;
alter table attendence enable row level security;


-- =====================================================
-- 5️⃣ CLASSROOM_SETTINGS POLICIES
-- =====================================================

create policy "classroom_select"
on classroom_settings
for select
using (true);

create policy "classroom_insert"
on classroom_settings
for insert
with check (true);

create policy "classroom_update"
on classroom_settings
for update
using (true)
with check (true);

create policy "classroom_delete"
on classroom_settings
for delete
using (true);


-- =====================================================
-- 6️⃣ ROLL_MAP POLICIES
-- =====================================================

create policy "roll_map_select"
on roll_map
for select
using (true);

create policy "roll_map_insert"
on roll_map
for insert
with check (true);

create policy "roll_map_update"
on roll_map
for update
using (true)
with check (true);

create policy "roll_map_delete"
on roll_map
for delete
using (true);


-- =====================================================
-- 7️⃣ ATTENDENCE POLICIES
-- =====================================================

create policy "attendence_select"
on attendence
for select
using (true);

create policy "attendence_insert"
on attendence
for insert
with check (true);

create policy "attendence_update"
on attendence
for update
using (true)
with check (true);

create policy "attendence_delete"
on attendence
for delete
using (true);
```

---

# 🧠 How Your Current Structure Works

### 1️⃣ classroom_settings

Stores:

* class_name
* code
* daily_limit
* is_open

---

### 2️⃣ roll_map

Linked to classroom_settings using:

```sql
references classroom_settings(class_name)
on delete cascade
```

Meaning:

If a class is deleted →
All its students are automatically deleted.

---

### 3️⃣ attendence

Also linked to classroom_settings.

If a class is deleted →
All attendance records are deleted automatically.

---

# 🎯 Important

This setup:

✔ Matches exactly what you created
✔ No extra tables
✔ No extra columns
✔ No teacher system added
✔ No modifications to your design



----------------------------
---

# 📊 DATABASE DIAGRAM (ER Representation)

```
+------------------------+
|   classroom_settings   |
+------------------------+
| 🔑 class_name (PK)     |
|    code                |
|    daily_limit         |
|    is_open             |
+------------------------+
            |
            |  (1 → Many)
            |  class_name (FK)
            ↓
+------------------------+
|        roll_map        |
+------------------------+
| 🔑 class_name (PK, FK) |
| 🔑 roll_number (PK)    |
|    name                |
+------------------------+


            |
            |  (1 → Many)
            |  class_name (FK)
            ↓
+------------------------------+
|          attendence          |
+------------------------------+
| 🔑 class_name (PK, FK)       |
| 🔑 roll_number (PK)          |
| 🔑 attendance_date (PK)      |
|    name                      |
|    attendance_time           |
+------------------------------+
```

---

# 🔗 Relationship Explanation

## 1️⃣ classroom_settings → roll_map

* One classroom
* Can have many students

Relationship:

```
classroom_settings (1)
        ↓
roll_map (Many)
```

Connected by:

```
roll_map.class_name
→ references classroom_settings.class_name
```

If classroom is deleted →
All students are deleted (CASCADE)

---

## 2️⃣ classroom_settings → attendence

* One classroom
* Can have many attendance records

Relationship:

```
classroom_settings (1)
        ↓
attendence (Many)
```

If classroom deleted →
All attendance records deleted.

---

# 📌 Important Note

Currently:

`attendence` is NOT directly connected to `roll_map`
(There is no foreign key linking roll_number + class_name together)

So technically:

* Attendance can exist even if student not in roll_map
* Database does not enforce student validity

But that is your current design — I did not modify it.

---

# 🏗 Visual Flow Structure

```
classroom_settings
        ↓
   roll_map
        ↓
   attendence
```

---

# 🎯 Type of Relationships

| From               | To         | Type        |
| ------------------ | ---------- | ----------- |
| classroom_settings | roll_map   | One-to-Many |
| classroom_settings | attendence | One-to-Many |



Perfect 👍
I’ll give you:

1. ✅ Clean explanation of Supabase RLS syntax (short + clear)
2. ✅ All your Supabase tables (classroom, roll_map, attendance)
3. ✅ One dummy table
4. ✅ Full RLS policies for all tables
5. ✅ Everything in one structured SQL file with `-- headings`

You can paste this directly into Supabase SQL Editor.

---

# 🚀 COMPLETE SUPABASE ATTENDANCE SETUP (TABLES + RLS + DUMMY)

```sql
-- =====================================================
-- SUPABASE ATTENDANCE PORTAL - FULL SETUP
-- =====================================================


-- =====================================================
-- 1️⃣ CLASSROOM SETTINGS TABLE
-- =====================================================

create table if not exists classroom_settings (
  class_name text primary key,
  code text not null unique,
  daily_limit int not null default 10 check (daily_limit > 0),
  is_open boolean not null default false,
  created_at timestamptz default now()
);


-- =====================================================
-- 2️⃣ ROLL MAP TABLE
-- =====================================================

create table if not exists roll_map (
  class_name text not null
    references classroom_settings(class_name)
    on delete cascade,
  roll_number integer not null,
  name text not null,
  created_at timestamptz default now(),
  primary key (class_name, roll_number)
);


-- =====================================================
-- 3️⃣ ATTENDENCE TABLE
-- =====================================================

create table if not exists attendence (
  class_name text not null
    references classroom_settings(class_name)
    on delete cascade,
  roll_number integer not null,
  name text not null,
  attendance_date date not null default current_date,
  attendance_time time not null default current_time,
  created_at timestamptz default now(),
  primary key (class_name, roll_number, attendance_date)
);


-- =====================================================
-- 4️⃣ DUMMY TABLE (FOR TESTING RLS)
-- =====================================================

create table if not exists dummy_table (
  id uuid primary key default gen_random_uuid(),
  message text,
  created_at timestamptz default now()
);


-- =====================================================
-- 5️⃣ ENABLE ROW LEVEL SECURITY (RLS)
-- =====================================================

alter table classroom_settings enable row level security;
alter table roll_map enable row level security;
alter table attendence enable row level security;
alter table dummy_table enable row level security;


-- =====================================================
-- 6️⃣ CLASSROOM_SETTINGS POLICIES
-- =====================================================

create policy "classroom_select"
on classroom_settings
for select
using (true);

create policy "classroom_insert"
on classroom_settings
for insert
with check (true);

create policy "classroom_update"
on classroom_settings
for update
using (true)
with check (true);

create policy "classroom_delete"
on classroom_settings
for delete
using (true);


-- =====================================================
-- 7️⃣ ROLL_MAP POLICIES
-- =====================================================

create policy "roll_map_select"
on roll_map
for select
using (true);

create policy "roll_map_insert"
on roll_map
for insert
with check (true);

create policy "roll_map_update"
on roll_map
for update
using (true)
with check (true);

create policy "roll_map_delete"
on roll_map
for delete
using (true);


-- =====================================================
-- 8️⃣ ATTENDENCE POLICIES
-- =====================================================

create policy "attendence_select"
on attendence
for select
using (true);

create policy "attendence_insert"
on attendence
for insert
with check (true);

create policy "attendence_update"
on attendence
for update
using (true)
with check (true);

create policy "attendence_delete"
on attendence
for delete
using (true);


-- =====================================================
-- 9️⃣ DUMMY TABLE POLICIES
-- =====================================================

create policy "dummy_select"
on dummy_table
for select
using (true);

create policy "dummy_insert"
on dummy_table
for insert
with check (true);

create policy "dummy_update"
on dummy_table
for update
using (true)
with check (true);

create policy "dummy_delete"
on dummy_table
for delete
using (true);
```

---

# 🧠 SIMPLE SUPABASE RLS EXPLANATION (CLEAR VERSION)

## 🔐 Step 1: Enable RLS

```sql
alter table table_name enable row level security;
```

Without this → policies do nothing.

---

## 🔐 Step 2: Create Policy

```sql
create policy "policy_name"
on table_name
for operation
using (condition)
with check (condition);
```

---

## 🧩 What Each Part Means

### 🔹 `for select`

Controls reading rows.

### 🔹 `for insert`

Controls inserting rows.

### 🔹 `for update`

Controls modifying rows.

### 🔹 `for delete`

Controls deleting rows.

---

## 🔹 `using(condition)`

Controls which rows user can ACCESS.

Used in:

* SELECT
* UPDATE
* DELETE

Example:

```sql
using (true)
```

Means allow all rows.

---

## 🔹 `with check(condition)`

Controls what data user can WRITE.

Used in:

* INSERT
* UPDATE

Example:

```sql
with check (true)
```

Means allow writing anything.

---

# 🔥 Rule Table (Important)

| Operation | USING | WITH CHECK |
| --------- | ----- | ---------- |
| SELECT    | ✅     | ❌          |
| INSERT    | ❌     | ✅          |
| UPDATE    | ✅     | ✅          |
| DELETE    | ✅     | ❌          |

---

# 🧠 What is `auth.uid()` in Supabase?

```sql
auth.uid()
```

Returns currently logged-in user’s UUID from Supabase Auth.

You use it for secure policies like:

```sql
using (teacher_id = auth.uid())
```
