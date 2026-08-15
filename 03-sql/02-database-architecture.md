# Database Architecture — Collections, Communication & How Everything Connects

Understanding how a database is **organized internally** and how all the pieces — Database, DBMS, SQL, and the User — **communicate with each other** is what separates someone who just uses a database from someone who truly understands it. This note is the "big picture" that ties everything together.

---

## Table of Contents

1. [What is a Collection?](#what-is-a-collection)
2. [Database Hierarchy — How Data is Organized](#database-hierarchy--how-data-is-organized)
3. [How Database, DBMS, SQL, and User Communicate](#how-database-dbms-sql-and-user-communicate)
4. [The Request-Response Cycle of a Database](#the-request-response-cycle-of-a-database)
5. [Database Objects — Everything Inside a Database](#database-objects--everything-inside-a-database)
   - [Tables](#tables)
   - [Views](#views)
   - [Indexes](#indexes)
   - [Stored Procedures](#stored-procedures)
   - [Functions](#functions)
   - [Triggers](#triggers)
   - [Events](#events)
   - [Schemas](#schemas)
6. [Keys — The Glue That Holds Data Together](#keys--the-glue-that-holds-data-together)
7. [Relationships Between Tables](#relationships-between-tables)
8. [Constraints — Rules That Protect Your Data](#constraints--rules-that-protect-your-data)
9. [The MySQL Client-Server Model](#the-mysql-client-server-model)
10. [How PHP Talks to MySQL](#how-php-talks-to-mysql)
11. [The Complete Communication Flow](#the-complete-communication-flow)
12. [Quick Revision](#quick-revision)

---

## What is a Collection?

- The word **"collection"** in database context means a **container that holds related data**.
- Different database systems use different words for their collections — but the idea is the same: group related data together.

```
Different databases use different terms for their "collections":

Relational Databases (MySQL, PostgreSQL):
  "Table" → A collection of related rows and columns.
  "Database" or "Schema" → A collection of related tables.

NoSQL Document Databases (MongoDB):
  "Collection" → A group of related documents (equivalent to a SQL table).
  "Database" → A group of related collections.

NoSQL Key-Value (Redis):
  "Database" (numbered 0-15) → A collection of key-value pairs.

NoSQL Wide-Column (Cassandra):
  "Table" → A collection of rows with variable columns.
  "Keyspace" → A collection of tables (equivalent to a SQL database).

Elasticsearch:
  "Index" → A collection of documents (equivalent to a SQL table/database).

────────────────────────────────────────────────────────────────

In MySQL specifically, the word "collection" is NOT used.
MySQL uses:
  DATABASE (or SCHEMA) → contains → TABLES → contain → ROWS
```

### The Term "Collection" in MongoDB

```
In MongoDB (the most common NoSQL database), a "collection" is:
  → A group of JSON-like documents
  → Equivalent to a TABLE in MySQL
  → No fixed schema — each document can have different fields
  → Sits inside a MongoDB database

MongoDB structure:
  MongoDB Server
    ├── myshop (database)
    │     ├── users       ← collection (like a table)
    │     │     ├── { _id: 1, name: "Phyo", email: "phyo@..." }  ← document (like a row)
    │     │     └── { _id: 2, name: "Alice", phone: "09-123" }   ← different fields!
    │     ├── products    ← collection
    │     └── orders      ← collection
    └── blog (database)
          ├── posts       ← collection
          └── comments    ← collection

MySQL structure (equivalent):
  MySQL Server
    ├── myshop (database)
    │     ├── users     ← table (every row has SAME columns)
    │     ├── products  ← table
    │     └── orders    ← table
    └── blog (database)
          ├── posts     ← table
          └── comments  ← table
```

> 💡 **In this note's context (MySQL):** When we say "collection" broadly, we mean any organized container of data — a database, a table, an index, or a view. In MySQL specifically, the primary "collection" is the **table**, and tables are grouped inside a **database** (also called a schema).

---

## Database Hierarchy — How Data is Organized

MySQL organizes everything in a clear hierarchy from the server down to individual data values.

```
MYSQL SERVER (the running software/machine)
│
│ One MySQL server can host MANY databases
│
├── DATABASE: "myshop_db"
│     │
│     │ One database holds MANY tables
│     │
│     ├── TABLE: "users"
│     │     │
│     │     │ One table has MANY columns and MANY rows
│     │     │
│     │     ├── COLUMNS: id | name | email | status | created_at
│     │     │
│     │     ├── ROW 1:   1  | Phyo | phyo@. | active | 2026-01-15
│     │     ├── ROW 2:   2  | Alice| alice@. | active | 2026-02-20
│     │     └── ROW 3:   3  | Bob  | bob@.  | banned | 2026-03-01
│     │
│     ├── TABLE: "products"
│     │     ├── COLUMNS: id | name | price | stock | ...
│     │     └── ROWS: ...
│     │
│     ├── TABLE: "orders"
│     │     ├── COLUMNS: id | user_id | total | status | ...
│     │     └── ROWS: ...
│     │
│     ├── INDEX: (fast lookup structure on users.email)
│     ├── VIEW:  "active_users" (saved SELECT query)
│     └── TRIGGER: (auto-runs code when data changes)
│
├── DATABASE: "blog_db"
│     ├── TABLE: "posts"
│     ├── TABLE: "comments"
│     └── TABLE: "tags"
│
└── DATABASE: "mysql"  ← SYSTEM DATABASE (MySQL's own internal data)
      ├── TABLE: "user"          (stores MySQL user accounts)
      ├── TABLE: "db"            (stores database permissions)
      └── TABLE: "tables_priv"   (stores table-level permissions)
```

### The Five Levels of the MySQL Hierarchy

| Level | What It Is | Example | SQL to See It |
|---|---|---|---|
| 1. Server | The running MySQL instance | MySQL 8.0 on localhost:3306 | `SHOW VARIABLES;` |
| 2. Database | A named collection of tables | `myshop_db`, `blog_db` | `SHOW DATABASES;` |
| 3. Table | A structured collection of rows | `users`, `products` | `SHOW TABLES;` |
| 4. Row | One complete record | Phyo's user record | `SELECT * FROM users;` |
| 5. Cell | A single value in one column | `'phyo@example.com'` | `SELECT email FROM users WHERE id=1;` |

---

## How Database, DBMS, SQL, and User Communicate

This is the heart of understanding how databases work. Let's define each player first, then trace how they talk to each other.

```
THE FOUR PLAYERS:

┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  USER / APPLICATION                                                 │
│  Who: You (the developer), PHP code, phpMyAdmin, MySQL Workbench   │
│  Role: Sends requests (SQL queries) and receives results            │
│                                                                     │
│  SQL                                                                │
│  Who: The language itself (not a person or software)               │
│  Role: The agreed-upon language used to communicate requests       │
│  Example: SELECT * FROM users WHERE status = 'active';              │
│                                                                     │
│  DBMS (Database Management System)                                  │
│  Who: MySQL software (mysqld process running on the server)         │
│  Role: Receives SQL, processes it, executes it, returns results     │
│  Lives: On the server as a running service                          │
│                                                                     │
│  DATABASE                                                           │
│  Who: The actual stored data (files on disk)                        │
│  Role: Passive storage — holds the tables, rows, indexes           │
│  Lives: As files in /var/lib/mysql/ on the server                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### The Communication Chain

```
USER / APPLICATION
      │
      │  "I want all active users"
      │  (forms this into SQL)
      │
      ▼
  SQL QUERY
  SELECT * FROM users WHERE status = 'active';
      │
      │  (sent over TCP/IP to port 3306, or via Unix socket)
      │
      ▼
  DBMS (MySQL server process — mysqld)
      │
      │  Receives the SQL text
      │  Authenticates the connection
      │  Parses the SQL
      │  Checks permissions
      │  Optimizes the query
      │  Executes against the storage engine
      │
      ▼
  DATABASE (data files on disk / in memory)
      │
      │  InnoDB storage engine reads the data
      │  Uses indexes to find matching rows
      │  Returns raw data to the DBMS
      │
      ▼
  DBMS (processes and formats the result)
      │
      │  Assembles the result set
      │  Formats as rows + columns
      │
      ▼
  SQL RESULT SET
  +----+-------+------------------+--------+
  | id | name  | email            | status |
  +----+-------+------------------+--------+
  |  1 | Phyo  | phyo@example.com | active |
  |  2 | Alice | alice@ex.com     | active |
  +----+-------+------------------+--------+
      │
      │  (sent back over the connection)
      │
      ▼
USER / APPLICATION
  PHP receives rows as an array
  Displays to the browser
```

### Analogy — The Restaurant

```
The restaurant analogy maps perfectly to a database:

  YOU (Customer)        = User / PHP Application
  YOUR ORDER (Menu)     = SQL Query
  WAITER                = DBMS (MySQL)
  KITCHEN               = Storage Engine (InnoDB)
  INGREDIENTS (Pantry)  = Database (actual data on disk)

1. You (PHP) write a "food order" (SQL query)
2. The waiter (DBMS) takes your order
   → Checks if you're allowed to order this (authentication)
   → Translates it to kitchen instructions (query optimization)
3. The kitchen (storage engine) prepares the food (reads data)
4. The waiter (DBMS) brings it to your table (returns result set)
5. You (PHP) enjoy the meal (uses the data)

Key insight: YOU don't go into the kitchen yourself (directly accessing data files).
The WAITER (DBMS) is always in the middle — protecting the kitchen (database).
```

---

## The Request-Response Cycle of a Database

Every interaction with MySQL follows this detailed cycle:

```
PHASE 1 — CONNECTION ESTABLISHMENT
─────────────────────────────────────────────────────────────────────

  PHP Code:
  $pdo = new PDO("mysql:host=localhost;dbname=myshop_db", "app_user", "password");

  What happens:
  ① PHP sends a connection request to MySQL on port 3306
  ② MySQL's Connection Manager receives it
  ③ MySQL checks: Does this user exist?
                  Is the password correct?
                  Is this user allowed to connect from this host?
  ④ If all checks pass → connection is established
  ⑤ MySQL creates a "session" — a dedicated thread for this connection
  ⑥ PHP receives a connection handle ($pdo)

  Cost: ~1-5ms per connection (this is why connection pooling matters)

─────────────────────────────────────────────────────────────────────

PHASE 2 — QUERY SENDING
─────────────────────────────────────────────────────────────────────

  PHP Code:
  $stmt = $pdo->prepare("SELECT * FROM users WHERE status = ? AND role = ?");
  $stmt->execute(['active', 'admin']);

  What happens:
  ① PHP sends the SQL text to MySQL over the established connection
  ② MySQL receives it as a raw text string

─────────────────────────────────────────────────────────────────────

PHASE 3 — QUERY PROCESSING (inside MySQL)
─────────────────────────────────────────────────────────────────────

  Step A: PARSING
    MySQL reads: "SELECT * FROM users WHERE status = ? AND role = ?"
    Checks SQL syntax — is this valid SQL?
    Builds an internal parse tree (abstract syntax tree)
    Identifies: SELECT statement, table=users, WHERE conditions

  Step B: PREPROCESSING
    Resolves table and column names
    Checks: Does table 'users' exist in 'myshop_db'?
            Does column 'status' exist in 'users'?
            Does column 'role' exist in 'users'?
    Checks permissions: Can 'app_user' SELECT from 'myshop_db.users'?

  Step C: OPTIMIZATION (The Query Optimizer — the smart part)
    MySQL looks at:
    → What indexes exist on the users table?
    → How many rows in the table? (~10,000)
    → How many rows have status='active'? (~8,000, 80%)
    → How many rows have role='admin'? (~50, 0.5%)
    → Cost of full table scan vs index usage?

    Decision: "Use the index on 'role' column first
               (only 50 admins) then filter for 'active'"
    This is the EXECUTION PLAN — the chosen strategy.

    You can see the execution plan with:
    EXPLAIN SELECT * FROM users WHERE status = 'active' AND role = 'admin';

  Step D: EXECUTION
    The Storage Engine (InnoDB) executes the plan:
    → Checks the Buffer Pool (RAM cache) first — is this data already in memory?
    → If yes → reads from RAM (fast: microseconds)
    → If no  → reads from disk (.ibd files) → loads into Buffer Pool for next time
    → Applies WHERE filters
    → Assembles matching rows

─────────────────────────────────────────────────────────────────────

PHASE 4 — RESULT TRANSMISSION
─────────────────────────────────────────────────────────────────────

  MySQL sends back:
  → Column metadata (names, types, sizes)
  → The actual rows of data
  → Status: "2 rows returned in 0.0008 sec"

  PHP receives:
  → Array of rows (via PDO::fetch())
  → Each row is an associative array: ['id'=>1,'name'=>'Phyo',...]

─────────────────────────────────────────────────────────────────────

PHASE 5 — CONNECTION CLOSE (or reuse)
─────────────────────────────────────────────────────────────────────

  PHP:
  $pdo = null;  // closes the connection

  OR: Connection Pooling keeps the connection open for reuse
  (saves the 1-5ms connection cost on the next query)
```

---

## Database Objects — Everything Inside a Database

A database is not just tables. It contains several types of **objects**, each serving a different purpose.

---

### Tables

- The primary storage unit — rows and columns of data.
- Everything else (views, indexes, triggers) is built around tables.

```sql
-- Create a table
CREATE TABLE users (
  id         INT UNSIGNED NOT NULL AUTO_INCREMENT,
  name       VARCHAR(100) NOT NULL,
  email      VARCHAR(255) NOT NULL,
  created_at TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id)
);

-- See all tables in current database
SHOW TABLES;

-- See table structure
DESCRIBE users;
SHOW CREATE TABLE users;
```

---

### Views

- A **view** is a **saved SELECT query** that looks and behaves like a table.
- It doesn't store data itself — when you query a view, MySQL runs the underlying SELECT.
- Use views to simplify complex queries, hide sensitive columns, or present data in a specific format.

```sql
-- Create a view — "active_users" shows only non-deleted active users
CREATE VIEW active_users AS
  SELECT id, name, email, role, created_at
  FROM users
  WHERE status = 'active' AND deleted_at IS NULL;

-- Use the view just like a table
SELECT * FROM active_users;
SELECT * FROM active_users WHERE role = 'admin';

-- View hides the password column — safe to share with read-only users
-- View always shows current data — it's a live SELECT, not a copy

-- Create a more complex view
CREATE VIEW order_summary AS
  SELECT
    o.id           AS order_id,
    u.name         AS customer_name,
    u.email        AS customer_email,
    o.total        AS order_total,
    o.status       AS order_status,
    o.created_at   AS ordered_at
  FROM orders o
  JOIN users u ON o.user_id = u.id
  WHERE o.deleted_at IS NULL;

-- Query the complex view easily
SELECT * FROM order_summary WHERE order_status = 'shipped';

-- See all views
SHOW FULL TABLES WHERE Table_type = 'VIEW';

-- Drop a view
DROP VIEW IF EXISTS active_users;
```

---

### Indexes

- An **index** is a separate data structure that MySQL maintains to speed up data retrieval.
- Like the index at the back of a book — instead of reading every page, you jump straight to what you need.
- MySQL automatically maintains indexes when you INSERT, UPDATE, or DELETE rows.

```sql
-- Types of indexes:

-- PRIMARY KEY index (created with the table)
CREATE TABLE users (
  id INT UNSIGNED NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (id)           ← MySQL creates a clustered index on id
);

-- UNIQUE index (no duplicates allowed)
CREATE UNIQUE INDEX users_email_unique ON users (email);
ALTER TABLE users ADD UNIQUE INDEX users_email_unique (email);

-- Regular INDEX (allows duplicates — for speeding up WHERE/ORDER BY)
CREATE INDEX users_status_index ON users (status);
CREATE INDEX users_created_at_index ON users (created_at);

-- COMPOSITE index (covers multiple columns — for queries that filter on both)
CREATE INDEX orders_user_status ON orders (user_id, status);
-- Fast for: WHERE user_id = 1 AND status = 'shipped'

-- FULLTEXT index (for text search)
CREATE FULLTEXT INDEX products_search ON products (name, description);
-- Enables: SELECT * FROM products WHERE MATCH(name, description) AGAINST('wireless headphones');

-- See all indexes on a table
SHOW INDEX FROM users;

-- Drop an index
DROP INDEX users_status_index ON users;
ALTER TABLE users DROP INDEX users_status_index;
```

```
How an index works (B-tree):

Without index: Find user with email = 'phyo@example.com'
  MySQL reads row 1 → not a match
  MySQL reads row 2 → not a match
  ...
  MySQL reads row 50,000 → MATCH!
  Time: O(n) — gets slower as table grows

With index on email:
  MySQL checks the B-tree index → directly finds the page for 'phyo@...'
  Time: O(log n) — stays fast even with 100 million rows!

Trade-off: Indexes speed up reads but slow down writes slightly
(every INSERT/UPDATE/DELETE must also update all indexes on that table)
Rule: Index columns you frequently filter, sort, or join on.
Don't over-index — every index costs disk space and write performance.
```

---

### Stored Procedures

- A **stored procedure** is a named, reusable block of SQL code stored in the database itself.
- Called by name, can accept parameters, return results, contain logic (IF, WHILE, LOOP).
- Runs inside MySQL — the logic executes on the database server, not in PHP.

```sql
-- Create a stored procedure
DELIMITER //

CREATE PROCEDURE GetActiveUsers(IN user_role VARCHAR(50))
BEGIN
  SELECT id, name, email, role, created_at
  FROM users
  WHERE status = 'active'
    AND role = user_role
    AND deleted_at IS NULL
  ORDER BY name ASC;
END //

DELIMITER ;

-- Call the procedure
CALL GetActiveUsers('admin');
CALL GetActiveUsers('user');

-- Procedure with output parameter
DELIMITER //

CREATE PROCEDURE GetUserCount(IN user_status VARCHAR(20), OUT total INT)
BEGIN
  SELECT COUNT(*) INTO total
  FROM users
  WHERE status = user_status;
END //

DELIMITER ;

-- Call and get the output
CALL GetUserCount('active', @count);
SELECT @count;   -- 42

-- See all stored procedures
SHOW PROCEDURE STATUS WHERE Db = 'myshop_db';

-- Drop a procedure
DROP PROCEDURE IF EXISTS GetActiveUsers;
```

---

### Functions

- Like stored procedures but **always return a single value** and can be used inside SELECT statements.
- Two types: **built-in functions** (provided by MySQL: `COUNT()`, `NOW()`, `UPPER()`) and **user-defined functions** (you write them).

```sql
-- Create a user-defined function
DELIMITER //

CREATE FUNCTION CalculateDiscount(price DECIMAL(10,2), discount_percent DECIMAL(5,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
  RETURN price - (price * discount_percent / 100);
END //

DELIMITER ;

-- Use the function just like a built-in function
SELECT
  name,
  price,
  CalculateDiscount(price, 10) AS price_after_10_percent_off
FROM products;

-- Another useful function: format a full name
DELIMITER //

CREATE FUNCTION FormatName(first_name VARCHAR(50), last_name VARCHAR(50))
RETURNS VARCHAR(101)
DETERMINISTIC
BEGIN
  RETURN CONCAT(first_name, ' ', last_name);
END //

DELIMITER ;

SELECT FormatName('Phyo', 'Min Paing');  -- Returns: Phyo Min Paing

-- See all functions
SHOW FUNCTION STATUS WHERE Db = 'myshop_db';

-- Drop a function
DROP FUNCTION IF EXISTS CalculateDiscount;
```

---

### Triggers

- A **trigger** is a procedure that **automatically executes** when a specific event (INSERT, UPDATE, DELETE) happens on a table.
- You don't call triggers — MySQL fires them automatically.
- Useful for auditing, logging, maintaining derived data, enforcing business rules.

```sql
-- Trigger: log every time a user's status changes
CREATE TABLE user_audit_log (
  id         INT UNSIGNED NOT NULL AUTO_INCREMENT,
  user_id    INT UNSIGNED NOT NULL,
  field      VARCHAR(100) NOT NULL,
  old_value  TEXT NULL,
  new_value  TEXT NULL,
  changed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id)
);

DELIMITER //

CREATE TRIGGER users_status_change
AFTER UPDATE ON users
FOR EACH ROW
BEGIN
  IF OLD.status != NEW.status THEN
    INSERT INTO user_audit_log (user_id, field, old_value, new_value)
    VALUES (NEW.id, 'status', OLD.status, NEW.status);
  END IF;
END //

DELIMITER ;

-- Now: whenever a user's status changes...
UPDATE users SET status = 'banned' WHERE id = 5;
-- ...MySQL automatically inserts a row into user_audit_log!

-- Trigger types:
-- BEFORE INSERT / AFTER INSERT
-- BEFORE UPDATE / AFTER UPDATE
-- BEFORE DELETE / AFTER DELETE

-- Trigger to auto-set updated_at (alternative to ON UPDATE CURRENT_TIMESTAMP)
DELIMITER //

CREATE TRIGGER users_updated_at
BEFORE UPDATE ON users
FOR EACH ROW
BEGIN
  SET NEW.updated_at = NOW();
END //

DELIMITER ;

-- See all triggers
SHOW TRIGGERS;
SHOW TRIGGERS FROM myshop_db;

-- Drop a trigger
DROP TRIGGER IF EXISTS users_status_change;
```

---

### Events

- A **MySQL Event** is like a cron job inside MySQL — scheduled SQL that runs automatically at a specified time or interval.
- Requires the Event Scheduler to be enabled.

```sql
-- Enable the event scheduler
SET GLOBAL event_scheduler = ON;

-- Event: Delete old soft-deleted records every night at 2 AM
CREATE EVENT purge_old_deleted_records
ON SCHEDULE EVERY 1 DAY STARTS '2026-07-01 02:00:00'
DO
  DELETE FROM users
  WHERE deleted_at IS NOT NULL
    AND deleted_at < (NOW() - INTERVAL 90 DAY);

-- Event: Reset daily stats every midnight
CREATE EVENT reset_daily_stats
ON SCHEDULE EVERY 1 DAY STARTS (CURRENT_DATE + INTERVAL 1 DAY)
DO BEGIN
  TRUNCATE TABLE daily_page_views;
  TRUNCATE TABLE daily_search_queries;
END;

-- See all events
SHOW EVENTS;

-- Drop an event
DROP EVENT IF EXISTS purge_old_deleted_records;
```

---

### Schemas

- In MySQL, **schema** and **database** are synonyms — they mean exactly the same thing.
- `CREATE SCHEMA myapp;` does the same thing as `CREATE DATABASE myapp;`
- In other databases (PostgreSQL, Oracle), schema means a namespace within a database — a subtle but important distinction.

```sql
-- In MySQL: these are identical
CREATE DATABASE myshop_db;
CREATE SCHEMA myshop_db;   -- same thing!

USE myshop_db;
USE SCHEMA myshop_db;      -- same!

-- In PostgreSQL: different meaning
-- A database contains schemas, schemas contain tables:
-- PostgreSQL: database → schema → table
-- MySQL:      database (= schema) → table
```

### Database Objects Summary

| Object | Stores | Runs When | Use For |
|---|---|---|---|
| **Table** | Actual rows of data | — | Primary data storage |
| **View** | A saved SELECT query (no data) | On query | Simplify complex queries, hide columns |
| **Index** | B-tree lookup structure | On query | Speed up reads |
| **Stored Procedure** | Reusable SQL code with logic | On CALL | Complex operations, reduce network trips |
| **Function** | Returns a single value | In SELECT/WHERE | Reusable calculations |
| **Trigger** | Auto-fired SQL | On INSERT/UPDATE/DELETE | Auditing, automation, data integrity |
| **Event** | Scheduled SQL | On schedule (cron) | Maintenance, cleanup, periodic reports |

---

## Keys — The Glue That Holds Data Together

Keys are the mechanism that defines identity and relationships between tables.

```
PRIMARY KEY:
  Uniquely identifies every row in a table.
  Only ONE primary key per table.
  Cannot be NULL. Cannot have duplicate values.
  MySQL creates a clustered index on the primary key.

  Example: id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY

  ┌────┬────────┬──────────────────────┐
  │ id │ name   │ email                │  ← id is the PRIMARY KEY
  ├────┼────────┼──────────────────────┤
  │  1 │ Phyo   │ phyo@example.com     │  ← id=1 uniquely identifies this row
  │  2 │ Alice  │ alice@example.com    │  ← id=2 uniquely identifies this row
  └────┴────────┴──────────────────────┘

──────────────────────────────────────────────────────────────────────

FOREIGN KEY:
  A column in one table that references the PRIMARY KEY of another table.
  Creates a relationship between tables.
  Enforces referential integrity — you can't reference a row that doesn't exist.

  Example:
  orders.user_id → references → users.id

  users table:                    orders table:
  ┌────┬────────┐                 ┌────┬─────────┬────────┐
  │ id │ name   │                 │ id │ user_id │ total  │
  ├────┼────────┤                 ├────┼─────────┼────────┤
  │  1 │ Phyo   │◄────────────────│  1 │    1    │  99.99 │
  │  2 │ Alice  │◄────────────────│  2 │    1    │  45.00 │
  └────┴────────┘     ◄───────────│  3 │    2    │ 150.00 │
                                   └────┴─────────┴────────┘

  user_id=1 references users.id=1 (Phyo has 2 orders)
  user_id=2 references users.id=2 (Alice has 1 order)
  You CANNOT insert an order with user_id=99 if no user with id=99 exists!

──────────────────────────────────────────────────────────────────────

UNIQUE KEY:
  Like a primary key but allows multiple per table.
  Ensures no two rows have the same value in this column.
  Can be NULL (but only one NULL per unique column).

  Example: email VARCHAR(255) UNIQUE
  → If Phyo registers with phyo@example.com, nobody else can use that email.

──────────────────────────────────────────────────────────────────────

COMPOSITE KEY:
  A key made of TWO OR MORE columns together.
  The combination must be unique, but individual columns can repeat.

  Example: A student cannot be enrolled in the same course twice.
  PRIMARY KEY (student_id, course_id) ← the combination is unique

  ┌────────────┬───────────┐
  │ student_id │ course_id │  ← (student_id + course_id) together = unique
  ├────────────┼───────────┤
  │     1      │    101    │  ← Phyo in PHP course ✅
  │     1      │    202    │  ← Phyo in MySQL course ✅
  │     2      │    101    │  ← Alice in PHP course ✅
  │     1      │    101    │  ← Phyo in PHP course AGAIN ❌ DUPLICATE!
  └────────────┴───────────┘
```

---

## Relationships Between Tables

The "relational" in "relational database" refers to these relationships — the different ways tables can be connected.

### One-to-Many (Most Common)

```
One user can have MANY orders.
One order belongs to EXACTLY ONE user.

users           orders
┌────┬──────┐   ┌────┬─────────┬────────┐
│ id │ name │   │ id │ user_id │ total  │
├────┼──────┤   ├────┼─────────┼────────┤
│  1 │ Phyo │──►│  1 │    1    │  99.99 │
│    │      │──►│  2 │    1    │  45.00 │
│  2 │Alice │──►│  3 │    2    │ 150.00 │
└────┴──────┘   └────┴─────────┴────────┘
  1 user : MANY orders

Implementation: Put the foreign key on the "many" side.
orders.user_id references users.id
```

### One-to-One

```
One user has exactly ONE profile.
One profile belongs to exactly ONE user.

users                     user_profiles
┌────┬──────────┐         ┌─────────┬──────┬─────────┐
│ id │ name     │         │ user_id │ bio  │ website │
├────┼──────────┤         ├─────────┼──────┼─────────┤
│  1 │ Phyo     │────────►│    1    │"Dev" │"phyo.io"│
│  2 │ Alice    │────────►│    2    │"Dev" │ NULL    │
└────┴──────────┘         └─────────┴──────┴─────────┘

Implementation: user_profiles.user_id is UNIQUE (ensures one-to-one).
```

### Many-to-Many

```
One order can have MANY products.
One product can be in MANY orders.
(You need a JUNCTION/PIVOT table in the middle)

orders           order_items (junction)    products
┌────┬───────┐   ┌──────────┬────────────┬─────┐   ┌────┬──────────┐
│ id │ total │   │ order_id │ product_id │ qty │   │ id │ name     │
├────┼───────┤   ├──────────┼────────────┼─────┤   ├────┼──────────┤
│  1 │ 99.99 │──►│    1     │     5      │  1  │──►│  5 │ Shirt    │
│    │       │──►│    1     │     8      │  2  │──►│  8 │ Pants    │
│  2 │ 45.00 │──►│    2     │     5      │  1  │──►│  5 │ Shirt    │
└────┴───────┘   └──────────┴────────────┴─────┘   └────┴──────────┘

Order 1 has 2 products (Shirt x1, Pants x2)
Order 2 has 1 product (Shirt x1)
Shirt appears in both orders

The order_items table has a COMPOSITE PRIMARY KEY: (order_id, product_id)
```

### Self-Referential (Hierarchical)

```
A category can have a PARENT category (which is also a category).
Employees can have a manager (who is also an employee).

categories
┌────┬───────────────┬───────────┐
│ id │ name          │ parent_id │
├────┼───────────────┼───────────┤
│  1 │ Electronics   │   NULL    │ ← top-level (no parent)
│  2 │ Computers     │     1     │ ← child of Electronics
│  3 │ Laptops       │     2     │ ← child of Computers
│  4 │ Gaming Laptops│     3     │ ← child of Laptops
│  5 │ Phones        │     1     │ ← child of Electronics
└────┴───────────────┴───────────┘

parent_id references id in the SAME TABLE
Electronics → Computers → Laptops → Gaming Laptops (tree structure)
```

---

## Constraints — Rules That Protect Your Data

Constraints are rules enforced by the DBMS that prevent invalid data from ever entering the database.

```sql
-- NOT NULL — column must always have a value
name VARCHAR(100) NOT NULL

-- UNIQUE — no two rows can have the same value in this column
email VARCHAR(255) NOT NULL UNIQUE

-- PRIMARY KEY — NOT NULL + UNIQUE + clustered index
id INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY

-- FOREIGN KEY — value must exist in the referenced table
FOREIGN KEY (user_id) REFERENCES users(id)

-- CHECK — value must satisfy a condition (MySQL 8.0.16+)
age   TINYINT UNSIGNED CHECK (age >= 18 AND age <= 120)
price DECIMAL(10,2)    CHECK (price >= 0)
email VARCHAR(255)     CHECK (email LIKE '%@%.%')

-- DEFAULT — value if not specified on INSERT
status ENUM('active','inactive') NOT NULL DEFAULT 'active'
stock  INT UNSIGNED NOT NULL DEFAULT 0
balance DECIMAL(10,2) NOT NULL DEFAULT 0.00

-- AUTO_INCREMENT — auto-assign next sequential integer
id INT UNSIGNED NOT NULL AUTO_INCREMENT

-- Complete table with all constraint types:
CREATE TABLE products (
  id          INT UNSIGNED   NOT NULL AUTO_INCREMENT,
  name        VARCHAR(255)   NOT NULL,
  sku         VARCHAR(100)   NOT NULL,
  price       DECIMAL(10,2)  NOT NULL DEFAULT 0.00,
  stock       INT UNSIGNED   NOT NULL DEFAULT 0,
  category_id INT UNSIGNED   NULL,
  is_active   TINYINT(1)     NOT NULL DEFAULT 1,
  created_at  TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP,

  PRIMARY KEY (id),
  UNIQUE KEY products_sku_unique (sku),          -- SKU must be unique
  KEY products_category_idx (category_id),       -- for fast category filtering
  KEY products_is_active_idx (is_active),        -- for fast active filtering

  CONSTRAINT products_category_fk
    FOREIGN KEY (category_id) REFERENCES categories(id)
    ON DELETE SET NULL    -- if category deleted → set category_id to NULL
    ON UPDATE CASCADE,    -- if category id changes → update here too

  CONSTRAINT products_price_positive
    CHECK (price >= 0),   -- price cannot be negative

  CONSTRAINT products_stock_positive
    CHECK (stock >= 0)    -- stock cannot be negative

) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Foreign Key Actions (ON DELETE / ON UPDATE)

```
RESTRICT (default):
  → Prevents the action if related rows exist
  → Cannot delete a user if they have orders
  → Safest option

CASCADE:
  → Automatically performs the same action on related rows
  → Delete a user → automatically deletes all their orders too
  → Dangerous if used carelessly

SET NULL:
  → Sets the foreign key column to NULL in related rows
  → Delete a category → sets category_id to NULL in all products
  → Only works if the FK column is nullable

SET DEFAULT:
  → Sets the FK column to its default value
  → Rarely used

NO ACTION:
  → Same as RESTRICT in MySQL (differs in some other databases)

ON DELETE CASCADE example:
  DELETE FROM users WHERE id = 5;
  → automatically deletes all orders WHERE user_id = 5
  → and all order_items WHERE order_id IN (those order ids)
  → cascade goes as deep as the FK chain

Best practice: Use RESTRICT as default. Only use CASCADE when
you truly want automatic deletion of child records.
```

---

## The MySQL Client-Server Model

MySQL uses a **client-server architecture** — the server and clients are separate processes, possibly on different machines.

```
┌─────────────────────────────────────────────────────────────────────┐
│                     MYSQL CLIENT-SERVER MODEL                       │
│                                                                     │
│  CLIENTS (anywhere on the network):                                 │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────────┐   │
│  │  PHP App       │  │  phpMyAdmin    │  │  MySQL Workbench   │   │
│  │  (PDO/MySQLi)  │  │  (web browser) │  │  (desktop GUI)     │   │
│  └───────┬────────┘  └───────┬────────┘  └────────┬───────────┘   │
│          │                   │                     │               │
│          └───────────────────┼─────────────────────┘               │
│                              │ MySQL Protocol over TCP/IP           │
│                              │ Host: localhost, Port: 3306          │
│                              │                                     │
│                              ▼                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    MYSQL SERVER (mysqld)                      │  │
│  │                                                              │  │
│  │  ┌─────────────────────────────────────────────────────┐    │  │
│  │  │              Connection Manager                      │    │  │
│  │  │  One thread per connection, handles auth             │    │  │
│  │  └─────────────────────────────────────────────────────┘    │  │
│  │                           │                                  │  │
│  │  ┌────────────┐  ┌────────────────┐  ┌──────────────────┐   │  │
│  │  │   Parser   │  │  Preprocessor  │  │   Optimizer      │   │  │
│  │  └────────────┘  └────────────────┘  └──────────────────┘   │  │
│  │                           │                                  │  │
│  │  ┌─────────────────────────────────────────────────────┐    │  │
│  │  │              Storage Engine Layer                    │    │  │
│  │  │  InnoDB  │  MyISAM  │  Memory  │  CSV  │  Archive   │    │  │
│  │  └─────────────────────────────────────────────────────┘    │  │
│  │                           │                                  │  │
│  │  ┌─────────────────────────────────────────────────────┐    │  │
│  │  │              File System (Disk)                      │    │  │
│  │  │  /var/lib/mysql/myshop_db/users.ibd                 │    │  │
│  │  │  /var/lib/mysql/myshop_db/products.ibd              │    │  │
│  │  └─────────────────────────────────────────────────────┘    │  │
│  └──────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘

Connection options:
  localhost via TCP:  host=127.0.0.1, port=3306
  localhost via socket: socket=/var/run/mysqld/mysqld.sock (faster on Linux)
  remote server:      host=db.example.com, port=3306
```

---

## How PHP Talks to MySQL

This is the practical connection between your PHP code and the database.

```php
<?php
// ─── The Full Communication Stack ────────────────────────────────────────

// 1. PHP establishes a connection to MySQL
$pdo = new PDO(
    "mysql:host=localhost;port=3306;dbname=myshop_db;charset=utf8mb4",
    "app_user",
    "secure_password",
    [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,  // throw exceptions
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,        // arrays not objects
        PDO::ATTR_EMULATE_PREPARES   => false,                    // real prepared statements
    ]
);
// What happened: PHP opened a TCP connection to port 3306
//                MySQL authenticated "app_user"
//                MySQL selected "myshop_db" as the current database
//                A session (thread) was created on the MySQL server

// 2. PHP sends a parameterized query
//    (Parameters prevent SQL injection — ALWAYS use these)
$stmt = $pdo->prepare(
    "SELECT id, name, email, role
     FROM users
     WHERE status = :status AND deleted_at IS NULL
     ORDER BY name ASC
     LIMIT :limit"
);

// 3. PHP binds values and executes
$stmt->bindValue(':status', 'active', PDO::PARAM_STR);
$stmt->bindValue(':limit',  10,       PDO::PARAM_INT);
$stmt->execute();
// What happened: MySQL's parser checked the SQL syntax
//                MySQL's optimizer created an execution plan
//                InnoDB read the matching rows from disk/cache

// 4. PHP retrieves the results
$users = $stmt->fetchAll();
// What happened: MySQL sent the rows back over the connection
//                PDO converted them to PHP arrays

// 5. Use the data
foreach ($users as $user) {
    echo $user['name'] . " - " . $user['email'] . "\n";
}

// 6. Connection is automatically closed when $pdo goes out of scope
//    OR: $pdo = null; // explicit close
?>
```

### Why Prepared Statements Matter

```php
<?php
// ❌ DANGEROUS — SQL Injection vulnerability
$status = $_GET['status'];  // attacker sends: ' OR '1'='1
$sql    = "SELECT * FROM users WHERE status = '$status'";
// Resulting SQL:
// SELECT * FROM users WHERE status = '' OR '1'='1'
// Returns ALL users regardless of status — attacker bypassed the filter!

// ✅ SAFE — Prepared Statement (parameterized query)
$stmt = $pdo->prepare("SELECT * FROM users WHERE status = ?");
$stmt->execute([$_GET['status']]);
// MySQL treats the ? value as pure DATA, never as SQL code
// Even if attacker sends "' OR '1'='1", it's treated as a literal string value
?>
```

---

## The Complete Communication Flow

Let's trace a complete real-world request from a user clicking a button to data appearing on screen.

```
SCENARIO: User clicks "View My Orders" on an e-commerce site

─────────────────────────────────────────────────────────────────────
LAYER 1: USER (Browser)
─────────────────────────────────────────────────────────────────────
  User: clicks "My Orders" button
  Browser: sends HTTP GET request to https://shop.example.com/orders
  Cookie: session_id=abc123 (sent automatically)

─────────────────────────────────────────────────────────────────────
LAYER 2: WEB SERVER (Nginx)
─────────────────────────────────────────────────────────────────────
  Nginx receives the request
  Routes it to PHP-FPM (application server)
  Passes the session cookie along

─────────────────────────────────────────────────────────────────────
LAYER 3: PHP APPLICATION (your code)
─────────────────────────────────────────────────────────────────────
  PHP starts:
    session_start();
    $userId = $_SESSION['user_id'];  // e.g., 42 — from Redis session store

  PHP checks Redis cache first:
    $cacheKey = "user:42:orders";
    $cached   = $redis->get($cacheKey);
    if ($cached) return json_decode($cached);  // ← cache hit, skip MySQL

  Cache miss → PHP queries MySQL:
    $sql  = "SELECT o.id, o.total, o.status, o.created_at,
                    COUNT(oi.id) AS item_count
             FROM orders o
             JOIN order_items oi ON o.id = oi.order_id
             WHERE o.user_id = :user_id
               AND o.deleted_at IS NULL
             GROUP BY o.id
             ORDER BY o.created_at DESC
             LIMIT 10";

    $stmt = $pdo->prepare($sql);
    $stmt->execute([':user_id' => $userId]);
    $orders = $stmt->fetchAll();

─────────────────────────────────────────────────────────────────────
LAYER 4: MYSQL (DBMS)
─────────────────────────────────────────────────────────────────────
  Connection Manager: verifies 'app_user'@'localhost'
  Parser: validates SQL syntax, builds parse tree
  Preprocessor: confirms tables/columns exist, checks permissions
  Optimizer:
    → Sees JOIN between orders and order_items
    → Checks indexes: orders.user_id has an INDEX ✅
    → Checks indexes: order_items.order_id has an INDEX ✅
    → Plan: "Use orders.user_id index to get user 42's orders (fast),
              then join order_items for each order"
  Execution:
    InnoDB checks Buffer Pool (RAM) for the data
    → Found in cache! No disk read needed
    → Filters deleted_at IS NULL: 2 orders excluded
    → Groups by order id
    → Sorts by created_at DESC
    → Returns first 10 rows

─────────────────────────────────────────────────────────────────────
LAYER 5: BACK IN PHP
─────────────────────────────────────────────────────────────────────
  PHP receives the array of order rows
  Stores in Redis cache for next time (5 minute TTL):
    $redis->setex($cacheKey, 300, json_encode($orders));

  PHP builds HTML:
    foreach ($orders as $order) { ... }
  OR for an API:
    echo json_encode(['status' => 'success', 'data' => $orders]);

─────────────────────────────────────────────────────────────────────
LAYER 6: BACK TO USER
─────────────────────────────────────────────────────────────────────
  Nginx sends the HTTP response to the browser
  Browser renders the HTML / JavaScript processes the JSON
  User sees their order list

Total time: ~15ms (including 2ms MySQL query, 1ms Redis cache store)
```

---

## Quick Revision

- A **collection** is any organized container of related data. In MySQL = table/database. In MongoDB = collection. In Cassandra = keyspace/table. In Redis = database.
- MySQL organizes data in 5 levels: **Server → Database → Table → Row → Cell**. One server hosts many databases, one database holds many tables, one table holds many rows.
- The **4 players:** USER/APP (sends SQL), SQL (the language), DBMS (processes everything), DATABASE (stores data on disk). The DBMS is ALWAYS the middleware — you never touch raw data files directly.
- The **5 phases of a query:** Connection → Parsing → Preprocessing (permission check) → Optimization (choosing the best execution plan) → Execution (reading data) → Result returned.
- **Database objects:** Tables (data), Views (saved queries), Indexes (fast lookups), Stored Procedures (reusable logic), Functions (return a value), Triggers (auto-fire on changes), Events (scheduled cron-like SQL).
- **Key types:** Primary Key (unique row identity, only one per table), Foreign Key (links to another table's PK, enforces referential integrity), Unique Key (no duplicates, multiple per table), Composite Key (multiple columns together form a key).
- **Relationship types:** One-to-Many (user has many orders — most common), One-to-One (user has one profile — FK with UNIQUE), Many-to-Many (orders and products — needs a junction table).
- **Constraints:** NOT NULL, UNIQUE, PRIMARY KEY, FOREIGN KEY, CHECK, DEFAULT, AUTO_INCREMENT — these are rules the DBMS enforces to prevent invalid data.
- **Foreign key actions:** RESTRICT (prevent deletion if referenced), CASCADE (auto-delete/update children), SET NULL (set FK to NULL), SET DEFAULT.
- **MySQL Client-Server:** MySQL runs as a service (mysqld) on port 3306. Clients (PHP, phpMyAdmin, Workbench) connect via TCP/IP or Unix socket, authenticate, and communicate via the MySQL protocol.
- **PHP communicates via PDO:** prepare → bindValue → execute → fetchAll. Always use **prepared statements with parameters** — never concatenate user input into SQL (SQL injection risk).
- **The complete flow:** Browser → HTTP → Nginx → PHP → Redis check → MySQL → parse → optimize → execute → InnoDB → Buffer Pool → result → PHP → format → HTTP → Browser.
- **EXPLAIN** is your best debugging tool: `EXPLAIN SELECT * FROM users WHERE email = ?;` shows you the execution plan, whether indexes are used, and how many rows MySQL will scan.