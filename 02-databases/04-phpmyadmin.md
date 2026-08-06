# Using MySQL with phpMyAdmin — GUI Database Management

**phpMyAdmin** is a free, web-based graphical interface for managing MySQL databases directly from your browser — no command line required. It is the most widely used MySQL GUI tool in the world, pre-installed on virtually every shared hosting platform, and bundled with XAMPP, MAMP, and WAMP for local development.

---

## Table of Contents

1. [What is phpMyAdmin?](#what-is-phpmyadmin)
2. [Installing phpMyAdmin](#installing-phpmyadmin)
   - [With XAMPP (Windows/Mac)](#with-xampp-windowsmac)
   - [With MAMP (Mac)](#with-mamp-mac)
   - [Ubuntu / Debian (Manual)](#ubuntu--debian-manual)
   - [Laragon (Windows)](#laragon-windows)
3. [Logging In to phpMyAdmin](#logging-in-to-phpmyadmin)
4. [Understanding the phpMyAdmin Interface](#understanding-the-phpmyadmin-interface)
5. [Creating a New Database](#creating-a-new-database)
   - [Method 1 — Using the GUI](#method-1--using-the-gui)
   - [Method 2 — Using the SQL Tab](#method-2--using-the-sql-tab)
6. [Creating Tables](#creating-tables)
7. [Inserting Data (Rows)](#inserting-data-rows)
8. [Browsing & Searching Data](#browsing--searching-data)
9. [Editing and Deleting Data](#editing-and-deleting-data)
10. [Running SQL Queries](#running-sql-queries)
11. [Importing and Exporting Databases](#importing-and-exporting-databases)
12. [Managing Users in phpMyAdmin](#managing-users-in-phpmyadmin)
13. [phpMyAdmin Keyboard Shortcuts & Tips](#phpmyadmin-keyboard-shortcuts--tips)
14. [Quick Revision](#quick-revision)

---

## What is phpMyAdmin?

- **phpMyAdmin** is a free, open-source web application written in PHP that provides a graphical user interface (GUI) for managing MySQL and MariaDB databases.
- Instead of typing SQL commands in a terminal, you click buttons, fill in forms, and drag-and-drop — and phpMyAdmin writes and executes the SQL for you behind the scenes.
- You can always see the SQL that phpMyAdmin generated, which is a great way to learn SQL by watching what the GUI does.

```
Without phpMyAdmin (command line):
  mysql -u root -p
  CREATE DATABASE myshop CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
  CREATE TABLE products (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255));
  INSERT INTO products (name) VALUES ('T-Shirt');
  SELECT * FROM products;

With phpMyAdmin (GUI):
  → Open browser → http://localhost/phpmyadmin
  → Click "New" → Type "myshop" → Click "Create"
  → Click "Create table" → Fill in fields → Click "Save"
  → Click "Insert" → Fill in values → Click "Go"
  → Click "Browse" → See all rows
  (Same result, zero typing of SQL)
```

> 💡 **Why learn phpMyAdmin?** Every shared hosting account uses cPanel which includes phpMyAdmin. It's the tool your clients, your team, and your hosting provider all use. Even if you prefer the command line later, you need to know phpMyAdmin for professional PHP development.

---

## Installing phpMyAdmin

---

### With XAMPP (Windows/Mac)

phpMyAdmin is included by default in XAMPP — nothing extra to install.

```
1. Download XAMPP: https://www.apachefriends.org/
2. Install and open the XAMPP Control Panel
3. Start "Apache" and "MySQL" by clicking their Start buttons
4. Open your browser and go to:
   http://localhost/phpmyadmin

That's it. phpMyAdmin is running.

Default login:
  Username: root
  Password: (empty — just press Enter / leave blank)
```

```
XAMPP Control Panel:
┌──────────────────────────────────────────────────────┐
│  Module      │ PID(s)  │ Port(s) │ Actions           │
├──────────────┼─────────┼─────────┼───────────────────┤
│  Apache      │ 8432    │ 80, 443 │ [Stop][Admin][...] │
│  MySQL       │ 9121    │ 3306    │ [Stop][Admin][...] │ ← "Admin" opens phpMyAdmin
└──────────────┴─────────┴─────────┴───────────────────┘
```

> 💡 **Tip:** In XAMPP, clicking the "Admin" button next to MySQL opens phpMyAdmin directly in your browser.

---

### With MAMP (Mac)

```
1. Download MAMP: https://www.mamp.info/
2. Install and open MAMP
3. Click "Start Servers"
4. Open browser and go to:
   http://localhost:8888/phpmyadmin
   (MAMP uses port 8888, not 80)

Default login:
  Username: root
  Password: root
```

---

### Ubuntu / Debian (Manual)

```bash
# Step 1: Install phpMyAdmin
sudo apt update
sudo apt install phpmyadmin

# During installation you will be asked:
# → "Which web server?" → Select apache2 (press Space to select, Enter to confirm)
# → "Configure database with dbconfig-common?" → YES
# → "MySQL application password for phpMyAdmin:" → Enter a password

# Step 2: Enable the phpMyAdmin config in Apache
sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
# OR it may already be at:
sudo phpenmod mbstring
sudo systemctl restart apache2

# Step 3: Access in browser
# http://localhost/phpmyadmin
# OR if server:
# http://your-server-ip/phpmyadmin

# Step 4: Login
# Username: root (or any MySQL user)
# Password: your MySQL root password
```

> ⚠️ **Production Security Warning:** If phpMyAdmin is on a production server, you MUST restrict access to it. Exposing phpMyAdmin to the public internet is a major security risk. Options:
> - Password-protect the phpMyAdmin directory with `.htaccess`
> - Allow access only from specific IPs using Apache/Nginx config
> - Move it to a non-obvious URL path like `/secret-admin-panel/`
> - Use a VPN to access it

```apache
# Restrict phpMyAdmin to your IP only (/etc/apache2/conf-available/phpmyadmin.conf)
<Directory /usr/share/phpmyadmin>
    Order Deny,Allow
    Deny from All
    Allow from 127.0.0.1     # localhost only
    Allow from 203.0.113.50  # your office/home IP
</Directory>
```

---

### Laragon (Windows)

```
Laragon is a fast, modern local development environment for Windows.
phpMyAdmin is included by default.

1. Download: https://laragon.org/
2. Install and start Laragon
3. Click "Database" button in Laragon → phpMyAdmin opens
   OR go to: http://localhost/phpmyadmin

Default login:
  Username: root
  Password: (empty)
```

---

## Logging In to phpMyAdmin

```
Open your browser and go to:
  Local XAMPP:  http://localhost/phpmyadmin
  Local MAMP:   http://localhost:8888/phpmyadmin
  Ubuntu:       http://localhost/phpmyadmin
  Remote:       http://your-server-ip/phpmyadmin

Login screen:
  ┌────────────────────────────────────────┐
  │         Welcome to phpMyAdmin          │
  │                                        │
  │  Username: [root            ]          │
  │  Password: [••••••••••••••••]          │
  │                                        │
  │              [ Go ]                    │
  └────────────────────────────────────────┘

Common login credentials:
  XAMPP:   root / (no password)
  MAMP:    root / root
  Laragon: root / (no password)
  Ubuntu:  root / (your mysql root password)
  Hosting: (given by your hosting provider in cPanel)
```

> ⚠️ **Warning:** If you see "Access denied for user 'root'@'localhost'", your root user may use `auth_socket` instead of password authentication. Fix it with:
> ```bash
> sudo mysql
> ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'your_password';
> FLUSH PRIVILEGES;
> ```

---

## Understanding the phpMyAdmin Interface

Once logged in, you'll see the main interface. Let's understand every part of it.

```
phpMyAdmin Main Interface Layout:
┌─────────────────────────────────────────────────────────────────┐
│  [phpMyAdmin Logo]  [Home] [SQL] [Status] [User Accounts]...   │ ← Top Menu Bar
├───────────────────┬─────────────────────────────────────────────┤
│                   │                                             │
│  Left Panel       │  Right Panel (Main Content Area)           │
│  (Navigation)     │                                             │
│                   │  General Settings / Database Info          │
│  Recent           │                                             │
│  ┌─────────────┐  │                                             │
│  │ information │  │                                             │
│  │ _schema     │  │                                             │
│  │ mysql       │  │                                             │
│  │ performance │  │                                             │
│  │ _schema     │  │                                             │
│  │ sys         │  │                                             │
│  └─────────────┘  │                                             │
│                   │                                             │
│  [ New ]          │                                             │
│  ─────────────    │                                             │
│  + myapp_db       │                                             │
│    + users        │                                             │
│    + products     │                                             │
│    + orders       │                                             │
│                   │                                             │
└───────────────────┴─────────────────────────────────────────────┘
```

### Key Areas Explained

```
Left Panel (Navigation Tree):
  Shows all databases on this MySQL server.
  Click a database name → expands to show its tables.
  Click a table name → shows the table's data and options.
  "New" button at top → creates a new database.

Top Menu Bar (changes based on what you've selected):
  When on SERVER level:  Databases | SQL | Status | User accounts | Export | Import
  When on DATABASE level: Structure | SQL | Search | Query | Export | Import | Operations
  When on TABLE level:   Browse | Structure | SQL | Search | Insert | Export | Import

Right Panel (Main Content):
  This is where everything happens — tables, query results,
  settings, import/export dialogs, user management.
```

---

## Creating a New Database

This is the most fundamental operation — creating a database for your project.

---

### Method 1 — Using the GUI

```
Step-by-step with screenshots described:

STEP 1: Look at the LEFT PANEL
  At the very top of the left panel, you'll see a "New" button.
  Click it.

  ┌─────────────────┐
  │  [ New ]        │ ← Click this
  │  ─────────────  │
  │  information... │
  │  mysql          │
  └─────────────────┘

STEP 2: The "Create database" form appears in the RIGHT PANEL

  ┌─────────────────────────────────────────────────────────┐
  │  Create database                                        │
  │                                                         │
  │  Database name: [myshop_db        ]                     │
  │                                                         │
  │  Collation:     [utf8mb4_unicode_ci ▼]                  │
  │                                                         │
  │                              [ Create ]                 │
  └─────────────────────────────────────────────────────────┘

STEP 3: Fill in the details:
  Database name: myshop_db
    (Use lowercase letters, numbers, underscores. No spaces!)
    Good names:  myshop_db, blog_app, user_management
    Bad names:   "My Shop DB", my-shop, 123database

  Collation: utf8mb4_unicode_ci
    (Search "utf8mb4_unicode" in the dropdown)
    Why: Supports full Unicode including Burmese, emoji, Chinese
    If you leave it as "utf8mb4_general_ci" → also fine for most apps
    NEVER choose just "utf8" → it's broken and can't store emoji!

STEP 4: Click the "Create" button

STEP 5: Success!
  ✅ "Database myshop_db has been created"
  The new database appears in the left panel navigation tree.
  You're automatically taken inside the new database.
```

> ⚠️ **Database Naming Rules:**
> - Use only lowercase letters, numbers, and underscores
> - No spaces (use `_` instead: `my_shop` not `my shop`)
> - No hyphens (use `_` instead: `my_shop` not `my-shop`)
> - No reserved words as database name: `database`, `select`, `table`
> - Keep it descriptive: `myshop_db`, `blog_production`, `user_auth`

---

### Method 2 — Using the SQL Tab

```
You can also create a database by typing SQL directly in phpMyAdmin.

STEP 1: Click the "SQL" tab in the TOP MENU BAR (at server level)
  ┌──────────────────────────────────────────────────────────┐
  │  Databases | SQL | Status | User accounts | Export | ... │
  │             ↑                                            │
  │         Click this                                       │
  └──────────────────────────────────────────────────────────┘

STEP 2: A SQL editor appears:
  ┌─────────────────────────────────────────────────────────┐
  │  Run SQL query/queries on the MySQL server:             │
  │  ┌──────────────────────────────────────────────────┐  │
  │  │ CREATE DATABASE myshop_db                        │  │
  │  │   CHARACTER SET utf8mb4                          │  │
  │  │   COLLATE utf8mb4_unicode_ci;                    │  │
  │  └──────────────────────────────────────────────────┘  │
  │                                           [ Go ]        │
  └─────────────────────────────────────────────────────────┘

STEP 3: Click "Go"
  ✅ "Your SQL query has been executed successfully"
```

```sql
-- Full SQL for creating a production-ready database:
CREATE DATABASE IF NOT EXISTS myshop_db
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
```

> 💡 **Pro tip:** The SQL tab is incredibly useful for learning — paste in any SQL query, run it, and immediately see the results. It's also the fastest way to do bulk operations that would take many GUI clicks.

---

## Creating Tables

After creating a database, you need tables to store data.

```
STEP 1: Click on your database in the left panel
  → "myshop_db" appears selected

STEP 2: In the right panel you see:
  ┌────────────────────────────────────────────────────┐
  │  Database myshop_db has no tables                  │
  │                                                    │
  │  Create table:                                     │
  │  Name: [users        ]                             │
  │  Number of columns: [4  ]                          │
  │                            [ Go ]                  │
  └────────────────────────────────────────────────────┘

STEP 3: Enter table name "users", set columns to 4, click "Go"

STEP 4: The table definition form appears (one row per column):

  ┌────────┬──────────────┬──────────┬─────┬─────────┬───────────────────┐
  │ Name   │ Type         │ Length   │ Null│ Default │ Extra             │
  ├────────┼──────────────┼──────────┼─────┼─────────┼───────────────────┤
  │ id     │ INT          │          │ No  │ None    │ AUTO_INCREMENT     │
  │ name   │ VARCHAR      │ 100      │ No  │ None    │                   │
  │ email  │ VARCHAR      │ 255      │ No  │ None    │                   │
  │ created│ TIMESTAMP    │          │ No  │ CURRENT │                   │
  │ _at    │              │          │     │ _TIMEST.│                   │
  └────────┴──────────────┴──────────┴─────┴─────────┴───────────────────┘

  For the "id" column:
    → Set "Index" to PRIMARY
    → Check "A_I" (AUTO_INCREMENT) checkbox

STEP 5: Scroll down → Click "Save"

  ✅ Table 'users' has been created

phpMyAdmin generates and runs this SQL behind the scenes:
  CREATE TABLE `users` (
    `id`         INT           NOT NULL AUTO_INCREMENT,
    `name`       VARCHAR(100)  NOT NULL,
    `email`      VARCHAR(255)  NOT NULL,
    `created_at` TIMESTAMP     NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Common Column Types in phpMyAdmin

| Type | phpMyAdmin Name | Use For | Example |
|---|---|---|---|
| `INT` | INT | Whole numbers, IDs | `1`, `42`, `-5` |
| `BIGINT` | BIGINT | Very large numbers | User IDs at scale |
| `VARCHAR` | VARCHAR(n) | Short text (max n chars) | Names, emails, titles |
| `TEXT` | TEXT | Long text (up to 65,535 chars) | Blog posts, descriptions |
| `LONGTEXT` | LONGTEXT | Very long text | Article content |
| `DECIMAL(10,2)` | DECIMAL | Exact decimals (money) | `99.99` |
| `FLOAT` | FLOAT | Approximate decimals | Scientific data |
| `BOOLEAN` / `TINYINT(1)` | TINYINT(1) | True/False flags | `0` or `1` |
| `DATE` | DATE | Date only | `2026-06-28` |
| `DATETIME` | DATETIME | Date + time | `2026-06-28 14:30:00` |
| `TIMESTAMP` | TIMESTAMP | Auto-updated time | `CURRENT_TIMESTAMP` |
| `ENUM` | ENUM | Fixed set of values | `'active','inactive'` |
| `JSON` | JSON | JSON data (MySQL 5.7+) | `{"key": "value"}` |

---

## Inserting Data (Rows)

```
STEP 1: Click on a table in the left panel (e.g., "users")

STEP 2: Click the "Insert" tab in the top menu

  ┌────────────────────────────────────────────────────────────┐
  │  Browse | Structure | SQL | Search | [Insert] | Export ... │
  │                                      ↑ Click this          │
  └────────────────────────────────────────────────────────────┘

STEP 3: Fill in the form (one row = one record):

  ┌──────────────┬────────────────────────────────┐
  │ Column       │ Value                           │
  ├──────────────┼────────────────────────────────┤
  │ id           │ (leave empty — auto-generated) │
  │ name         │ Phyo Min Paing                 │
  │ email        │ phyo@example.com               │
  │ created_at   │ (leave empty — auto-generated) │
  └──────────────┴────────────────────────────────┘

STEP 4: Click "Go" at the bottom

  ✅ 1 row inserted.

  phpMyAdmin ran:
  INSERT INTO `users` (`name`, `email`) VALUES ('Phyo Min Paing', 'phyo@example.com');

STEP 5: Insert another row — phpMyAdmin shows a second set of fields below
  You can insert 2 rows at once by filling in both sections.

STEP 6: Click "Go" again
  ✅ Another row inserted.
```

---

## Browsing & Searching Data

```
VIEW ALL ROWS:

STEP 1: Click your table in the left panel (e.g., "users")
STEP 2: Click the "Browse" tab (it's usually the default view)

  Result table appears:
  ┌──────────────────────────────────────────────────────────────┐
  │  ✎ Edit  🗑 Delete  │ id │ name            │ email           │
  ├──────────────────────────────────────────────────────────────┤
  │  ✎ 🗑                │  1 │ Phyo Min Paing  │ phyo@ex...     │
  │  ✎ 🗑                │  2 │ Alice           │ alice@ex...    │
  │  ✎ 🗑                │  3 │ Bob             │ bob@ex...      │
  └──────────────────────────────────────────────────────────────┘

  Controls at the bottom:
  ← Previous | 1-3 / 3 rows | Next →   Rows per page: [25 ▼]

────────────────────────────────────────────────────────────────

SEARCH DATA:

STEP 1: Click the "Search" tab
  ┌─────────────────────────────────────────────────────────┐
  │  Search by value     OR     Select by criteria          │
  │                                                         │
  │  Column: email                                          │
  │  Operator: [ = ▼]   Value: [phyo@example.com]          │
  │                                                         │
  │                                      [ Go ]             │
  └─────────────────────────────────────────────────────────┘

  Available operators: =, !=, <, >, <=, >=, LIKE, NOT LIKE, IN, IS NULL, IS NOT NULL

SEARCH WITH LIKE (pattern matching):
  Column: name
  Operator: LIKE
  Value: %Phyo%       ← finds any name containing "Phyo"
                         % = wildcard (any characters)
```

---

## Editing and Deleting Data

```
EDITING A ROW:

STEP 1: In Browse view, click the ✎ (Edit/pencil icon) for the row you want

STEP 2: A form appears with all current values pre-filled:
  ┌──────────────┬────────────────────────────────┐
  │ Column       │ Value                           │
  ├──────────────┼────────────────────────────────┤
  │ id           │ 1 (cannot edit primary key)     │
  │ name         │ [Phyo Min Paing        ]        │ ← Change this
  │ email        │ [phyo@example.com      ]        │ ← Change this
  └──────────────┴────────────────────────────────┘

STEP 3: Modify the values and click "Go"
  ✅ 1 row affected.

  phpMyAdmin ran:
  UPDATE `users` SET `name`='Phyo Updated', `email`='new@email.com' WHERE `id`=1;

────────────────────────────────────────────────────────────────

DELETING A ROW:

STEP 1: In Browse view, click the 🗑 (Delete/trash icon) for the row

STEP 2: A confirmation dialog appears:
  ┌──────────────────────────────────────────┐
  │  Do you really want to execute:          │
  │  DELETE FROM `users` WHERE `id` = 1     │
  │                                          │
  │         [OK]          [Cancel]           │
  └──────────────────────────────────────────┘

STEP 3: Click "OK"
  ✅ 1 row deleted.

────────────────────────────────────────────────────────────────

DELETING MULTIPLE ROWS:

STEP 1: Check the checkbox(es) at the start of each row you want to delete
STEP 2: At the bottom of the table, choose "Delete" from the dropdown
STEP 3: Click "Go" → Confirm

────────────────────────────────────────────────────────────────

DELETING ALL ROWS (TRUNCATE):

Click the table in the left panel
→ "Operations" tab
→ "Delete all rows (TRUNCATE)" button
⚠️ This deletes ALL data but keeps the table structure.
```

> ⚠️ **Warning:** There is NO undo for DELETE in phpMyAdmin (and MySQL in general, unless you use transactions). Always take a backup before mass-deleting data.

---

## Running SQL Queries

The SQL tab is the most powerful feature of phpMyAdmin — you can run any SQL command directly.

```
STEP 1: Select a database in the left panel (click on "myshop_db")

STEP 2: Click the "SQL" tab in the top menu

STEP 3: Type your SQL in the editor box:

  ┌───────────────────────────────────────────────────────────┐
  │  Run SQL query/queries on database myshop_db:             │
  │  ┌───────────────────────────────────────────────────┐   │
  │  │ SELECT u.name, u.email,                           │   │
  │  │        COUNT(o.id) AS total_orders,               │   │
  │  │        SUM(o.total) AS lifetime_value             │   │
  │  │ FROM users u                                      │   │
  │  │ LEFT JOIN orders o ON u.id = o.user_id            │   │
  │  │ GROUP BY u.id                                     │   │
  │  │ ORDER BY lifetime_value DESC;                     │   │
  │  └───────────────────────────────────────────────────┘   │
  │                                                           │
  │  ☑ Retain query box (keep the SQL visible after running) │
  │  [ Format ]   [ Clear ]              [ Go ]               │
  └───────────────────────────────────────────────────────────┘

STEP 4: Click "Go"

  Results appear below:
  ┌──────────────────┬──────────────────────┬──────────────┬─────────────────┐
  │ name             │ email                │ total_orders │ lifetime_value  │
  ├──────────────────┼──────────────────────┼──────────────┼─────────────────┤
  │ Phyo Min Paing   │ phyo@example.com     │ 5            │ 499.95          │
  │ Alice            │ alice@example.com    │ 3            │ 299.97          │
  │ Bob              │ bob@example.com      │ 1            │ 49.99           │
  └──────────────────┴──────────────────────┴──────────────┴─────────────────┘

  3 rows returned. (Query took 0.0013 sec)
```

### Quick SQL Examples to Try in phpMyAdmin

```sql
-- Show all tables in current database
SHOW TABLES;

-- Show structure of a table
DESCRIBE users;
-- OR
SHOW COLUMNS FROM users;

-- Count all rows in a table
SELECT COUNT(*) FROM users;

-- Get 10 most recent users
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;

-- Find users from a specific city (if you have a city column)
SELECT * FROM users WHERE city = 'Yangon';

-- Update all inactive users to active
UPDATE users SET status = 'active' WHERE status = 'inactive';

-- Show all indexes on a table
SHOW INDEX FROM users;

-- Show create statement of a table
SHOW CREATE TABLE users;

-- Check database size (in MB)
SELECT
  table_schema AS 'Database',
  ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.tables
WHERE table_schema = 'myshop_db'
GROUP BY table_schema;
```

> 💡 **Great learning habit:** Whenever you do something in the phpMyAdmin GUI (create table, insert row, edit data), always look at the SQL that phpMyAdmin generated at the bottom of the page. Reading real SQL helps you learn it naturally.

---

## Importing and Exporting Databases

This is essential for: backups, moving databases between servers, sharing your database with teammates, and restoring from a backup.

### Exporting (Creating a Backup)

```
EXPORT A FULL DATABASE:

STEP 1: Click your database in the left panel ("myshop_db")
STEP 2: Click the "Export" tab in the top menu
STEP 3: Choose export method:
  ○ Quick   → exports all tables as a .sql file (default, recommended)
  ○ Custom  → choose specific tables, formats, options

STEP 4: Format: SQL (default — creates a .sql file with all CREATE TABLE + INSERT statements)

STEP 5: Click "Go"
  → Browser downloads: myshop_db.sql

The downloaded .sql file contains everything:
  -- MySQL dump 10.13  Distrib 8.0.xx
  CREATE TABLE `users` (...);
  INSERT INTO `users` VALUES (1, 'Phyo', 'phyo@example.com', '2026-01-01');
  CREATE TABLE `products` (...);
  INSERT INTO `products` VALUES (1, 'T-Shirt', 19.99);
  ...

EXPORT FROM COMMAND LINE (faster for large databases):
  mysqldump -u root -p myshop_db > myshop_db_backup_2026.sql
  mysqldump -u root -p --all-databases > all_databases_backup.sql
```

### Importing (Restoring a Backup)

```
IMPORT A .sql FILE:

STEP 1: Create the target database first if it doesn't exist
  (Either via GUI "New" button, or via SQL tab)

STEP 2: Click on the target database in the left panel

STEP 3: Click the "Import" tab in the top menu

STEP 4: Click "Choose File" → select your .sql file

  ┌────────────────────────────────────────────────────────────┐
  │  File to Import:                                           │
  │  [ Choose File ]  myshop_db.sql                           │
  │                                                            │
  │  Character set:  utf8mb4                                   │
  │  Format:         SQL                                       │
  │                                                            │
  │                              [ Go ]                        │
  └────────────────────────────────────────────────────────────┘

STEP 5: Click "Go"
  ✅ Import has been successfully finished, 15 queries executed.

  ⚠️ Maximum file size: phpMyAdmin has an upload limit.
  For large databases, use command line import instead:
  mysql -u root -p myshop_db < myshop_db_backup.sql
```

> ⚠️ **phpMyAdmin import file size limit:** By default, phpMyAdmin only allows importing files up to 2MB. For larger databases, you need to either:
> - Increase `upload_max_filesize` and `post_max_size` in `php.ini`
> - Use the command line: `mysql -u root -p myshop_db < large_file.sql`

---

## Managing Users in phpMyAdmin

```
STEP 1: Click "User accounts" in the TOP MENU BAR (at server level, not database level)

STEP 2: You see a list of all MySQL users:
  ┌──────────────────┬───────────┬──────────────────────────────────┐
  │ User name        │ Host name │ Password │ Global privileges │ ... │
  ├──────────────────┼───────────┼──────────────────────────────────┤
  │ myapp_user       │ localhost │ Yes      │ SELECT, INSERT... │ ✎ 🗑│
  │ root             │ localhost │ Yes      │ ALL PRIVILEGES    │ ✎ 🗑│
  └──────────────────┴───────────┴──────────────────────────────────┘

STEP 3: To create a new user:
  Click "Add user account" at the bottom of the user list

  ┌──────────────────────────────────────────────────────────────┐
  │  Login Information                                           │
  │  User name: [myshop_user              ]                     │
  │  Host name: localhost   (or % for any host)                 │
  │  Password: [••••••••••••••••          ]                     │
  │  Re-enter: [••••••••••••••••          ]                     │
  │  ◉ Use text field (type password)                           │
  │  ○ Use password generator                                   │
  │                                                             │
  │  Database for user account:                                 │
  │  ☑ Create database with same name and grant all privileges  │
  │  ☑ Grant all privileges on wildcard name (username\_%)      │
  │                                                             │
  │  Global privileges:                                         │
  │  ☐ SELECT ☐ INSERT ☐ UPDATE ☐ DELETE ☐ CREATE ☐ DROP...    │
  │  (Leave unchecked for app users — grant per-database below) │
  │                                                             │
  │                              [ Go ]                         │
  └──────────────────────────────────────────────────────────────┘

STEP 4: To edit an existing user's privileges:
  Click "Edit privileges" (✎) next to any user
  → Change Global or Database-level permissions
  → Click "Go"
```

---

## phpMyAdmin Keyboard Shortcuts & Tips

### Useful Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl + Enter` | Execute SQL query (in SQL editor) |
| `Ctrl + A` | Select all text in SQL editor |
| `Ctrl + Z` | Undo in SQL editor |
| `F5` | Refresh current view |

### Pro Tips

```
TIP 1: Bookmark the phpMyAdmin URL
  Add http://localhost/phpmyadmin to your browser bookmarks.
  You'll open it dozens of times a day during development.

TIP 2: Always check the SQL phpMyAdmin generates
  After any GUI action (create table, insert row, etc.),
  scroll down — phpMyAdmin shows the SQL it ran.
  Copy it into your notes — this is how you learn SQL fast.

TIP 3: Use the SQL tab for everything complex
  GUI clicks are great for simple operations.
  For anything involving WHERE conditions, JOINs, or aggregates
  → type the SQL directly in the SQL tab. Much faster.

TIP 4: Enable "Retain query box"
  Check "Retain query box" under the SQL editor.
  This keeps your SQL visible after running — you can
  modify and re-run it without retyping.

TIP 5: Multiple SQL statements
  You can run multiple SQL statements in one go:
    CREATE TABLE categories (id INT PRIMARY KEY AUTO_INCREMENT, name VARCHAR(100));
    INSERT INTO categories (name) VALUES ('Electronics');
    INSERT INTO categories (name) VALUES ('Clothing');
    INSERT INTO categories (name) VALUES ('Books');
  All 4 statements run in sequence when you click Go.

TIP 6: The "Profiling" checkbox
  Under the SQL editor, there's a "Profiling" checkbox.
  Enable it to see exactly how long each part of your query took.
  Great for identifying slow operations.

TIP 7: Filter rows in Browse
  In Browse view, there's a "Filter rows" input box at the top.
  Type a value → rows are filtered in real-time on the current page.
  (Not a database-level search — just filters what's visible.)

TIP 8: Sort columns
  In Browse view, click any column header to sort ascending.
  Click again to sort descending.
  Very useful for seeing newest/oldest records.

TIP 9: Quick table structure view
  Click a table in the left panel → the right panel shows
  the table name and row count immediately.
  Click "Structure" tab → see all columns, types, indexes.

TIP 10: The "Create PHP code" feature
  After running a SELECT query, phpMyAdmin can generate
  the PHP code to run that same query.
  Click "Create PHP code" under the results.
  Useful when you know the SQL but need the PHP syntax.
```

---

## Quick Revision

- **phpMyAdmin** is a free web-based GUI for MySQL — runs in your browser, accessed at `http://localhost/phpmyadmin`. It's the standard tool on shared hosting and bundled with XAMPP, MAMP, and Laragon.
- **Login:** username = `root` (or your MySQL user), password = your MySQL root password. XAMPP default has no password.
- **Interface layout:** Left panel = navigation tree (databases → tables). Right panel = main content area. Top menu = context-sensitive tabs (Browse, Structure, SQL, Insert, Export, Import).
- **Create a database:** Click "New" in the left panel → enter name → select `utf8mb4_unicode_ci` collation → click "Create". Always use `utf8mb4` for full Unicode and Burmese support.
- **Create a table:** Inside a database, fill in table name + column count → click Go → define each column's name, type, length, NULL, default → set `id` as PRIMARY KEY with AUTO_INCREMENT → click Save.
- **Common column types:** `INT` (whole numbers), `VARCHAR(n)` (short text), `TEXT` (long text), `DECIMAL(10,2)` (money), `TINYINT(1)` (boolean), `DATETIME` (date + time), `TIMESTAMP` (auto-updated time).
- **Insert rows:** Click table → Insert tab → fill in values → click Go. Leave `id` and `CURRENT_TIMESTAMP` columns empty — MySQL fills them automatically.
- **Browse data:** Click table → Browse tab → see all rows in a paginated table with Edit (✎) and Delete (🗑) icons per row.
- **SQL tab:** The most powerful feature — type any SQL and click Go. Results appear instantly. Always check the SQL phpMyAdmin generates after GUI operations — it teaches you SQL.
- **Export (backup):** Click database → Export tab → choose SQL format → click Go → downloads a `.sql` file with all CREATE TABLE and INSERT statements.
- **Import (restore):** Click database → Import tab → choose your `.sql` file → click Go. For large files, use command line: `mysql -u root -p myshop_db < backup.sql`.
- **User management:** Top menu → User accounts → Add user account → set username, host, password, database privileges.
- **Always look at the SQL phpMyAdmin generates** — it's the fastest way to learn SQL naturally while using the GUI.
- **Production security:** Never expose phpMyAdmin publicly. Restrict to specific IPs, move to a non-obvious URL, or use a VPN.