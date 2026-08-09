# MySQL phpMyAdmin — CRUD Operations, Drop & Import/Export

Once your database and tables are created, the next core skill is working with the actual **data inside those tables** — inserting, reading, updating, and deleting rows. These four operations are the heartbeat of every backend application, and every interaction a user has with your app ultimately triggers one of them.

---

## Table of Contents

1. [The CRUD Concept](#the-crud-concept)
2. [Insert Row — Adding New Data](#insert-row--adding-new-data)
   - [Single Row Insert via GUI](#single-row-insert-via-gui)
   - [Multiple Rows at Once](#multiple-rows-at-once)
   - [Insert via SQL Tab](#insert-via-sql-tab)
3. [Browse Row — Reading Data](#browse-row--reading-data)
   - [Basic Browse](#basic-browse)
   - [Sorting Rows](#sorting-rows)
   - [Filtering Rows](#filtering-rows)
   - [Browsing via SQL Tab](#browsing-via-sql-tab)
4. [Update Row — Editing Data](#update-row--editing-data)
   - [Update Single Row via GUI](#update-single-row-via-gui)
   - [Update via SQL Tab](#update-via-sql-tab)
5. [Delete Row — Removing Data](#delete-row--removing-data)
   - [Delete Single Row via GUI](#delete-single-row-via-gui)
   - [Delete Multiple Rows](#delete-multiple-rows)
   - [Truncate — Delete All Rows](#truncate--delete-all-rows)
   - [Delete via SQL Tab](#delete-via-sql-tab)
6. [Drop Table — Deleting a Table](#drop-table--deleting-a-table)
7. [Drop Database — Deleting a Database](#drop-database--deleting-a-database)
8. [Export — Backing Up Your Database](#export--backing-up-your-database)
   - [Export a Full Database](#export-a-full-database)
   - [Export a Single Table](#export-a-single-table)
   - [Export Options Explained](#export-options-explained)
   - [Export via Command Line](#export-via-command-line)
9. [Import — Restoring a Database](#import--restoring-a-database)
   - [Import into phpMyAdmin](#import-into-phpmyadmin)
   - [Import via Command Line](#import-via-command-line)
10. [Search — Finding Data](#search--finding-data)
11. [Operations Tab — Table-Level Actions](#operations-tab--table-level-actions)
12. [Quick Revision](#quick-revision)

---

## The CRUD Concept

Before diving in, understand the four fundamental database operations — everything you'll ever do with data falls into one of these:

| Letter | Operation | SQL Command | phpMyAdmin Tab | What It Does |
|---|---|---|---|---|
| **C** | Create | `INSERT` | Insert | Adds new rows |
| **R** | Read | `SELECT` | Browse | Retrieves rows |
| **U** | Update | `UPDATE` | Edit (✎ icon) | Modifies existing rows |
| **D** | Delete | `DELETE` | Delete (🗑 icon) | Removes rows |

```
Every user action in your app maps to CRUD:

User signs up         → INSERT into users
User logs in          → SELECT from users WHERE email = ?
User updates profile  → UPDATE users SET name = ? WHERE id = ?
User deletes account  → DELETE FROM users WHERE id = ?
                        (or UPDATE users SET deleted_at = NOW())

Admin views all orders → SELECT from orders
Admin ships an order   → UPDATE orders SET status = 'shipped' WHERE id = ?
Admin removes a product→ DELETE / soft-delete
```

---

## Insert Row — Adding New Data

### Single Row Insert via GUI

```
STEP 1: Click your table in the LEFT PANEL
        (e.g., click "users" under "myshop_db")

STEP 2: Click the "Insert" tab in the TOP MENU
  ┌──────────────────────────────────────────────────────────────────┐
  │  Browse | Structure | SQL | Search | [Insert] | Export | Import │
  │                                      ↑ Click this               │
  └──────────────────────────────────────────────────────────────────┘

STEP 3: The insert form appears — one row per column:

  ┌───────────────────────────────────────────────────────────────────┐
  │  Column        │ Type              │ Function     │ Value         │
  ├───────────────────────────────────────────────────────────────────┤
  │ id             │ int(10) unsigned  │ [None ▼]     │              │ ← leave empty!
  │ name           │ varchar(100)      │ [None ▼]     │ Phyo Min Paing│
  │ email          │ varchar(255)      │ [None ▼]     │ phyo@example.com│
  │ password       │ varchar(255)      │ [None ▼]     │ (paste hash) │
  │ role           │ enum              │ [None ▼]     │ user         │
  │ status         │ enum              │ [None ▼]     │ active       │
  │ created_at     │ timestamp         │ [None ▼]     │              │ ← leave empty!
  │ updated_at     │ timestamp         │ [None ▼]     │              │ ← leave empty!
  │ deleted_at     │ timestamp         │ [None ▼]     │              │ ← leave empty!
  └───────────────────────────────────────────────────────────────────┘

STEP 4: Fill in the values:
  id          → LEAVE EMPTY (AUTO_INCREMENT fills this automatically)
  name        → "Phyo Min Paing"
  email       → "phyo@example.com"
  password    → (paste your bcrypt hash here)
  role        → "user"
  status      → "active"
  created_at  → LEAVE EMPTY (DEFAULT CURRENT_TIMESTAMP fills this)
  updated_at  → LEAVE EMPTY (DEFAULT CURRENT_TIMESTAMP fills this)
  deleted_at  → LEAVE EMPTY (DEFAULT NULL — NULL means not deleted)

STEP 5: At the bottom, choose what to do after insert:
  ○ Go back to previous page
  ● Insert another new row   ← useful for inserting many rows
  ○ Edit the new row

STEP 6: Click "Go"

  ✅ 1 row inserted.
  MySQL auto-assigned: id = 1, created_at = 2026-06-28 14:30:00

phpMyAdmin shows the SQL it ran:
  INSERT INTO `users`
    (`name`, `email`, `password`, `role`, `status`)
  VALUES
    ('Phyo Min Paing', 'phyo@example.com', '$2y$10$...', 'user', 'active');
```

> 💡 **The "Function" column in phpMyAdmin Insert:** The dropdown next to each value lets you apply a MySQL function to the value. Common uses:
> - `NOW()` — insert the current timestamp as the value
> - `MD5()` — hash the value with MD5 (use PHP's password_hash instead)
> - `UUID()` — generate a UUID
> - `NULL` — explicitly insert NULL

---

### Multiple Rows at Once

```
The Insert tab shows TWO sets of fields by default.
This lets you insert 2 rows in one click.

STEP 1: Fill in the first row's values (top section)
STEP 2: Fill in the second row's values (bottom section)
STEP 3: Click "Go"

  ✅ 2 rows inserted.

To insert more than 2 rows at once:
  → Use the SQL tab (shown below) — much faster for bulk inserts
```

---

### Insert via SQL Tab

```
STEP 1: Select your database/table in the left panel
STEP 2: Click the "SQL" tab
STEP 3: Type your INSERT statement:
```

```sql
-- Insert ONE row
INSERT INTO users (name, email, password, role, status)
VALUES ('Phyo Min Paing', 'phyo@example.com', '$2y$10$hashed...', 'user', 'active');

-- Insert MULTIPLE rows in one statement (much faster!)
INSERT INTO users (name, email, password, role, status)
VALUES
  ('Phyo Min Paing', 'phyo@example.com',  '$2y$10$hash1...', 'user',  'active'),
  ('Alice',          'alice@example.com', '$2y$10$hash2...', 'admin', 'active'),
  ('Bob',            'bob@example.com',   '$2y$10$hash3...', 'user',  'active'),
  ('Charlie',        'charlie@example.com','$2y$10$hash4...', 'editor','inactive');

-- Insert with explicit timestamp
INSERT INTO users (name, email, password, created_at)
VALUES ('Dave', 'dave@example.com', '$2y$10$hash5...', '2026-01-15 09:00:00');

-- Insert only, but IGNORE errors (like duplicate email) instead of stopping
INSERT IGNORE INTO users (name, email, password)
VALUES ('Phyo', 'phyo@example.com', '$2y$10$hash1...');
-- If phyo@example.com already exists → skips silently (no error)

-- Insert OR update if the unique key already exists
INSERT INTO users (id, name, email, status)
VALUES (1, 'Phyo Updated', 'phyo@example.com', 'active')
ON DUPLICATE KEY UPDATE
  name   = VALUES(name),
  status = VALUES(status);
-- If id=1 already exists → updates name and status instead of failing
```

---

## Browse Row — Reading Data

### Basic Browse

```
STEP 1: Click a table in the LEFT PANEL (e.g., "users")
STEP 2: The "Browse" tab opens automatically (default view)
        OR click the "Browse" tab explicitly

  The results table:
  ┌──────────────────────────────────────────────────────────────────────┐
  │  ☐ │ ✎  🗑  │ id │ name             │ email               │ status │
  ├──────────────────────────────────────────────────────────────────────┤
  │  ☐ │ ✎  🗑  │  1 │ Phyo Min Paing   │ phyo@example.com    │ active │
  │  ☐ │ ✎  🗑  │  2 │ Alice            │ alice@example.com   │ active │
  │  ☐ │ ✎  🗑  │  3 │ Bob              │ bob@example.com     │ active │
  └──────────────────────────────────────────────────────────────────────┘

  Under the table:
  ← Previous  [1-3 / 3 total]  Next →    Rows per page: [25 ▼]

Key icons:
  ☐   → Checkbox to select this row for bulk actions
  ✎   → Edit this row (opens Update form)
  🗑  → Delete this row (with confirmation)
```

---

### Sorting Rows

```
Click any COLUMN HEADER to sort by that column:

  First click:  ↑ Ascending (A→Z, 0→9, oldest→newest)
  Second click: ↓ Descending (Z→A, 9→0, newest→oldest)

  Example: Click "created_at" header
  → Rows sorted by newest first
  → phpMyAdmin runs: SELECT * FROM `users` ORDER BY `created_at` DESC

  To sort by multiple columns:
  → Use the SQL tab (shown below)
```

---

### Filtering Rows

```
At the top of the Browse view there is a "Filter rows" input:
  ┌──────────────────────────────────────────────────────────────────────┐
  │  Filter rows:  [ Search this table... ]                              │
  └──────────────────────────────────────────────────────────────────────┘
  → This filters the CURRENT PAGE only — it's a visual filter, not a DB query.

For a REAL database-level search, use the "Search" tab (covered later).
```

---

### Browsing via SQL Tab

The SQL tab gives you full control over what you read.

```sql
-- Select ALL rows and ALL columns
SELECT * FROM users;

-- Select specific columns only (better practice — don't use SELECT *)
SELECT id, name, email, status, created_at FROM users;

-- Filter with WHERE
SELECT * FROM users WHERE status = 'active';
SELECT * FROM users WHERE role = 'admin';
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE email = 'phyo@example.com';

-- Multiple conditions
SELECT * FROM users WHERE status = 'active' AND role = 'user';
SELECT * FROM users WHERE role = 'admin' OR role = 'editor';

-- Pattern matching with LIKE
SELECT * FROM users WHERE name LIKE 'Phyo%';    -- names starting with "Phyo"
SELECT * FROM users WHERE name LIKE '%Min%';     -- names containing "Min"
SELECT * FROM users WHERE email LIKE '%@gmail.com'; -- Gmail addresses only

-- Sorting
SELECT * FROM users ORDER BY name ASC;           -- alphabetically A→Z
SELECT * FROM users ORDER BY created_at DESC;    -- newest first
SELECT * FROM users ORDER BY role ASC, name ASC; -- by role, then name

-- Limiting results
SELECT * FROM users LIMIT 10;             -- first 10 rows
SELECT * FROM users LIMIT 10 OFFSET 20;  -- rows 21-30 (pagination: page 3)
SELECT * FROM users ORDER BY created_at DESC LIMIT 5; -- 5 newest users

-- Count rows
SELECT COUNT(*) AS total_users FROM users;
SELECT COUNT(*) AS active_users FROM users WHERE status = 'active';

-- Exclude soft-deleted rows
SELECT * FROM users WHERE deleted_at IS NULL;

-- Only soft-deleted rows
SELECT * FROM users WHERE deleted_at IS NOT NULL;

-- NULL checks
SELECT * FROM users WHERE phone IS NULL;     -- users without phone
SELECT * FROM users WHERE phone IS NOT NULL; -- users with phone

-- Range queries
SELECT * FROM users WHERE created_at >= '2026-01-01';
SELECT * FROM users WHERE created_at BETWEEN '2026-01-01' AND '2026-06-30';

-- IN — match any value in a list
SELECT * FROM users WHERE role IN ('admin', 'editor');
SELECT * FROM users WHERE id IN (1, 5, 10, 42);

-- NOT IN
SELECT * FROM users WHERE status NOT IN ('banned', 'inactive');
```

> 💡 **Use the SQL tab for everything more complex than "show all rows." It's faster, more flexible, and you can copy the SQL into your PHP code directly.**

---

## Update Row — Editing Data

### Update Single Row via GUI

```
STEP 1: In Browse view, find the row you want to edit
STEP 2: Click the ✎ (Edit/pencil icon) at the start of that row

  The edit form opens — identical to the insert form but pre-filled:
  ┌────────────────────────────────────────────────────────────────┐
  │  Column      │ Type         │ Function   │ Value              │
  ├────────────────────────────────────────────────────────────────┤
  │ id           │ int unsigned │            │ 1  (read-only)     │ ← can't change PK
  │ name         │ varchar(100) │            │ [Phyo Min Paing  ] │ ← change this
  │ email        │ varchar(255) │            │ [phyo@example.com] │
  │ role         │ enum         │            │ [user ▼]           │ ← dropdown for ENUM
  │ status       │ enum         │            │ [active ▼]         │
  │ created_at   │ timestamp    │            │ 2026-01-15 09:30:00│ ← usually don't change
  │ updated_at   │ timestamp    │ [NOW() ▼] │ 2026-01-15 09:30:00│ ← set Function to NOW()
  └────────────────────────────────────────────────────────────────┘

STEP 3: Change the values you want to update
  Example: Change name from "Phyo Min Paing" to "Phyo Updated"

STEP 4: Click "Go"

  ✅ 1 row affected.

phpMyAdmin ran:
  UPDATE `users`
  SET `name` = 'Phyo Updated'
  WHERE `id` = 1;

(MySQL automatically updates `updated_at` because of ON UPDATE CURRENT_TIMESTAMP)
```

> 💡 **Tip for `updated_at`:** In the Function dropdown next to `updated_at`, select `NOW()` — this sets it to the current time. But if your table has `ON UPDATE CURRENT_TIMESTAMP` on the `updated_at` column, MySQL updates it automatically — you don't need to set it manually.

---

### Update via SQL Tab

```sql
-- Update a specific row (ALWAYS use WHERE!)
UPDATE users
SET name = 'Phyo Updated', email = 'phyo_new@example.com'
WHERE id = 1;

-- Update a single field
UPDATE users SET status = 'inactive' WHERE id = 5;

-- Update based on a condition (not just by ID)
UPDATE users SET status = 'active' WHERE email_verified_at IS NOT NULL;

-- Update multiple rows at once
UPDATE users SET status = 'inactive' WHERE role = 'user' AND created_at < '2020-01-01';

-- Increment a value
UPDATE products SET stock = stock + 10 WHERE id = 42;
UPDATE users    SET points = points + 100 WHERE id = 1;

-- Soft delete (marking as deleted without actually removing)
UPDATE users SET deleted_at = NOW() WHERE id = 5;

-- Restore a soft-deleted row
UPDATE users SET deleted_at = NULL WHERE id = 5;

-- Update with ENUM
UPDATE users SET role = 'admin' WHERE email = 'phyo@example.com';
```

> ⚠️ **Critical Warning — ALWAYS use WHERE with UPDATE!**

```sql
-- ❌ CATASTROPHIC MISTAKE — no WHERE clause
UPDATE users SET status = 'banned';
-- → THIS BANS EVERY SINGLE USER IN THE TABLE
-- → If you have 100,000 users, all 100,000 are now banned
-- → There is NO undo (unless you have a backup or use transactions)

-- ✅ CORRECT — target specific rows
UPDATE users SET status = 'banned' WHERE id = 5;
UPDATE users SET status = 'banned' WHERE email = 'spammer@example.com';
```

> 💡 **Safe UPDATE practice:** Before running an UPDATE, first run the equivalent SELECT to see which rows will be affected:
> ```sql
> -- First: CHECK what rows will be changed
> SELECT * FROM users WHERE role = 'user' AND created_at < '2020-01-01';
> -- See the results — do they look right? Yes?
>
> -- Then: RUN the update
> UPDATE users SET status = 'inactive' WHERE role = 'user' AND created_at < '2020-01-01';
> ```

---

## Delete Row — Removing Data

### Delete Single Row via GUI

```
STEP 1: In Browse view, find the row you want to delete
STEP 2: Click the 🗑 (Delete/trash icon) next to that row

STEP 3: A confirmation dialog appears:
  ┌─────────────────────────────────────────────────────────┐
  │                                                         │
  │  Do you really want to execute:                         │
  │  DELETE FROM `users` WHERE `id` = 3                    │
  │                                                         │
  │             [ OK ]           [ Cancel ]                 │
  └─────────────────────────────────────────────────────────┘

STEP 4: Click "OK"

  ✅ 1 row deleted.

Note: The AUTO_INCREMENT counter does NOT reset.
If you had id 1, 2, 3 and you delete id 3,
the next insert gets id 4 — not id 3 again.
This is intentional. (Covered in previous notes)
```

---

### Delete Multiple Rows

```
STEP 1: In Browse view, check the ☐ checkboxes at the left of the rows you want to delete

  ┌──────────────────────────────────────────────────────────────────────┐
  │  ☑ │ ✎  🗑  │  2 │ Alice   │ alice@example.com   │ active           │
  │  ☐ │ ✎  🗑  │  3 │ Bob     │ bob@example.com     │ active           │
  │  ☑ │ ✎  🗑  │  4 │ Charlie │ charlie@example.com │ inactive         │
  └──────────────────────────────────────────────────────────────────────┘
  → Row 2 and Row 4 are selected (rows 2 and 4 will be deleted)

STEP 2: At the BOTTOM of the table, in the dropdown that says "With selected:":
  [Delete ▼]    [Go]

STEP 3: Select "Delete" from the dropdown

STEP 4: Click "Go"

STEP 5: Confirm in the dialog

  ✅ 2 rows deleted.

phpMyAdmin ran:
  DELETE FROM `users` WHERE `id` IN (2, 4);
```

---

### Truncate — Delete All Rows

```
TRUNCATE removes ALL rows from a table instantly — but KEEPS the table structure.
Think of it as "empty the table completely."

STEP 1: Click your table in the left panel
STEP 2: Click the "Operations" tab
STEP 3: Find the "Delete data or table" section:
  ┌──────────────────────────────────────────────────────────────────────┐
  │  Delete data or table                                                │
  │  [ Empty the table (TRUNCATE) ]     [ Delete the table (DROP) ]    │
  └──────────────────────────────────────────────────────────────────────┘
STEP 4: Click "Empty the table (TRUNCATE)"
STEP 5: Confirm

  ✅ Table 'users' has been emptied.

TRUNCATE vs DELETE (no WHERE):
  DELETE FROM users;  → Deletes rows one by one, can be rolled back, slow for large tables
  TRUNCATE users;     → Drops and recreates the table, instant, resets AUTO_INCREMENT to 1
```

---

### Delete via SQL Tab

```sql
-- Delete a specific row
DELETE FROM users WHERE id = 5;

-- Delete rows matching a condition
DELETE FROM users WHERE status = 'banned';
DELETE FROM users WHERE created_at < '2020-01-01';
DELETE FROM users WHERE email LIKE '%@spammer.com';

-- Delete multiple specific rows
DELETE FROM users WHERE id IN (2, 5, 10);

-- SAFE DELETE — verify count first
SELECT COUNT(*) FROM users WHERE status = 'banned';  -- check: 3 rows
DELETE FROM users WHERE status = 'banned';            -- now delete them

-- Truncate (empty all rows, reset AUTO_INCREMENT)
TRUNCATE TABLE users;

-- Delete all rows without resetting AUTO_INCREMENT
DELETE FROM users;    -- slow, but doesn't reset counter
```

> ⚠️ **Warning — ALWAYS use WHERE with DELETE!**

```sql
-- ❌ CATASTROPHIC — no WHERE clause
DELETE FROM users;
-- → Deletes EVERY user. ALL of them. Gone.

-- ❌ Even more dangerous:
DELETE FROM orders;  -- Every order gone
DELETE FROM products; -- Every product gone

-- ✅ Always target specific rows:
DELETE FROM users WHERE id = 5;
DELETE FROM users WHERE status = 'banned' AND created_at < '2025-01-01';
```

> 💡 **Soft Delete vs Hard Delete — The Professional Approach:**

```sql
-- ❌ Hard delete (permanent, no recovery):
DELETE FROM users WHERE id = 5;

-- ✅ Soft delete (recoverable, auditable):
UPDATE users SET deleted_at = NOW() WHERE id = 5;

-- Then always filter in queries:
SELECT * FROM users WHERE deleted_at IS NULL;

-- If you need to restore:
UPDATE users SET deleted_at = NULL WHERE id = 5;

-- If you truly need to purge (after 90 days of soft deletion):
DELETE FROM users WHERE deleted_at < (NOW() - INTERVAL 90 DAY);
```

---

## Drop Table — Deleting a Table

- **DROP TABLE** permanently deletes the table AND all its data — structure and content, all gone.
- This is different from TRUNCATE (which removes data but keeps the table structure).

```
METHOD 1 — Via phpMyAdmin Operations Tab:

STEP 1: Click the table in the left panel
STEP 2: Click the "Operations" tab
STEP 3: Find the "Delete data or table" section:
  [ Empty the table (TRUNCATE) ]   [ Delete the table (DROP) ]
                                     ↑ Click this
STEP 4: phpMyAdmin asks for confirmation:
  "Do you really want to execute DROP TABLE `products`?"
STEP 5: Click "OK"

  ✅ Table 'products' has been dropped.


METHOD 2 — Via Structure tab:

STEP 1: Click your DATABASE (not a table) in the left panel
STEP 2: Click the "Structure" tab
STEP 3: A list of all tables appears:
  ┌──────────────────────────────────────────────────────────┐
  │  ☐  Table       │ Rows │ Type   │ Actions               │
  ├──────────────────────────────────────────────────────────┤
  │  ☐  users       │ 3    │ InnoDB │ Browse Struct Drop ... │
  │  ☐  products    │ 10   │ InnoDB │ Browse Struct [Drop]   │ ← Click Drop
  │  ☐  orders      │ 5    │ InnoDB │ Browse Struct Drop ... │
  └──────────────────────────────────────────────────────────┘
STEP 4: Click "Drop" next to the table you want to delete
STEP 5: Confirm

  ✅ Table 'products' has been dropped.


METHOD 3 — Via SQL Tab:

  DROP TABLE products;
  DROP TABLE IF EXISTS products;   -- No error if table doesn't exist
  DROP TABLE users, products, orders;  -- Drop multiple tables at once
```

> ⚠️ **Warning:** `DROP TABLE` is **permanent and instant**. There is no undo. All data in the table is lost forever. Always export/backup the table BEFORE dropping it.

---

## Drop Database — Deleting a Database

- **DROP DATABASE** deletes the entire database — all tables, all data, all indexes, everything inside it.
- The most destructive MySQL command. Use with extreme caution.

```
METHOD 1 — Via phpMyAdmin:

STEP 1: Click the DATABASE in the left panel (e.g., "myshop_db")
STEP 2: Click the "Operations" tab in the top menu
STEP 3: Find the "Remove database" section at the bottom:
  ┌────────────────────────────────────────────────────────────┐
  │  Remove database                                           │
  │  [ Drop the database (DROP) ]                              │
  └────────────────────────────────────────────────────────────┘
STEP 4: Click "Drop the database"
STEP 5: phpMyAdmin asks:
  "Do you really want to DROP the database 'myshop_db' ?"
STEP 6: Click "OK"

  ✅ Database 'myshop_db' has been dropped.
  (The database disappears from the left panel)


METHOD 2 — Via SQL Tab (at server level, not inside a database):

STEP 1: Click the phpMyAdmin HOME icon (top left)
        to ensure you're at server level, not inside a database
STEP 2: Click the "SQL" tab
STEP 3: Type:

  DROP DATABASE myshop_db;
  -- OR (no error if it doesn't exist):
  DROP DATABASE IF EXISTS myshop_db;


METHOD 3 — Command Line:

  mysqladmin -u root -p drop myshop_db
```

> ⚠️ **Extreme Warning:** `DROP DATABASE` deletes **everything** inside that database instantly. Every table, every row, every index, every view, every stored procedure — gone forever. This cannot be undone. **Always export/backup FIRST.**

---

## Export — Backing Up Your Database

Exporting creates a `.sql` file containing all the SQL commands to recreate your database — `CREATE TABLE` statements followed by `INSERT` statements for all the data. It's your backup.

---

### Export a Full Database

```
STEP 1: Click your DATABASE in the left panel (e.g., "myshop_db")
        Make sure you clicked the DATABASE, not a table inside it

STEP 2: Click the "Export" tab in the top menu
  ┌──────────────────────────────────────────────────────────────────┐
  │  Databases | SQL | Status | Export | Import | ...               │
  │                              ↑ Click this (at database level)    │
  └──────────────────────────────────────────────────────────────────┘

STEP 3: Choose Export Method:
  ● Quick  — Exports everything with default settings ← use this for most backups
  ○ Custom — Advanced options to customize exactly what to export

STEP 4: Format: SQL  ← This is the default and what you want for backups

STEP 5: Click "Go"

  → Your browser downloads: myshop_db.sql (or your database name)

  The downloaded file contains:
    -- MySQL dump ...
    -- Server version  8.0.xx
    CREATE DATABASE IF NOT EXISTS `myshop_db`;
    USE `myshop_db`;
    CREATE TABLE `users` (
      `id` int unsigned NOT NULL AUTO_INCREMENT,
      ...
    );
    INSERT INTO `users` VALUES (1,'Phyo Min Paing','phyo@example.com',...);
    INSERT INTO `users` VALUES (2,'Alice','alice@example.com',...);
    CREATE TABLE `products` (...);
    INSERT INTO `products` VALUES ...;
    ...
```

---

### Export a Single Table

```
STEP 1: Click the specific TABLE in the left panel (e.g., "users")
STEP 2: Click the "Export" tab
STEP 3: Choose Quick → SQL → Go

  → Downloads: users.sql
  → Contains only the CREATE TABLE and INSERTs for that one table
```

---

### Export Options Explained

When you choose "Custom" export method, you get many options:

```
TABLES TO EXPORT:
  ☑ users         → include users table
  ☑ products      → include products table
  ☑ orders        → include orders table
  ☐ sessions      → exclude sessions table (you might not want to backup temp data)

FORMAT:
  ● SQL    → Creates .sql file with CREATE TABLE + INSERT (standard backup format)
  ○ CSV    → Comma-separated values (for spreadsheets, data transfer)
  ○ Excel  → For opening in Microsoft Excel
  ○ JSON   → JSON format
  ○ XML    → XML format
  ○ PDF    → Visual table (not a proper backup — can't restore from PDF)

SQL-SPECIFIC OPTIONS (shown when SQL is selected):

Object creation options:
  ☑ Add DROP TABLE / VIEW / PROCEDURE / FUNCTION / EVENT / TRIGGER statement
    → Adds DROP TABLE IF EXISTS before each CREATE TABLE
    → Needed when restoring to a database that already has these tables
    → The DROP removes the old version before creating the new one

  ☑ Add IF NOT EXISTS
    → CREATE TABLE IF NOT EXISTS (no error if table exists)

Data creation options:
  ☑ Insert delayed rows
  ☑ Use a transaction for inserts   ← RECOMMENDED for large exports
    → Wraps all INSERTs in a transaction for faster, safer restore

  ☑ Dump binary columns in hexadecimal notation
  Max length of created query [0] → 0 = no limit per INSERT line

Output:
  ○ Show output in a text box    → display SQL in browser (for small exports)
  ● Save output to a file        ← ALWAYS use this for backups
  Compression:
    ● None       → plain .sql file
    ○ zipped     → .sql.zip (smaller file)
    ○ gzipped    → .sql.gz  (even smaller)
```

> 💡 **Backup Schedule for Professional Projects:**
> - **Local development:** Export manually before big changes.
> - **Production servers:** Automated daily backup with `mysqldump` via cron job, keep 30 days of backups, test restoring from backup regularly.

---

### Export via Command Line

For large databases (>50MB), phpMyAdmin can be slow or hit size limits. Use the command line instead.

```bash
# Export a single database
mysqldump -u root -p myshop_db > myshop_db_backup_2026.sql

# Export with timestamp in filename
mysqldump -u root -p myshop_db > myshop_db_$(date +%Y%m%d_%H%M%S).sql

# Export ALL databases
mysqldump -u root -p --all-databases > all_databases_backup.sql

# Export specific tables only
mysqldump -u root -p myshop_db users products > users_and_products.sql

# Export compressed (much smaller file)
mysqldump -u root -p myshop_db | gzip > myshop_db_backup.sql.gz

# Export with all recommended options for a proper backup
mysqldump \
  -u root -p \
  --single-transaction \     # Consistent backup without locking tables
  --routines \               # Include stored procedures and functions
  --triggers \               # Include triggers
  --events \                 # Include scheduled events
  --set-gtid-purged=OFF \    # Avoid issues when restoring to replica
  myshop_db > myshop_db_full_backup.sql

# Automated daily backup via cron (add to crontab with: crontab -e)
0 2 * * * mysqldump -u root -p'YourPassword' myshop_db > /backups/myshop_db_$(date +\%Y\%m\%d).sql
```

---

## Import — Restoring a Database

Importing loads a `.sql` file back into MySQL — recreating all tables and data from the backup.

---

### Import into phpMyAdmin

```
STEP 1: Make sure the TARGET DATABASE exists
  If you're restoring to a fresh MySQL install, create the database first:
  → Click "New" in left panel → name it "myshop_db" → Create

  OR via SQL tab (at server level):
  CREATE DATABASE IF NOT EXISTS myshop_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

STEP 2: Click the TARGET DATABASE in the left panel (e.g., "myshop_db")
        Very important! Click the DATABASE, not a table

STEP 3: Click the "Import" tab in the top menu

STEP 4: The import form appears:
  ┌───────────────────────────────────────────────────────────────────┐
  │  File to Import:                                                  │
  │  Location of the text file: [ Choose File ] myshop_db.sql       │
  │  Character set of the file: [ utf-8 ▼ ]                         │
  │                                                                   │
  │  Partial Import:                                                  │
  │  ☐ Allow interrupt of import if script detects it may take long  │
  │  Number of records (queries) to skip from start: [ 0 ]           │
  │                                                                   │
  │  Other options:                                                   │
  │  ☐ Do not use AUTO_VALUE_ON_ZERO                                  │
  │  ☑ Enable foreign key checks                                      │
  │                                                                   │
  │  Format: [ SQL ▼ ]                                               │
  │                                                                   │
  │                                          [ Go ]                   │
  └───────────────────────────────────────────────────────────────────┘

STEP 5: Click "Choose File" → select your .sql backup file

STEP 6: Click "Go"

  ✅ Import has been successfully finished, 42 queries executed.
  (The number of queries = CREATE TABLE statements + INSERT statements)
```

> ⚠️ **phpMyAdmin Import File Size Limit:**
> phpMyAdmin's default maximum upload is **2MB**. For larger databases, you get an error like "File exceeds the maximum allowed size."

```bash
# Fix phpMyAdmin upload size limit:
# Edit php.ini (the one Apache/Nginx uses for phpMyAdmin):
sudo nano /etc/php/8.3/apache2/php.ini

# Change these values:
upload_max_filesize = 256M
post_max_size = 256M
memory_limit = 512M
max_execution_time = 600

# Restart Apache:
sudo systemctl restart apache2

# OR: For large files, use command line import instead (no size limit):
mysql -u root -p myshop_db < myshop_db_backup.sql
```

---

### Import via Command Line

For large databases or automation, the command line is the right tool.

```bash
# Import into an existing database
mysql -u root -p myshop_db < myshop_db_backup.sql

# Import into a database (creating it first if needed)
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS myshop_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
mysql -u root -p myshop_db < myshop_db_backup.sql

# Import a compressed backup
gunzip < myshop_db_backup.sql.gz | mysql -u root -p myshop_db
# OR:
zcat myshop_db_backup.sql.gz | mysql -u root -p myshop_db

# Import with progress (shows output as it runs)
mysql -u root -p myshop_db < myshop_db_backup.sql --verbose

# Import multiple SQL files
cat file1.sql file2.sql file3.sql | mysql -u root -p myshop_db

# Show progress for large imports
pv myshop_db_backup.sql | mysql -u root -p myshop_db
# (requires: sudo apt install pv)
```

---

## Search — Finding Data

The Search tab in phpMyAdmin lets you query data without writing SQL.

```
STEP 1: Click a table in the left panel
STEP 2: Click the "Search" tab

  ┌─────────────────────────────────────────────────────────────────────┐
  │  Search options:                                                    │
  │                                                                     │
  │  Word(s) to search for (wildcard: "%"):  [phyo                 ]  │
  │                                                                     │
  │  Find rows where:                                                   │
  │  ┌─────────────┬────────────────┬─────────────────────────────┐    │
  │  │ Column      │ Operator       │ Value                        │    │
  │  ├─────────────┼────────────────┼─────────────────────────────┤    │
  │  │ name    ▼  │ LIKE %...%  ▼ │ [Phyo                     ] │    │
  │  │ email   ▼  │ =           ▼ │ [phyo@example.com         ] │    │
  │  │ status  ▼  │ =           ▼ │ [active                   ] │    │
  │  └─────────────┴────────────────┴─────────────────────────────┘    │
  │                                                                     │
  │  Add/remove conditions: [+] [-]                                     │
  │                                                                     │
  │  Select columns to display: ☑ id ☑ name ☑ email ☑ status ...      │
  │                                                                     │
  │  Order by: [ created_at ▼ ] [ DESC ▼ ]                             │
  │                                                                     │
  │  Rows per page: [25]                                                │
  │                                                                     │
  │                                          [ Go ]                     │
  └─────────────────────────────────────────────────────────────────────┘

Available operators:
  = (equals)
  != (not equals)
  < (less than)
  > (greater than)
  <= (less than or equal)
  >= (greater than or equal)
  LIKE (pattern match with %)
  NOT LIKE
  IN (comma-separated list)
  NOT IN
  IS NULL
  IS NOT NULL
  BETWEEN ... AND ... (two values needed)
  REGEXP (regular expression)
```

---

## Operations Tab — Table-Level Actions

The Operations tab contains many useful maintenance actions.

```
STEP 1: Click a table in the left panel
STEP 2: Click the "Operations" tab

  ┌─────────────────────────────────────────────────────────────────────┐
  │  Table options:                                                     │
  │  Table name: [users       ]    → RENAME the table                  │
  │  Table comments: [         ]   → Add description                   │
  │  Storage Engine: [InnoDB ▼]    → Change storage engine             │
  │  Collation: [utf8mb4_unicode_ci ▼]  → Change character set        │
  │                       [ Go ]                                        │
  ├─────────────────────────────────────────────────────────────────────┤
  │  Copy table to (database.table):                                    │
  │  [myshop_db ▼] . [users_backup  ]   → Make a copy of the table    │
  │  ○ Structure only                                                   │
  │  ● Structure and data    [ Go ]                                     │
  ├─────────────────────────────────────────────────────────────────────┤
  │  Move table to (database.table):                                    │
  │  [other_db ▼] . [users         ]   → Move table to another DB     │
  │                  [ Go ]                                             │
  ├─────────────────────────────────────────────────────────────────────┤
  │  Table maintenance:                                                 │
  │  [Analyze table] [Check table] [Checksum table]                    │
  │  [Optimize table] [Repair table] [Flush the table]                 │
  ├─────────────────────────────────────────────────────────────────────┤
  │  Delete data or table:                                              │
  │  [Empty the table (TRUNCATE)]   [Delete the table (DROP)]          │
  └─────────────────────────────────────────────────────────────────────┘
```

### Key Operations Explained

```
RENAME TABLE:
  Change the table name: users → app_users
  SQL equivalent: RENAME TABLE users TO app_users;

COPY TABLE:
  Create "users_backup" with same structure + data
  Useful before making risky changes to a table
  SQL equivalent: CREATE TABLE users_backup SELECT * FROM users;

OPTIMIZE TABLE:
  Defragments the table data — reclaims space after many DELETEs
  Run this if a table has had lots of inserts and deletes
  SQL equivalent: OPTIMIZE TABLE users;

ANALYZE TABLE:
  Updates statistics MySQL uses for query optimization
  Run this if queries have become slow after large data changes
  SQL equivalent: ANALYZE TABLE users;

REPAIR TABLE:
  Repairs a corrupted MyISAM table (rare in InnoDB)
  SQL equivalent: REPAIR TABLE users;
```

---

## Quick Revision

- **INSERT** adds new rows. Leave `id`, `created_at`, `updated_at` empty — MySQL fills them automatically. Use the SQL tab for bulk inserts.
- **BROWSE** shows all rows in a table. Click column headers to sort. Use the SQL tab (`SELECT`) for filtered, sorted, paginated queries.
- **UPDATE** modifies existing rows. In the GUI, click the ✎ icon → change values → Go. In SQL, **ALWAYS use WHERE** — `UPDATE without WHERE` changes every single row.
- **DELETE** removes rows permanently. In the GUI, click the 🗑 icon → confirm. In SQL, **ALWAYS use WHERE** — `DELETE without WHERE` removes every row. Prefer soft delete: `UPDATE table SET deleted_at = NOW() WHERE id = ?`.
- **TRUNCATE** removes ALL rows instantly and resets AUTO_INCREMENT to 1. Faster than DELETE for clearing a whole table.
- **DROP TABLE** deletes the table AND all its data permanently. No undo. Always backup first.
- **DROP DATABASE** deletes the entire database — all tables and all data. Instant and irreversible. Always backup first.
- **EXPORT** creates a `.sql` backup file. Database level → Export tab → Quick → SQL → Go → downloads `.sql` file. For large databases use `mysqldump` on the command line.
- **IMPORT** restores from a `.sql` file. Create the database first → click it in left panel → Import tab → Choose File → Go. phpMyAdmin has a 2MB limit — use `mysql -u root -p dbname < backup.sql` for larger files.
- **SEARCH** tab — visual query builder. Choose column, operator, value, sort order without writing SQL.
- **OPERATIONS** tab — rename table, copy table, move to another database, optimize, analyze, truncate, drop.
- **Golden rule for UPDATE and DELETE:** Always run the equivalent `SELECT` first to see which rows will be affected before running the destructive command.
- **Soft delete pattern:** Never `DELETE` important records. Use `UPDATE table SET deleted_at = NOW()` and filter with `WHERE deleted_at IS NULL` in all queries.