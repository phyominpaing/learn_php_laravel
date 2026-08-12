# SQL — The Language of Databases

**SQL** (Structured Query Language) is the language you use to communicate with a relational database. It is the single most important language a backend developer must master — every piece of data your application reads, writes, updates, or deletes goes through SQL.

---

## Table of Contents

1. [What is SQL?](#what-is-sql)
2. [A Brief History of SQL](#a-brief-history-of-sql)
3. [How SQL Works](#how-sql-works)
4. [SQL Sublanguages — The Four Categories](#sql-sublanguages--the-four-categories)
   - [DDL — Data Definition Language](#ddl--data-definition-language)
   - [DML — Data Manipulation Language](#dml--data-manipulation-language)
   - [DQL — Data Query Language](#dql--data-query-language)
   - [DCL — Data Control Language](#dcl--data-control-language)
   - [TCL — Transaction Control Language](#tcl--transaction-control-language)
5. [SQL Data Types in MySQL](#sql-data-types-in-mysql)
   - [Numeric Types](#numeric-types)
   - [String Types](#string-types)
   - [Date and Time Types](#date-and-time-types)
   - [Binary Types](#binary-types)
   - [Special Types](#special-types)
6. [SQL Syntax Rules](#sql-syntax-rules)
7. [SQL Comments](#sql-comments)
8. [Naming Conventions](#naming-conventions)
9. [MySQL vs SQL Standard](#mysql-vs-sql-standard)
10. [Quick Revision](#quick-revision)

---

## What is SQL?

- **SQL** stands for **Structured Query Language** — pronounced "S-Q-L" or "sequel" (both are correct and widely used).
- It is a **special-purpose language** designed specifically for communicating with relational databases.
- SQL is **declarative** — you describe *what* you want, not *how* to get it. The database engine figures out the most efficient way to execute your request.
- SQL is used for **everything** you do with a relational database:
  - Creating tables and databases
  - Inserting, reading, updating, and deleting data
  - Managing users and permissions
  - Controlling transactions

```
Imperative (how to do it — like PHP loops):
  for each row in the users file:
      if row.status == 'active':
          add row to results
      sort results by name

Declarative SQL (what you want — database handles the HOW):
  SELECT * FROM users WHERE status = 'active' ORDER BY name;
  ← You declare the goal. MySQL figures out the optimal execution plan.
```

> 💡 **SQL is the universal language of data.** Once you know SQL, you can work with MySQL, PostgreSQL, SQLite, Oracle, SQL Server, and MariaDB — they all use SQL (with minor variations). Learning SQL is one of the highest-return-on-investment skills in software development.

---

## A Brief History of SQL

Understanding where SQL came from helps you understand why it works the way it does.

```
1970 — Edgar F. Codd at IBM publishes "A Relational Model of Data for Large
       Shared Data Banks" — the theoretical foundation of relational databases.

1974 — IBM develops SEQUEL (Structured English Query Language) based on
       Codd's model. Later renamed SQL due to trademark issues.

1979 — Oracle Corporation releases the first commercially available SQL
       relational database (Oracle V2).

1986 — SQL becomes an ANSI (American National Standards Institute) standard.
       SQL-86 (also called SQL1) is the first official standard.

1992 — SQL-92 (SQL2) — major expansion, widely supported, still the baseline
       most databases implement.

1999 — SQL:1999 (SQL3) — adds recursive queries, triggers, object-relational
       features.

2003 — SQL:2003 — adds XML features, window functions (OVER, PARTITION BY).

2011 — SQL:2011 — temporal data (period-sensitive queries).

2023 — SQL:2023 — adds JSON features, graph tables, and more.

Today — MySQL, PostgreSQL, SQLite, Oracle, SQL Server all implement the core
        SQL standard with their own extensions and variations.
```

---

## How SQL Works

When you write and execute an SQL statement, MySQL processes it through several internal steps:

```
You write:
  SELECT name, email FROM users WHERE status = 'active' ORDER BY name LIMIT 10;

MySQL processes it:

  STEP 1 — PARSING
    MySQL reads your SQL text and checks for syntax errors.
    Breaks it into tokens: SELECT, name, email, FROM, users, WHERE, etc.
    Builds an internal parse tree (like an AST in PHP).

  STEP 2 — PREPROCESSING
    Checks that tables and columns exist.
    Checks your permissions (can you SELECT from users?).

  STEP 3 — OPTIMIZATION (The Query Optimizer)
    MySQL's smartest component.
    Looks at available indexes on users table.
    Calculates multiple execution strategies.
    Picks the CHEAPEST (fastest) plan:
      "Use the index on status to find active users,
       then sort by name, then return first 10."

  STEP 4 — EXECUTION
    The storage engine (InnoDB) reads data from disk or buffer pool.
    Applies the WHERE filter.
    Sorts the results.
    Returns the first 10 rows.

  STEP 5 — RESULT RETURNED
    MySQL sends the rows back to your PHP code.

Total time: 0.002 seconds (2 milliseconds) for millions of rows.
```

---

## SQL Sublanguages — The Four Categories

SQL is not one thing — it's actually divided into sub-languages, each for a different purpose. Understanding these categories makes it much easier to organize what you're learning.

---

### DDL — Data Definition Language

- **DDL** statements define and modify the **structure** of your database — creating, altering, and dropping tables, databases, indexes, and constraints.
- DDL changes are **auto-committed** — they cannot be rolled back (in most databases).
- These are the statements that change *what exists*, not *what data is in it*.

```
DDL Keywords: CREATE, ALTER, DROP, TRUNCATE, RENAME

CREATE DATABASE  → Make a new database
CREATE TABLE     → Define a new table and its columns
ALTER TABLE      → Modify an existing table (add/drop columns, change types)
DROP TABLE       → Delete a table and all its data
DROP DATABASE    → Delete an entire database
TRUNCATE TABLE   → Delete all rows (faster than DELETE, resets AUTO_INCREMENT)
RENAME TABLE     → Rename a table
CREATE INDEX     → Add an index to speed up queries
DROP INDEX       → Remove an index
```

```sql
-- DDL Examples

-- Create a database
CREATE DATABASE blog_app
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

-- Create a table
CREATE TABLE articles (
  id         INT UNSIGNED NOT NULL AUTO_INCREMENT,
  title      VARCHAR(255) NOT NULL,
  content    LONGTEXT     NOT NULL,
  author_id  INT UNSIGNED NOT NULL,
  published  TINYINT(1)   NOT NULL DEFAULT 0,
  created_at TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id)
);

-- Add a new column to an existing table
ALTER TABLE articles ADD COLUMN views INT UNSIGNED NOT NULL DEFAULT 0 AFTER published;

-- Change a column's type or properties
ALTER TABLE articles MODIFY COLUMN title VARCHAR(500) NOT NULL;

-- Rename a column
ALTER TABLE articles RENAME COLUMN published TO is_published;

-- Drop a column
ALTER TABLE articles DROP COLUMN views;

-- Add an index
CREATE INDEX articles_author_idx ON articles (author_id);

-- Rename a table
RENAME TABLE articles TO blog_posts;

-- Drop a table
DROP TABLE IF EXISTS blog_posts;

-- Truncate a table (empty it completely, reset AUTO_INCREMENT)
TRUNCATE TABLE articles;
```

---

### DML — Data Manipulation Language

- **DML** statements deal with the **data inside tables** — inserting, updating, and deleting rows.
- DML changes CAN be rolled back inside a transaction.
- These are the statements you use most in your PHP application code.

```
DML Keywords: INSERT, UPDATE, DELETE

INSERT → Add new rows to a table
UPDATE → Modify existing rows
DELETE → Remove rows from a table
```

```sql
-- DML Examples

-- INSERT — add a new row
INSERT INTO users (name, email, password, role)
VALUES ('Phyo Min Paing', 'phyo@example.com', '$2y$10$hashed', 'user');

-- INSERT multiple rows at once
INSERT INTO users (name, email, password, role)
VALUES
  ('Alice', 'alice@example.com', '$2y$10$hash2', 'admin'),
  ('Bob',   'bob@example.com',   '$2y$10$hash3', 'user');

-- UPDATE — modify existing rows
UPDATE users SET status = 'active' WHERE id = 1;

-- UPDATE multiple columns
UPDATE users
SET name = 'Phyo Updated', email = 'new@example.com', updated_at = NOW()
WHERE id = 1;

-- DELETE — remove rows
DELETE FROM users WHERE id = 5;
DELETE FROM users WHERE status = 'banned' AND created_at < '2020-01-01';
```

---

### DQL — Data Query Language

- **DQL** is technically part of DML in many references, but it deserves its own category because it's the most important and most used statement in SQL.
- There is only ONE DQL statement: **SELECT**.
- SELECT retrieves data without modifying anything.

```
DQL Keyword: SELECT
```

```sql
-- DQL Examples — The SELECT statement

-- Select all columns from a table
SELECT * FROM users;

-- Select specific columns
SELECT id, name, email FROM users;

-- Filter with WHERE
SELECT * FROM users WHERE status = 'active';

-- Sort results
SELECT * FROM users ORDER BY name ASC;

-- Limit results
SELECT * FROM users LIMIT 10;

-- Aggregate functions
SELECT COUNT(*) FROM users;
SELECT AVG(price) FROM products;
SELECT SUM(total) FROM orders WHERE status = 'delivered';
SELECT MAX(created_at) FROM users;
SELECT MIN(price) FROM products;

-- Join multiple tables
SELECT u.name, o.total, o.status
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.status = 'active';

-- Group and aggregate
SELECT role, COUNT(*) AS user_count
FROM users
GROUP BY role;
```

---

### DCL — Data Control Language

- **DCL** statements manage **user permissions and access control** — who can do what to which database objects.
- These are typically run by database administrators, not by application code.

```
DCL Keywords: GRANT, REVOKE

GRANT  → Give a user permission to do something
REVOKE → Take away a permission
```

```sql
-- DCL Examples

-- Grant permissions to a user
GRANT SELECT, INSERT, UPDATE, DELETE ON myapp.* TO 'app_user'@'localhost';

-- Grant all privileges
GRANT ALL PRIVILEGES ON myapp.* TO 'admin_user'@'localhost';

-- Grant with ability to pass permissions to others
GRANT SELECT ON myapp.* TO 'reader'@'localhost' WITH GRANT OPTION;

-- Revoke a permission
REVOKE DELETE ON myapp.* FROM 'app_user'@'localhost';

-- Revoke all permissions
REVOKE ALL PRIVILEGES ON myapp.* FROM 'app_user'@'localhost';

-- Apply privilege changes
FLUSH PRIVILEGES;
```

---

### TCL — Transaction Control Language

- **TCL** manages **transactions** — groups of SQL statements that must all succeed or all fail together (ACID atomicity).
- Essential for financial operations, inventory management, any multi-step operation that must be atomic.

```
TCL Keywords: BEGIN / START TRANSACTION, COMMIT, ROLLBACK, SAVEPOINT

BEGIN / START TRANSACTION → Start a transaction block
COMMIT                    → Save all changes made in this transaction
ROLLBACK                  → Undo all changes back to before the transaction started
SAVEPOINT name            → Mark a point you can ROLLBACK TO (partial rollback)
ROLLBACK TO SAVEPOINT     → Undo back to a specific savepoint
```

```sql
-- TCL Example — Bank Transfer

START TRANSACTION;

  -- Deduct from sender
  UPDATE accounts SET balance = balance - 500 WHERE id = 1;

  -- Credit to receiver
  UPDATE accounts SET balance = balance + 500 WHERE id = 2;

-- If both succeeded:
COMMIT;
-- Changes are now permanent

-- If anything failed:
-- ROLLBACK;
-- Everything is undone — no money lost

-- ─────────────────────────────────────────────────────────────

-- SAVEPOINT example
START TRANSACTION;

  INSERT INTO orders (user_id, total) VALUES (1, 99.99);
  SAVEPOINT order_created;    -- mark this point

  INSERT INTO order_items (order_id, product_id, qty) VALUES (LAST_INSERT_ID(), 5, 2);
  -- Oh no — product 5 is out of stock!

ROLLBACK TO SAVEPOINT order_created;   -- undo the order_items insert only
                                         -- the order itself is still in the transaction
COMMIT;   -- commit just the order (without items)
```

---

### SQL Sublanguages at a Glance

| Sublanguage | Full Name | Commands | Purpose |
|---|---|---|---|
| **DDL** | Data Definition Language | `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME` | Define/modify structure |
| **DML** | Data Manipulation Language | `INSERT`, `UPDATE`, `DELETE` | Modify data |
| **DQL** | Data Query Language | `SELECT` | Read data |
| **DCL** | Data Control Language | `GRANT`, `REVOKE` | Manage permissions |
| **TCL** | Transaction Control Language | `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT` | Manage transactions |

---

## SQL Data Types in MySQL

MySQL has a rich set of data types organized into categories. Choosing the right data type for each column is crucial — it affects storage size, performance, what values are valid, and what operations you can perform.

---

### Numeric Types

#### Integer Types

Every integer type has a **signed** (positive and negative) and **unsigned** (positive only) range. Always use `UNSIGNED` for columns that can never be negative (IDs, counts, ages, prices in cents).

| Type | Storage | Signed Min | Signed Max | Unsigned Min | Unsigned Max |
|---|---|---|---|---|---|
| `TINYINT` | 1 byte | -128 | 127 | 0 | 255 |
| `SMALLINT` | 2 bytes | -32,768 | 32,767 | 0 | 65,535 |
| `MEDIUMINT` | 3 bytes | -8,388,608 | 8,388,607 | 0 | 16,777,215 |
| `INT` | 4 bytes | -2,147,483,648 | 2,147,483,647 | 0 | 4,294,967,295 |
| `BIGINT` | 8 bytes | -9.2 × 10¹⁸ | 9.2 × 10¹⁸ | 0 | 1.8 × 10¹⁹ |

```sql
-- Choosing the right integer type

-- TINYINT — small numbers, booleans
age          TINYINT UNSIGNED           -- 0-255, enough for any age
rating       TINYINT UNSIGNED           -- 1-5 stars
is_active    TINYINT(1) NOT NULL DEFAULT 1  -- boolean: 0 or 1
priority     TINYINT UNSIGNED           -- 0-255 priority levels

-- SMALLINT — medium small numbers
year_built   SMALLINT UNSIGNED          -- 1901-2155 (years)
port_number  SMALLINT UNSIGNED          -- 0-65535 (network ports)

-- INT — most common for IDs and counts
id           INT UNSIGNED AUTO_INCREMENT PRIMARY KEY  -- up to 4.3 billion rows
user_id      INT UNSIGNED NOT NULL                    -- foreign key
views        INT UNSIGNED NOT NULL DEFAULT 0          -- page view counter
quantity     INT UNSIGNED NOT NULL DEFAULT 0          -- product stock

-- BIGINT — huge numbers
twitter_id   BIGINT UNSIGNED            -- Twitter-scale IDs
revenue_cents BIGINT NOT NULL DEFAULT 0  -- store money as cents to avoid decimals
file_size    BIGINT UNSIGNED            -- file size in bytes
```

#### Decimal / Float Types

| Type | Storage | Precision | Use For |
|---|---|---|---|
| `DECIMAL(M,D)` | Variable | Exact | Money, financial data |
| `NUMERIC(M,D)` | Variable | Exact | Same as DECIMAL (alias) |
| `FLOAT` | 4 bytes | ~7 digits (approximate) | Scientific, GPS |
| `DOUBLE` | 8 bytes | ~15 digits (approximate) | High-precision science |

```sql
-- DECIMAL — exact arithmetic, use for money ALWAYS
price        DECIMAL(10, 2) NOT NULL DEFAULT 0.00  -- up to 99,999,999.99
tax_rate     DECIMAL(5, 4) NOT NULL DEFAULT 0.0000  -- e.g. 0.0900 = 9.00%
discount     DECIMAL(5, 2) NOT NULL DEFAULT 0.00    -- e.g. 25.00 = 25%
weight       DECIMAL(8, 3) NOT NULL DEFAULT 0.000   -- e.g. 1.250 kg

-- Understanding DECIMAL(M, D):
-- M = total number of digits (both sides of decimal)
-- D = digits after the decimal point
-- DECIMAL(10, 2) → 10 total digits, 2 after decimal
--   Max value: 99,999,999.99 (8 before decimal + 2 after = 10 total)

-- FLOAT / DOUBLE — approximate, use for non-financial decimals
latitude     FLOAT NOT NULL DEFAULT 0          -- GPS: -90.0 to 90.0
longitude    FLOAT NOT NULL DEFAULT 0          -- GPS: -180.0 to 180.0
temperature  FLOAT                              -- sensor reading
score        DOUBLE                             -- high-precision calculation
```

> ⚠️ **NEVER use FLOAT or DOUBLE for money.** `0.1 + 0.2` in a FLOAT = `0.30000000000000004`. Use `DECIMAL(10,2)` for prices. If you need integer-only math, store prices in cents as `BIGINT`: $19.99 → 1999.

---

### String Types

#### Fixed vs Variable Length

| Type | Max Length | Storage | Description |
|---|---|---|---|
| `CHAR(n)` | 255 chars | Fixed (always n bytes) | Fixed-length string — padded with spaces |
| `VARCHAR(n)` | 65,535 bytes | Variable (actual length + 1-2 bytes) | Most common — variable-length text |
| `BINARY(n)` | 255 bytes | Fixed | Fixed-length binary data |
| `VARBINARY(n)` | 65,535 bytes | Variable | Variable-length binary data |

```sql
-- CHAR — use for truly fixed-length data
country_code CHAR(2)  NOT NULL              -- 'MM', 'US', 'GB' — always 2 chars
currency     CHAR(3)  NOT NULL DEFAULT 'USD' -- 'USD', 'EUR', 'MMK'
status_code  CHAR(10) NOT NULL               -- short fixed codes
uuid         CHAR(36) NOT NULL               -- '550e8400-e29b-41d4-a716-446655440000'

-- VARCHAR — use for most text columns
name         VARCHAR(100)  NOT NULL
email        VARCHAR(255)  NOT NULL
url          VARCHAR(2083) NOT NULL          -- max URL length
title        VARCHAR(255)  NOT NULL
slug         VARCHAR(255)  NOT NULL          -- URL slug: my-product-name
phone        VARCHAR(20)   NULL              -- +95 9 123 456 789
ip_address   VARCHAR(45)   NULL              -- IPv4 (15 chars) or IPv6 (45 chars)
```

#### Text Types — For Long Content

| Type | Max Size | Use For |
|---|---|---|
| `TINYTEXT` | 255 bytes | Very short text (rarely used — use VARCHAR) |
| `TEXT` | 65,535 bytes (~64 KB) | Medium text: descriptions, comments |
| `MEDIUMTEXT` | 16,777,215 bytes (~16 MB) | Long articles, large content |
| `LONGTEXT` | 4,294,967,295 bytes (~4 GB) | Huge text: books, very large documents |

```sql
-- TEXT types — for content longer than ~500 characters
bio          TEXT NULL                       -- user biography
description  TEXT NULL                       -- product description
content      LONGTEXT NOT NULL               -- blog post / article content
notes        TEXT NULL                       -- order notes, admin notes
meta_data    MEDIUMTEXT NULL                 -- serialized data

-- VARCHAR vs TEXT decision:
-- VARCHAR: can be indexed, can have DEFAULT value, stored in row
-- TEXT: larger capacity, cannot be indexed directly (needs prefix index),
--       no DEFAULT value, stored separately from row data
```

---

### Date and Time Types

| Type | Format | Range | Size | Use For |
|---|---|---|---|---|
| `DATE` | `YYYY-MM-DD` | 1000-01-01 to 9999-12-31 | 3 bytes | Date only — birthdays, deadlines |
| `TIME` | `HH:MM:SS` | -838:59:59 to 838:59:59 | 3 bytes | Duration or time of day |
| `YEAR` | `YYYY` | 1901 to 2155 | 1 byte | Year only |
| `DATETIME` | `YYYY-MM-DD HH:MM:SS` | 1000-01-01 to 9999-12-31 | 8 bytes | Specific moment, no timezone conversion |
| `TIMESTAMP` | `YYYY-MM-DD HH:MM:SS` | 1970-01-01 to 2038-01-19 | 4 bytes | Auto timestamps, timezone-aware |

```sql
-- DATE — date with no time component
birth_date    DATE NOT NULL              -- 1995-08-15
expiry_date   DATE NULL                  -- subscription expiry
event_date    DATE NOT NULL              -- calendar event

-- DATETIME — timezone-independent datetime
published_at  DATETIME NULL DEFAULT NULL  -- article publish schedule
scheduled_at  DATETIME NULL              -- appointment or job schedule
deleted_at    DATETIME NULL DEFAULT NULL  -- soft delete

-- TIMESTAMP — auto-managed, timezone-aware
-- These are the two most important columns in any table:
created_at    TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
updated_at    TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP

-- TIME — duration or time-only
duration      TIME NOT NULL    -- video duration: '01:30:00'
open_time     TIME NOT NULL    -- store opening: '09:00:00'

-- YEAR
model_year    YEAR NULL        -- car model year: 2024
```

### TIMESTAMP vs DATETIME — The Critical Difference

```
TIMESTAMP:
  - Stores UTC internally, converts to/from session timezone automatically
  - Range: 1970-01-01 00:00:01 UTC to 2038-01-19 03:14:07 UTC
  - Supports DEFAULT CURRENT_TIMESTAMP and ON UPDATE CURRENT_TIMESTAMP
  - 4 bytes (smaller than DATETIME)
  - Use for: created_at, updated_at (record timestamps)

DATETIME:
  - Stores exactly what you insert — no timezone conversion
  - Range: 1000-01-01 to 9999-12-31
  - Does not support ON UPDATE CURRENT_TIMESTAMP in older MySQL
  - 8 bytes (larger than TIMESTAMP)
  - Use for: scheduled events, appointments, published dates
    (where you want to preserve the exact local time as entered)

Practical rule:
  Use TIMESTAMP for automatic "when did this record change?" tracking.
  Use DATETIME when you're storing a specific scheduled moment in time.
```

---

### Binary Types

Used for storing raw binary data — images, files, encrypted values, etc. In practice, **don't store files in the database** — store them on disk or a cloud service (S3) and save the file path in a VARCHAR column.

| Type | Max Size | Use Case |
|---|---|---|
| `TINYBLOB` | 255 bytes | Tiny binary data |
| `BLOB` | 65,535 bytes | Small binary (small images) |
| `MEDIUMBLOB` | 16 MB | Medium files |
| `LONGBLOB` | 4 GB | Large files |

```sql
-- When you DO need binary storage:
thumbnail    BLOB NULL          -- small image thumbnail
public_key   BLOB NOT NULL      -- RSA public key
encrypted    VARBINARY(255) NULL -- small encrypted value

-- Best practice: don't store files in DB
avatar       VARCHAR(500) NULL   -- store the FILE PATH, not the file itself
-- e.g., '/uploads/avatars/user-1-photo.jpg'
-- actual file lives on disk or Amazon S3
```

---

### Special Types

#### ENUM

```sql
-- ENUM — a column that can only hold one value from a predefined list
-- Stored as an integer internally (1, 2, 3...) — very compact and fast
status   ENUM('active','inactive','banned') NOT NULL DEFAULT 'active'
role     ENUM('user','admin','editor','moderator') NOT NULL DEFAULT 'user'
gender   ENUM('male','female','other','unspecified') NULL
priority ENUM('low','medium','high','urgent') NOT NULL DEFAULT 'medium'
plan     ENUM('free','basic','pro','enterprise') NOT NULL DEFAULT 'free'

-- ENUM pros: fast, compact, self-documenting
-- ENUM cons: adding new values requires ALTER TABLE
--            (can lock the table in old MySQL, but fast in MySQL 8+ with INSTANT algorithm)
```

#### SET

```sql
-- SET — a column that can hold MULTIPLE values from a predefined list
-- Stored as a bitmask integer — very compact
permissions  SET('read','write','delete','publish','admin') NOT NULL DEFAULT 'read'
weekdays     SET('Mon','Tue','Wed','Thu','Fri','Sat','Sun') NOT NULL
interests    SET('php','mysql','nginx','redis','docker')

-- Example: a user with read+write permissions:
-- permissions = 'read,write'
-- Stored internally as a bitmask: 00000011 = 3

-- Less commonly used than ENUM — most apps use separate permission tables instead
```

#### JSON

```sql
-- JSON — store JSON data natively (MySQL 5.7.8+)
-- Indexed, validated, queryable with JSON functions
settings     JSON NULL             -- user preferences
metadata     JSON NULL             -- flexible extra data
address      JSON NULL             -- {"street":"Main St","city":"Yangon"}
tags         JSON NULL             -- ["php","mysql","backend"]
attributes   JSON NOT NULL DEFAULT (JSON_OBJECT())  -- product attributes

-- Querying JSON columns:
SELECT name, settings->>'$.theme' AS theme FROM users;
SELECT * FROM products WHERE attributes->>'$.color' = 'red';
SELECT * FROM users WHERE JSON_CONTAINS(tags, '"php"');
```

#### BOOLEAN

```sql
-- BOOLEAN / BOOL is just an alias for TINYINT(1)
-- 0 = false, 1 = true (PHP PDO returns these as 0/1 integers)
is_active    BOOLEAN NOT NULL DEFAULT TRUE   -- same as TINYINT(1) DEFAULT 1
is_verified  BOOLEAN NOT NULL DEFAULT FALSE  -- same as TINYINT(1) DEFAULT 0
is_featured  TINYINT(1) NOT NULL DEFAULT 0   -- identical to BOOLEAN

-- In practice, most MySQL developers use TINYINT(1) directly
-- PHP reads TINYINT(1) as 0 or 1 (not true/false)
-- Cast to bool in PHP: (bool) $row['is_active']
```

---

### MySQL Data Types Quick Reference

| Category | Type | Size | Use For |
|---|---|---|---|
| **Integer** | `TINYINT` | 1 byte | Booleans, ratings, small flags |
| | `SMALLINT` | 2 bytes | Years, small counters |
| | `INT` | 4 bytes | Most IDs and counts ← most common |
| | `BIGINT` | 8 bytes | Large IDs, social-media scale |
| **Decimal** | `DECIMAL(10,2)` | Variable | Money, prices ← always for money |
| | `FLOAT` | 4 bytes | GPS, scientific (approximate) |
| **String** | `CHAR(n)` | Fixed n bytes | Fixed-length codes (country, currency) |
| | `VARCHAR(n)` | Variable | Names, emails, URLs ← most common |
| | `TEXT` | Up to 64KB | Descriptions, comments, posts |
| | `LONGTEXT` | Up to 4GB | Article content, large documents |
| **Date/Time** | `DATE` | 3 bytes | Birthdays, deadlines (date only) |
| | `DATETIME` | 8 bytes | Scheduled moments, appointments |
| | `TIMESTAMP` | 4 bytes | created_at, updated_at ← most common |
| **Special** | `ENUM('a','b')` | 1-2 bytes | Status, role, fixed options |
| | `JSON` | Variable | Flexible structured data |
| | `TINYINT(1)` | 1 byte | Boolean (true/false) |

---

## SQL Syntax Rules

SQL has strict syntax rules. Breaking them causes errors. Understanding these rules helps you debug problems faster.

```sql
-- RULE 1: SQL is NOT case-sensitive for keywords
-- These three are identical:
SELECT * FROM users WHERE id = 1;
select * from users where id = 1;
Select * From Users Where Id = 1;

-- CONVENTION: Uppercase SQL keywords, lowercase identifiers
-- This makes SQL much easier to read
SELECT id, name, email    -- SQL keywords UPPERCASE
FROM users                -- table name lowercase
WHERE status = 'active';  -- column name lowercase

-- ─────────────────────────────────────────────────────────────────────

-- RULE 2: Statements end with a semicolon
SELECT * FROM users;   ← semicolon required in multi-statement scripts
                          (optional in single-statement command-line usage)

-- RULE 3: String values use SINGLE quotes (not double quotes)
WHERE name = 'Phyo'      ✅ correct
WHERE name = "Phyo"      ❌ syntax error in standard SQL
                             (MySQL accepts double quotes in some modes, but don't rely on it)

-- RULE 4: Identifiers (table/column names) use backticks in MySQL
SELECT `name`, `email` FROM `users`;  ← backticks are optional unless
                                         the identifier is a reserved word
SELECT name, email FROM users;        ← backticks optional when no reserved words

-- When are backticks REQUIRED?
SELECT `order`, `select`, `group` FROM `table`;  -- reserved words as column names!
-- Best practice: avoid naming columns after reserved words

-- RULE 5: NULL comparisons use IS NULL, not = NULL
WHERE deleted_at IS NULL       ✅ correct
WHERE deleted_at = NULL        ❌ ALWAYS returns false — NULL is not equal to anything

WHERE deleted_at IS NOT NULL   ✅ correct
WHERE deleted_at != NULL       ❌ ALWAYS returns false

-- RULE 6: String comparison is case-insensitive by default (with utf8mb4_unicode_ci)
WHERE name = 'phyo'    -- matches 'Phyo', 'PHYO', 'phyo' (all the same)
WHERE name = 'PHYO'    -- same result

-- For case-sensitive comparison:
WHERE BINARY name = 'Phyo'    -- ← only matches 'Phyo', not 'phyo' or 'PHYO'

-- RULE 7: Comments (covered in next section)
```

---

## SQL Comments

Comments are text in your SQL that MySQL ignores — useful for documentation, explaining complex queries, and temporarily disabling code.

```sql
-- This is a single-line comment (MySQL and SQL standard)
# This is also a single-line comment (MySQL specific)
/* This is a multi-line comment
   that spans multiple lines
   and is the SQL standard format */

-- Practical examples:
-- Get all active admin users
SELECT id, name, email
FROM users
WHERE status = 'active'   -- only active accounts
  AND role = 'admin'      -- only admins
ORDER BY name ASC;        -- alphabetical

/*
  This query calculates total revenue per user
  for the reporting dashboard.
  Author: Phyo Min Paing
  Date: 2026-06-28
*/
SELECT
  u.name,
  COUNT(o.id) AS order_count,
  SUM(o.total) AS total_revenue
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.status = 'delivered'
GROUP BY u.id, u.name
ORDER BY total_revenue DESC;
```

---

## Naming Conventions

Consistent naming makes SQL code readable and maintainable. These are the widely accepted conventions.

```
TABLE NAMES:
  ✅ Use lowercase, plural nouns
  ✅ Use underscores for multiple words (snake_case)
  ✅ Be descriptive

  Good:    users, products, order_items, blog_posts, user_sessions
  Bad:     User, USERS, OrderItems, tbl_user, u, usr

COLUMN NAMES:
  ✅ Use lowercase, singular, descriptive
  ✅ Use underscores (snake_case)

  Good:    id, first_name, email_address, created_at, is_active, user_id
  Bad:     ID, FirstName, EmailAddress, createdAt, isActive, UserId

FOREIGN KEY COLUMNS:
  ✅ Convention: referenced_table_singular + "_id"

  user_id      → references users.id
  product_id   → references products.id
  category_id  → references categories.id
  parent_id    → self-referencing parent in same table

PRIMARY KEY:
  ✅ Always name it "id"
  Never: uid, userid, user_id (confusing as primary key)

INDEX NAMES:
  ✅ Convention: tablename_columnname_idx  OR  tablename_columnname_index
  ✅ For unique indexes: tablename_columnname_unique

  users_email_unique
  orders_user_id_index
  products_category_id_index
  articles_title_fulltext

DATABASE NAMES:
  ✅ Lowercase, underscore-separated, descriptive

  Good:    myshop_db, blog_production, user_management
  Bad:     MyShop, "My Shop DB", myshop

SPECIAL COLUMN NAMES (standard across all projects):
  id           → Primary key (always)
  created_at   → When the record was created
  updated_at   → When the record was last modified
  deleted_at   → Soft delete timestamp (NULL = not deleted)
  is_*         → Boolean flags: is_active, is_verified, is_featured
  *_id         → Foreign key: user_id, product_id, category_id
  *_at         → Timestamp: created_at, updated_at, published_at
  *_count      → Denormalized counter: views_count, comments_count
```

---

## MySQL vs SQL Standard

MySQL mostly follows the SQL standard but has its own extensions and quirks. These are the differences you'll encounter:

| Feature | SQL Standard | MySQL Behavior |
|---|---|---|
| String quotes | Single quotes only `'hello'` | Both `'hello'` and `"hello"` (mode-dependent) |
| Identifier quotes | Double quotes `"column"` | Backticks `` `column` `` |
| `AUTO_INCREMENT` | `GENERATED ALWAYS AS IDENTITY` | `AUTO_INCREMENT` (MySQL extension) |
| Boolean | `BOOLEAN` | `TINYINT(1)` (alias only) |
| `LIMIT` | `FETCH FIRST n ROWS ONLY` | `LIMIT n` (MySQL extension, adopted widely) |
| `ENUM` | Not in standard | MySQL extension (and PostgreSQL) |
| Division by zero | Error | Returns NULL (with warning) |
| GROUP BY | Must include all non-aggregate columns | MySQL's `ONLY_FULL_GROUP_BY` mode (strict) |

> 💡 **For your learning:** Don't worry too much about the standard vs MySQL differences. Learn MySQL-specific SQL and note the differences as you encounter them. MySQL syntax works across MySQL, MariaDB, and is very similar to PostgreSQL and SQLite.

---

## Quick Revision

- **SQL** (Structured Query Language) is the standardized language for communicating with relational databases. It is **declarative** — you say *what* you want, the database engine figures out *how* to get it.
- SQL was standardized in 1986. MySQL, PostgreSQL, SQLite, Oracle, and SQL Server all implement SQL with their own extensions.
- **DDL** (`CREATE`, `ALTER`, `DROP`, `TRUNCATE`) — defines database structure. Auto-committed.
- **DML** (`INSERT`, `UPDATE`, `DELETE`) — manipulates data inside tables. Can be rolled back.
- **DQL** (`SELECT`) — reads data without modifying it. The most-used SQL statement.
- **DCL** (`GRANT`, `REVOKE`) — manages user permissions.
- **TCL** (`BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`) — manages transactions and ACID guarantees.
- **Integer types:** `TINYINT` (1 byte), `SMALLINT` (2), `INT` (4), `BIGINT` (8). Always use `UNSIGNED` for IDs and non-negative numbers.
- **Decimal types:** `DECIMAL(10,2)` for money (exact), `FLOAT`/`DOUBLE` for scientific data (approximate). NEVER use FLOAT for money.
- **String types:** `CHAR(n)` for fixed-length codes, `VARCHAR(n)` for most text (emails, names, titles), `TEXT` for long content, `LONGTEXT` for articles.
- **Date/Time types:** `DATE` (date only), `TIMESTAMP` (timezone-aware, for `created_at`/`updated_at`), `DATETIME` (no timezone, for scheduled moments).
- **Special types:** `ENUM('a','b')` for fixed value sets (status, role), `TINYINT(1)` for booleans, `JSON` for flexible structured data.
- **SQL keywords are case-insensitive** — convention is UPPERCASE keywords, lowercase identifiers.
- **String values use single quotes** `'value'` — double quotes work in MySQL but are non-standard.
- **NULL comparisons use `IS NULL` / `IS NOT NULL`** — `= NULL` always returns false.
- **Naming conventions:** `snake_case` for everything, plural table names (`users`), `id` as primary key, `user_id` as foreign key, `created_at`/`updated_at` as timestamps.