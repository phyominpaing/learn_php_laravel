# MySQL — Creating Tables & Defining Columns

A **table** is where your actual data lives — every user account, every product, every order is stored as a row inside a table. Designing a table correctly from the start is one of the most important skills in backend development. A poorly designed table causes bugs, performance issues, and painful migrations later.

---

## Table of Contents

1. [What is a Table?](#what-is-a-table)
2. [Planning a Table Before Creating It](#planning-a-table-before-creating-it)
3. [Creating a Table in phpMyAdmin — Step by Step](#creating-a-table-in-phpmyadmin--step-by-step)
4. [Understanding Every Column Option](#understanding-every-column-option)
   - [Column Name](#column-name)
   - [Data Type](#data-type)
   - [Length / Values](#length--values)
   - [Default Value](#default-value)
   - [NULL vs NOT NULL](#null-vs-not-null)
   - [Index / Primary Key](#index--primary-key)
   - [AUTO_INCREMENT](#auto_increment)
   - [Attributes](#attributes)
   - [Comments](#comments)
5. [MySQL Data Types — Deep Dive](#mysql-data-types--deep-dive)
   - [Integer Types](#integer-types)
   - [Decimal / Float Types](#decimal--float-types)
   - [String Types](#string-types)
   - [Date & Time Types](#date--time-types)
   - [Other Types](#other-types)
6. [The Essential Columns Every Table Should Have](#the-essential-columns-every-table-should-have)
7. [Creating Real-World Tables — Full Examples](#creating-real-world-tables--full-examples)
   - [Users Table](#users-table)
   - [Products Table](#products-table)
   - [Orders Table](#orders-table)
   - [Categories Table](#categories-table)
8. [Viewing and Modifying Table Structure](#viewing-and-modifying-table-structure)
9. [Common Mistakes When Creating Tables](#common-mistakes-when-creating-tables)
10. [Quick Revision](#quick-revision)

---

## What is a Table?

- A **table** is a structured container that stores data as **rows** (records) and **columns** (fields).
- Every column has a fixed **name** and **data type** — the data type defines what kind of information that column can hold.
- Think of a table like a spreadsheet — but stricter, faster, and with rules enforced by MySQL.

```
users table:
┌────┬──────────────────┬──────────────────────┬───────┬─────────────────────┐
│ id │ name             │ email                │ age   │ created_at          │
├────┼──────────────────┼──────────────────────┼───────┼─────────────────────┤
│  1 │ Phyo Min Paing   │ phyo@example.com     │  25   │ 2026-01-15 09:30:00 │
│  2 │ Alice            │ alice@example.com    │  30   │ 2026-02-20 14:00:00 │
│  3 │ Bob              │ bob@example.com      │  22   │ 2026-03-05 11:15:00 │
└────┴──────────────────┴──────────────────────┴───────┴─────────────────────┘
  ↑         ↑                   ↑                 ↑              ↑
  INT    VARCHAR(100)       VARCHAR(255)        TINYINT      TIMESTAMP
(Primary Key                                  (0 to 255)  (auto-filled)
 AUTO_INCREMENT)
```

---

## Planning a Table Before Creating It

Before you open phpMyAdmin, always **plan your table on paper first**. Changing a table's design after data is in it is painful — new columns need migrations, changing types can corrupt data.

Ask yourself these questions:

```
1. What is this table storing?
   → "Users of my application"

2. What pieces of information do I need for each record?
   → name, email, password, phone, city, status, role, created_at

3. What data type fits each piece of information?
   → name: VARCHAR(100)
   → email: VARCHAR(255) and UNIQUE
   → password: VARCHAR(255) (stores a hashed password)
   → phone: VARCHAR(20) (phone numbers have +, -, () so not INT!)
   → city: VARCHAR(100)
   → status: ENUM('active','inactive','banned')
   → role: ENUM('user','admin','editor')
   → created_at: TIMESTAMP with DEFAULT CURRENT_TIMESTAMP

4. Which column uniquely identifies each row?
   → id (INT, PRIMARY KEY, AUTO_INCREMENT)

5. Are there any columns that must be unique across all rows?
   → email (add UNIQUE index)

6. Are there any columns that reference another table?
   → Not yet for users, but orders will have user_id that links here
```

---

## Creating a Table in phpMyAdmin — Step by Step

```
STEP 1: Open phpMyAdmin → http://localhost/phpmyadmin

STEP 2: Click your database in the LEFT PANEL
  (e.g., click "myshop_db")

STEP 3: In the RIGHT PANEL, find the "Create table" section:
  ┌──────────────────────────────────────────────────┐
  │  Create table                                    │
  │  Name:              [ users            ]         │
  │  Number of columns: [ 7  ]                       │
  │                              [ Go ]              │
  └──────────────────────────────────────────────────┘

  - Name: Enter "users" (use lowercase + underscores)
  - Number of columns: Enter how many columns you need
    (You can always add more later — start with your planned count)
  - Click "Go"

STEP 4: The column definition form appears
  Each row in this form = one column in your table.
  You'll see columns for Name, Type, Length, Default, etc.

STEP 5: Fill in each column (covered in full detail below)

STEP 6: Scroll to the bottom → Look for the Table Options:
  ┌────────────────────────────────────────┐
  │  Table options                         │
  │  Storage Engine:  [ InnoDB ▼ ]        │ ← Always keep InnoDB
  │  Collation: [ utf8mb4_unicode_ci ▼ ]  │ ← Always set this
  │  Comments: [                    ]      │ ← Optional description
  └────────────────────────────────────────┘

STEP 7: Click "Save" at the bottom

  ✅ Table 'users' has been created.
```

---

## Understanding Every Column Option

When you define a column in phpMyAdmin, you see many options. Here is what every single one means.

```
The column definition form in phpMyAdmin:
┌──────────┬──────────────┬────────┬──────────┬─────┬─────────┬────────────┬──────────┬──────────┐
│ Name     │ Type         │ Length │ Default  │ Null│ Index   │ A_I        │ Attribute│ Comments │
├──────────┼──────────────┼────────┼──────────┼─────┼─────────┼────────────┼──────────┼──────────┤
│ id       │ INT          │        │ None     │ [ ] │ PRIMARY │ [✓]        │ UNSIGNED │          │
│ name     │ VARCHAR      │ 100    │ None     │ [ ] │         │            │          │          │
│ email    │ VARCHAR      │ 255    │ None     │ [ ] │ UNIQUE  │            │          │ Login    │
│ password │ VARCHAR      │ 255    │ None     │ [ ] │         │            │          │          │
│ status   │ ENUM         │active..│ active   │ [ ] │         │            │          │          │
│ role     │ ENUM         │user,...│ user     │ [ ] │         │            │          │          │
│ created  │ TIMESTAMP    │        │CURRENT_TI│ [ ] │         │            │on update │          │
│ _at      │              │        │MESTAMP   │     │         │            │          │          │
└──────────┴──────────────┴────────┴──────────┴─────┴─────────┴────────────┴──────────┴──────────┘
```

---

### Column Name

- The **identifier** you use to refer to this column in SQL queries.

```
Naming Rules:
  ✅ Use lowercase letters
  ✅ Use underscores for spaces: first_name, not firstName or "first name"
  ✅ Be descriptive: email not e, created_at not cat
  ✅ Be consistent: if one table uses "name", all should use "name"
  ❌ No spaces
  ❌ No hyphens
  ❌ No special characters (except underscore)
  ❌ Don't use MySQL reserved words: date, order, select, key, index

Good column names:     Bad column names:
  id                     ID, Id, i
  first_name             firstName, "first name", fn
  email_address          Email, e, eml
  phone_number           phone#, Tel, ph
  created_at             createdAt, date, timestamp
  is_active              active_or_not, flag1
  user_id                userID, uid, u_id
```

---

### Data Type

- The **most important decision** for each column — it controls what values can be stored.
- MySQL will **reject any value that doesn't match the declared type**.
- Choosing the right type saves disk space and prevents incorrect data.

```
phpMyAdmin groups data types in the Type dropdown:

  NUMERIC:    INT, TINYINT, SMALLINT, MEDIUMINT, BIGINT
              DECIMAL, FLOAT, DOUBLE, BIT

  DATE AND TIME: DATE, DATETIME, TIMESTAMP, TIME, YEAR

  STRING:     CHAR, VARCHAR, TINYTEXT, TEXT, MEDIUMTEXT, LONGTEXT
              ENUM, SET, BINARY, VARBINARY, BLOB, TINYBLOB, MEDIUMBLOB, LONGBLOB

  SPATIAL:    GEOMETRY, POINT, LINESTRING, POLYGON (for map/GPS data)

  JSON:       JSON (MySQL 5.7.8+)
```

---

### Length / Values

- For some types, you must specify a **maximum length** or a **list of allowed values**.

```
VARCHAR:  Length = maximum number of characters
  VARCHAR(100)  → max 100 characters  (names, titles)
  VARCHAR(255)  → max 255 characters  (emails, URLs) ← most common
  VARCHAR(500)  → max 500 characters  (short descriptions)
  VARCHAR(65535)→ max possible but use TEXT instead

INT:      Length = display width (historical, not storage size)
  INT(11) → the (11) is just display hint, doesn't limit value range
  In MySQL 8.0+ this is deprecated — just use INT, no length needed

DECIMAL:  Length = total digits, decimal places
  DECIMAL(10, 2) → 10 total digits, 2 after decimal
  Example: 99999999.99 (8 before + 2 after = 10 total)
  Use for: prices, financial data

ENUM:     Length = comma-separated list of allowed values (in quotes)
  'active','inactive','banned'
  Only these exact values can be stored

SET:      Like ENUM but a row can have MULTIPLE values
  'php','python','javascript'
  A row could store 'php,javascript' (both at once)

CHAR:     Fixed-length string (padded with spaces if shorter)
  CHAR(2) for country codes: 'US', 'MM', 'UK'
  More efficient than VARCHAR for fixed-length data
```

---

### Default Value

- The value MySQL automatically inserts if you don't specify one during INSERT.

```
Common defaults:

  None       → No default. Column must be provided on INSERT
               (unless it's nullable or AUTO_INCREMENT)

  As defined → phpMyAdmin shows specific values below:
    CURRENT_TIMESTAMP  → Fills with current date+time (use for created_at)
    NULL               → Stores NULL if not provided
    0                  → Numeric zero
    Empty string ''    → Empty text

  Custom     → Type your own default value:
    'active'   → for a status column (ENUM)
    'user'     → for a role column
    '0'        → for a numeric flag

Examples:
  id         → None    (AUTO_INCREMENT fills it)
  name       → None    (must always be provided)
  status     → 'active'    (new users start as active)
  role       → 'user'      (new users are regular users)
  balance    → 0.00        (start with zero balance)
  created_at → CURRENT_TIMESTAMP (auto-filled to now)
  updated_at → CURRENT_TIMESTAMP (auto-filled, updated on changes)
  is_active  → 1       (active by default)
```

---

### NULL vs NOT NULL

- **NULL** means "no value" — the field is unknown or doesn't apply.
- A column is either **nullable** (can store NULL) or **NOT NULL** (must always have a value).

```
In phpMyAdmin:
  The "Null" column has a checkbox.
  ☐ Unchecked → NOT NULL (must have a value)
  ☑ Checked   → NULL allowed (value is optional)

When to allow NULL:
  ✅ Optional data that genuinely may not exist:
     phone_number  → user might not have a phone
     bio           → user might not write a bio
     deleted_at    → NULL means "not deleted"

When to use NOT NULL:
  ✅ Required data that must always exist:
     name          → every user must have a name
     email         → every user must have an email
     password      → every user must have a password
     created_at    → every record needs a creation time
     status        → every user has some status

Rule of thumb:
  Start with NOT NULL (unchecked checkbox).
  Only allow NULL when the absence of data is genuinely meaningful.
  NULL in most columns causes complex "IS NULL" checks everywhere.
```

> ⚠️ **Beginner Mistake:** Making all columns nullable "just in case" — this leads to data quality problems. If a name is nullable, you might insert users without names. MySQL lets you do it. Your app breaks later.

---

### Index / Primary Key

- An **index** is a data structure that lets MySQL find rows much faster — like an index at the back of a book.
- There are four index types in phpMyAdmin:

```
PRIMARY:
  → The primary key — uniquely identifies every row
  → ONLY ONE primary key per table (usually the "id" column)
  → Cannot be NULL, cannot be duplicate
  → MySQL automatically creates an index on the primary key
  → Choose this for your "id" column

UNIQUE:
  → Values in this column must be unique across all rows
  → Unlike PRIMARY, you can have multiple UNIQUE indexes per table
  → Can be NULL (but only one NULL — NULLs don't count as duplicates)
  → Choose this for: email, username, phone, slug
  → Prevents duplicate emails, duplicate usernames

INDEX (plain):
  → Speeds up searches and JOINs on this column
  → Does NOT require unique values — duplicates allowed
  → Choose this for: status, role, user_id (foreign key columns),
    any column you search/filter/sort by often

FULLTEXT:
  → Special index for full-text search operations
  → Use with MATCH() AGAINST() queries
  → Choose this for: article content, product descriptions
  → Allows searching for words within long text
```

```
Real-world index choices:

Table: users
  id         → PRIMARY   (unique identifier)
  email      → UNIQUE    (no duplicate emails)
  status     → INDEX     (filter by status often)
  created_at → INDEX     (sort by date often)
  name       → (none usually — searching by name uses LIKE, not =)

Table: orders
  id         → PRIMARY
  user_id    → INDEX     (frequently look up orders by user)
  status     → INDEX     (filter shipped vs pending)
  created_at → INDEX     (sort by date)

Table: products
  id         → PRIMARY
  slug       → UNIQUE    (URL-friendly name must be unique)
  category_id→ INDEX     (filter by category often)
  name       → FULLTEXT  (search products by keyword)
```

---

### AUTO_INCREMENT

- When checked (the **A_I checkbox** in phpMyAdmin), MySQL automatically assigns the next available number when you insert a row.
- You NEVER need to provide the value manually — MySQL handles it.
- Only valid on **integer** columns and only one per table.
- Almost always used with PRIMARY KEY.

```
How AUTO_INCREMENT works:
  INSERT INTO users (name, email) VALUES ('Phyo', 'phyo@example.com');
  → MySQL assigns id = 1 automatically

  INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');
  → MySQL assigns id = 2 automatically

  DELETE FROM users WHERE id = 2;
  → id 2 is gone

  INSERT INTO users (name, email) VALUES ('Bob', 'bob@example.com');
  → MySQL assigns id = 3 (NOT 2! AUTO_INCREMENT never reuses deleted IDs)

The AUTO_INCREMENT counter only goes UP — never reuses deleted IDs.
This is intentional — it prevents old links (like /users/2) from
accidentally pointing to a different user after Bob was deleted and
someone else was inserted.
```

---

### Attributes

- Extra constraints or behaviors for numeric and timestamp columns.

```
UNSIGNED:
  → For integer columns only
  → Removes negative numbers, doubles the positive range
  → INT range:          -2,147,483,648 to 2,147,483,647
  → INT UNSIGNED range: 0 to 4,294,967,295
  → Use for: id, age, quantity, price (in cents), counts
  → Never makes sense to have a negative user ID or quantity

ZEROFILL:
  → Pads the number with leading zeros up to the display width
  → INT(5) ZEROFILL → 42 displays as 00042
  → Deprecated in MySQL 8.0.17 — avoid in new code

ON UPDATE CURRENT_TIMESTAMP:
  → For TIMESTAMP columns only
  → Every time a row is UPDATED, this column automatically changes
    to the current time
  → Perfect for "updated_at" columns
  → You never need to manually set updated_at in your SQL
```

---

### Comments

- A text description of what this column is for — stored in the database schema.
- Visible in phpMyAdmin's Structure view and in `SHOW CREATE TABLE` output.
- Useful for future-you reading the schema months later.

```
Good column comments:
  id         → "Primary key, auto-incremented"
  email      → "Unique email address used for login"
  password   → "bcrypt hashed password, never plain text"
  status     → "User account status: active, inactive, banned"
  deleted_at → "NULL = not deleted; timestamp = soft deletion time"
  user_id    → "Foreign key referencing users.id"
```

---

## MySQL Data Types — Deep Dive

---

### Integer Types

Use integers for whole numbers — IDs, counts, ages, quantities, flags.

| Type | Storage | Signed Range | Unsigned Range | Use For |
|---|---|---|---|---|
| `TINYINT` | 1 byte | -128 to 127 | 0 to 255 | Booleans (0/1), ratings (1-5), small flags |
| `SMALLINT` | 2 bytes | -32,768 to 32,767 | 0 to 65,535 | Year, small counters |
| `MEDIUMINT` | 3 bytes | -8.3M to 8.3M | 0 to 16.7M | Medium-sized IDs |
| `INT` | 4 bytes | -2.1B to 2.1B | 0 to 4.3B | Most IDs and counts ← use this |
| `BIGINT` | 8 bytes | -9.2 quintillion to 9.2 quintillion | 0 to 18.4 quintillion | Twitter-scale IDs, large counts |

```sql
-- Practical integer column examples:
id            INT UNSIGNED NOT NULL AUTO_INCREMENT  -- user/product/order ID
age           TINYINT UNSIGNED                      -- 0 to 255 (more than enough)
quantity      SMALLINT UNSIGNED                     -- 0 to 65,535 items
views         INT UNSIGNED                          -- 0 to 4.3 billion views
total_revenue BIGINT UNSIGNED                       -- very large number in cents
is_active     TINYINT(1) NOT NULL DEFAULT 1         -- 0=false, 1=true (boolean)
rating        TINYINT UNSIGNED                      -- 1 to 5 stars
```

> 💡 **Tip:** Use `TINYINT(1)` for boolean columns (true/false, yes/no, active/inactive). PHP PDO reads `TINYINT(1)` as a PHP boolean automatically. Alternatively, MySQL 8.0.17+ supports `BOOLEAN` as an alias.

---

### Decimal / Float Types

Use for numbers with decimal points — but choose carefully!

| Type | Storage | Precision | Use For |
|---|---|---|---|
| `DECIMAL(M,D)` | Varies | **Exact** — no rounding errors | Money, financial data, tax rates |
| `FLOAT` | 4 bytes | ~7 significant digits (approximate) | Scientific measurements, GPS coords |
| `DOUBLE` | 8 bytes | ~15 significant digits (approximate) | High-precision scientific data |

```sql
-- Money MUST use DECIMAL — NEVER use FLOAT for money!
price         DECIMAL(10, 2)    -- Up to 99,999,999.99 — perfect for prices
tax_rate      DECIMAL(5, 4)     -- e.g. 0.0900 (9.00% tax rate)
weight        DECIMAL(8, 3)     -- e.g. 1.250 kg
discount      DECIMAL(5, 2)     -- e.g. 25.00 (percent)

-- Float for non-financial decimals
latitude      FLOAT             -- GPS latitude: -90.0000 to 90.0000
longitude     FLOAT             -- GPS longitude: -180.0000 to 180.0000
temperature   FLOAT             -- Sensor reading
```

> ⚠️ **Critical Warning:** NEVER use `FLOAT` or `DOUBLE` for money. They store approximate values — `0.1 + 0.2` in a FLOAT column might equal `0.30000000000000004` — not `0.30`. This causes rounding errors in financial calculations. Always use `DECIMAL` for any monetary value.

---

### String Types

Use for text — but choose the right type for the length of text.

| Type | Max Size | Storage | Use For |
|---|---|---|---|
| `CHAR(n)` | 255 chars | Fixed (always n bytes) | Fixed-length codes: country codes, status codes |
| `VARCHAR(n)` | 65,535 bytes | Variable (only what's needed) | Most text: names, emails, titles, URLs |
| `TINYTEXT` | 255 bytes | Variable | Very short text |
| `TEXT` | 65,535 bytes (~64KB) | Variable | Blog posts, descriptions, comments |
| `MEDIUMTEXT` | 16,777,215 bytes (~16MB) | Variable | Long articles, code snippets |
| `LONGTEXT` | 4,294,967,295 bytes (~4GB) | Variable | Books, enormous text blobs |

```sql
-- Practical string column examples:
name          VARCHAR(100)   -- Full names (most names under 100 chars)
email         VARCHAR(255)   -- Email addresses (RFC limit is 254 chars)
password      VARCHAR(255)   -- bcrypt hash is 60 chars, argon2 is longer
phone         VARCHAR(20)    -- Phone with country code: +95 9 123 456 789
slug          VARCHAR(255)   -- URL slug: my-product-name
title         VARCHAR(255)   -- Article/product title
country_code  CHAR(2)        -- ISO country code: 'MM', 'US', 'GB' (always 2!)
status        CHAR(10)       -- Short fixed statuses if using CHAR
ip_address    VARCHAR(45)    -- IPv4 (15 chars) or IPv6 (39 chars) + buffer
description   TEXT           -- Short description (up to 64KB)
content       LONGTEXT       -- Blog post content, long articles
meta_keywords TEXT           -- SEO keywords
```

```
VARCHAR vs TEXT decision:
  VARCHAR:
    ✅ Can be part of an index (can be the primary or unique key)
    ✅ Can have a default value
    ✅ Stored inline with the row (faster for small text)
    ❌ Max 65,535 bytes per row (shared with all VARCHAR columns)
    → Use when the text is under ~500 characters

  TEXT:
    ✅ Can store much more data
    ❌ Cannot be a primary/unique key directly
    ❌ Cannot have a default value
    ❌ Stored externally (pointer in row) — slightly slower
    → Use for descriptions, posts, content — anything over ~500 chars
```

---

### Date & Time Types

| Type | Format | Range | Use For |
|---|---|---|---|
| `DATE` | `YYYY-MM-DD` | 1000-01-01 to 9999-12-31 | Birth dates, expiry dates, scheduled dates |
| `TIME` | `HH:MM:SS` | -838:59:59 to 838:59:59 | Duration, time of day |
| `DATETIME` | `YYYY-MM-DD HH:MM:SS` | 1000-01-01 to 9999-12-31 | Scheduled events, logs — timezone-independent |
| `TIMESTAMP` | `YYYY-MM-DD HH:MM:SS` | 1970-01-01 to 2038-01-19 | created_at, updated_at — timezone-aware |
| `YEAR` | `YYYY` | 1901 to 2155 | Year only |

```sql
-- Practical date/time column examples:
created_at    TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
updated_at    TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
deleted_at    TIMESTAMP NULL     DEFAULT NULL          -- NULL = not deleted (soft delete)
birth_date    DATE      NOT NULL                       -- Just the date, no time
published_at  DATETIME  NULL     DEFAULT NULL          -- Scheduled publish time
expires_at    DATETIME  NULL                           -- Subscription/token expiry
event_date    DATE      NOT NULL                       -- Calendar event date
```

### TIMESTAMP vs DATETIME — The Important Difference

```
TIMESTAMP:
  ✅ Automatically converts to UTC when stored
  ✅ Converts back to the session's timezone when read
  ✅ Supports DEFAULT CURRENT_TIMESTAMP and ON UPDATE CURRENT_TIMESTAMP
  ❌ Limited range: 1970 to 2038 (the "Year 2038 problem")
  → Use for: created_at, updated_at (automatic record timestamps)

DATETIME:
  ✅ Stores exactly what you give it — no timezone conversion
  ✅ Wider range: 1000 to 9999
  ❌ No automatic CURRENT_TIMESTAMP on update in older MySQL
  → Use for: scheduled events, appointments, published_at
    (where you want to store "3pm Friday" regardless of timezone)
```

---

### Other Types

```sql
-- ENUM: fixed list of allowed values
status  ENUM('active','inactive','banned')  NOT NULL  DEFAULT 'active'
role    ENUM('user','admin','editor','moderator')  NOT NULL  DEFAULT 'user'
gender  ENUM('male','female','other','prefer_not_to_say')  NULL

-- ENUM pros:
--   ✅ Only allows valid values — MySQL rejects anything not in the list
--   ✅ Stored as integer internally — fast and compact
--   ✅ Easy to read in queries and phpMyAdmin

-- ENUM cons:
--   ❌ Adding a new value requires ALTER TABLE (table lock in old MySQL)
--   ❌ Less flexible than a VARCHAR with a CHECK constraint

-- SET: column can hold multiple values from the list simultaneously
permissions  SET('read','write','delete','admin')

-- JSON: store JSON data directly (MySQL 5.7.8+)
settings     JSON                        -- {"theme":"dark","lang":"en"}
metadata     JSON  NULL  DEFAULT NULL

-- BOOLEAN / BOOL: alias for TINYINT(1)
is_active    BOOLEAN NOT NULL DEFAULT TRUE     -- = TINYINT(1)
is_verified  BOOLEAN NOT NULL DEFAULT FALSE

-- BLOB: binary data (images, files — but store files on disk/S3 instead!)
avatar_image BLOB    -- ❌ Don't store files in DB — use file paths instead
avatar_path  VARCHAR(500)  -- ✅ Store the file path, keep file on disk

-- UUID: universally unique identifier (alternative to AUTO_INCREMENT)
uuid         CHAR(36)  NOT NULL               -- 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11'
-- Or in MySQL 8.0+:
uuid         VARCHAR(36)  NOT NULL  DEFAULT (UUID())
```

---

## The Essential Columns Every Table Should Have

Professional backend developers include these columns in **almost every table**. They are the foundation of good data management.

### The "Standard 4" Columns

```sql
-- These go on EVERY table you create:

`id`         INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
`created_at` TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
`updated_at` TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
`deleted_at` TIMESTAMP    NULL     DEFAULT NULL,

-- id:
--   Uniquely identifies every row
--   AUTO_INCREMENT — MySQL assigns it automatically
--   UNSIGNED — IDs are never negative
--   PRIMARY KEY — the main unique index

-- created_at:
--   Automatically set to the current time when a row is inserted
--   You NEVER have to set this manually — MySQL does it
--   Lets you see when each record was created
--   Useful for sorting, reporting, auditing

-- updated_at:
--   Automatically updated to current time whenever the row is CHANGED
--   ON UPDATE CURRENT_TIMESTAMP — MySQL updates it on every UPDATE
--   Lets you see when a record was last modified
--   Crucial for caching (has this record changed since we last read it?)

-- deleted_at:
--   NULL = the record is active / not deleted
--   TIMESTAMP value = the record was "soft deleted" at that time
--   This is called "Soft Delete" — records are never truly removed
--   Why? So you can restore accidentally deleted data
--   So you can audit deletion history
--   So foreign key references don't break
```

### Soft Delete Explained

```sql
-- Instead of: DELETE FROM users WHERE id = 5;
-- Do:         UPDATE users SET deleted_at = NOW() WHERE id = 5;

-- Then in ALL your queries, add: WHERE deleted_at IS NULL
SELECT * FROM users WHERE deleted_at IS NULL;               -- Only active users
SELECT * FROM orders WHERE deleted_at IS NULL;              -- Only active orders

-- Restore a soft-deleted record:
UPDATE users SET deleted_at = NULL WHERE id = 5;

-- See all deleted records:
SELECT * FROM users WHERE deleted_at IS NOT NULL;

-- Laravel's Eloquent ORM handles this automatically with:
-- use Illuminate\Database\Eloquent\SoftDeletes;
```

> 💡 **Why soft delete?** Imagine a customer calls and says "I accidentally deleted my order, can you restore it?" With hard DELETE — impossible, gone forever. With soft delete — one UPDATE and the order is back. This is standard practice in every professional PHP application.

---

## Creating Real-World Tables — Full Examples

---

### Users Table

```sql
-- Complete users table — production-ready
CREATE TABLE `users` (
  `id`           INT UNSIGNED   NOT NULL AUTO_INCREMENT,
  `name`         VARCHAR(100)   NOT NULL                    COMMENT 'Full display name',
  `email`        VARCHAR(255)   NOT NULL                    COMMENT 'Unique login email',
  `password`     VARCHAR(255)   NOT NULL                    COMMENT 'bcrypt/argon2 hash',
  `phone`        VARCHAR(20)    NULL     DEFAULT NULL       COMMENT 'Optional phone number',
  `role`         ENUM('user','admin','editor') NOT NULL DEFAULT 'user',
  `status`       ENUM('active','inactive','banned') NOT NULL DEFAULT 'active',
  `email_verified_at` TIMESTAMP NULL DEFAULT NULL          COMMENT 'NULL = not verified',
  `avatar`       VARCHAR(500)   NULL     DEFAULT NULL       COMMENT 'Profile image path',
  `created_at`   TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at`   TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `deleted_at`   TIMESTAMP      NULL     DEFAULT NULL       COMMENT 'Soft delete timestamp',
  PRIMARY KEY (`id`),
  UNIQUE KEY `users_email_unique` (`email`),
  KEY `users_status_index` (`status`),
  KEY `users_role_index` (`role`),
  KEY `users_deleted_at_index` (`deleted_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**How to enter this in phpMyAdmin (column by column):**

```
Column 1: id
  Name:    id
  Type:    INT
  Length:  (empty)
  Default: None
  Null:    ☐ (unchecked — NOT NULL)
  Index:   PRIMARY
  A_I:     ☑ (checked)
  Attr:    UNSIGNED

Column 2: name
  Name:    name
  Type:    VARCHAR
  Length:  100
  Default: None
  Null:    ☐ (NOT NULL)

Column 3: email
  Name:    email
  Type:    VARCHAR
  Length:  255
  Default: None
  Null:    ☐ (NOT NULL)
  Index:   UNIQUE
  Comment: "Unique login email"

Column 4: password
  Name:    password
  Type:    VARCHAR
  Length:  255
  Default: None
  Null:    ☐ (NOT NULL)
  Comment: "bcrypt/argon2 hash — never plain text"

Column 5: role
  Name:    role
  Type:    ENUM
  Length:  'user','admin','editor'    ← type these in the Values box
  Default: user
  Null:    ☐ (NOT NULL)

Column 6: status
  Name:    status
  Type:    ENUM
  Length:  'active','inactive','banned'
  Default: active
  Null:    ☐ (NOT NULL)

Column 7: created_at
  Name:    created_at
  Type:    TIMESTAMP
  Default: CURRENT_TIMESTAMP
  Null:    ☐ (NOT NULL)

  → In the Default dropdown, look for "As defined:"
    and type CURRENT_TIMESTAMP
    OR some phpMyAdmin versions show it as a dropdown option

Column 8: updated_at
  Name:    updated_at
  Type:    TIMESTAMP
  Default: CURRENT_TIMESTAMP
  Null:    ☐ (NOT NULL)
  Attr:    on update CURRENT_TIMESTAMP  ← select from Attributes dropdown

Column 9: deleted_at
  Name:    deleted_at
  Type:    TIMESTAMP
  Default: NULL  (select "As defined" and type NULL, or leave blank)
  Null:    ☑ (checked — CAN be NULL)
  Comment: "Soft delete — NULL means active"

→ Click "Save"
```

---

### Products Table

```sql
CREATE TABLE `products` (
  `id`           INT UNSIGNED   NOT NULL AUTO_INCREMENT,
  `name`         VARCHAR(255)   NOT NULL,
  `slug`         VARCHAR(255)   NOT NULL          COMMENT 'URL-friendly name: my-product',
  `description`  TEXT           NULL,
  `price`        DECIMAL(10,2)  NOT NULL DEFAULT 0.00,
  `sale_price`   DECIMAL(10,2)  NULL     DEFAULT NULL,
  `stock`        INT UNSIGNED   NOT NULL DEFAULT 0,
  `sku`          VARCHAR(100)   NULL     DEFAULT NULL  COMMENT 'Stock keeping unit',
  `category_id`  INT UNSIGNED   NULL     DEFAULT NULL,
  `image`        VARCHAR(500)   NULL     DEFAULT NULL,
  `is_active`    TINYINT(1)     NOT NULL DEFAULT 1,
  `created_at`   TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at`   TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `deleted_at`   TIMESTAMP      NULL     DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `products_slug_unique` (`slug`),
  UNIQUE KEY `products_sku_unique` (`sku`),
  KEY `products_category_id_index` (`category_id`),
  KEY `products_is_active_index` (`is_active`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### Orders Table

```sql
CREATE TABLE `orders` (
  `id`           INT UNSIGNED   NOT NULL AUTO_INCREMENT,
  `user_id`      INT UNSIGNED   NOT NULL              COMMENT 'References users.id',
  `subtotal`     DECIMAL(10,2)  NOT NULL DEFAULT 0.00,
  `tax`          DECIMAL(10,2)  NOT NULL DEFAULT 0.00,
  `shipping`     DECIMAL(10,2)  NOT NULL DEFAULT 0.00,
  `total`        DECIMAL(10,2)  NOT NULL DEFAULT 0.00,
  `status`       ENUM('pending','confirmed','processing','shipped','delivered','cancelled','refunded')
                                NOT NULL DEFAULT 'pending',
  `notes`        TEXT           NULL,
  `shipped_at`   TIMESTAMP      NULL     DEFAULT NULL,
  `delivered_at` TIMESTAMP      NULL     DEFAULT NULL,
  `created_at`   TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at`   TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `deleted_at`   TIMESTAMP      NULL     DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `orders_user_id_index` (`user_id`),
  KEY `orders_status_index` (`status`),
  KEY `orders_created_at_index` (`created_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

### Categories Table

```sql
CREATE TABLE `categories` (
  `id`           INT UNSIGNED   NOT NULL AUTO_INCREMENT,
  `parent_id`    INT UNSIGNED   NULL     DEFAULT NULL  COMMENT 'NULL = top-level category',
  `name`         VARCHAR(100)   NOT NULL,
  `slug`         VARCHAR(100)   NOT NULL,
  `description`  TEXT           NULL,
  `sort_order`   INT UNSIGNED   NOT NULL DEFAULT 0     COMMENT 'Display order',
  `is_active`    TINYINT(1)     NOT NULL DEFAULT 1,
  `created_at`   TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at`   TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `categories_slug_unique` (`slug`),
  KEY `categories_parent_id_index` (`parent_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## Viewing and Modifying Table Structure

### View the Structure of a Table

```
In phpMyAdmin:
  STEP 1: Click table name in left panel
  STEP 2: Click "Structure" tab

  You'll see:
  ┌──────────────┬──────────────────┬──────┬─────┬───────────────────┬───────────┐
  │ Name         │ Type             │ Null │ Key │ Default           │ Extra     │
  ├──────────────┼──────────────────┼──────┼─────┼───────────────────┼───────────┤
  │ id           │ int unsigned     │ No   │ PRI │ NULL              │ auto_incr │
  │ name         │ varchar(100)     │ No   │     │ NULL              │           │
  │ email        │ varchar(255)     │ No   │ UNI │ NULL              │           │
  │ status       │ enum(act,inact..)│ No   │     │ active            │           │
  │ created_at   │ timestamp        │ No   │     │ CURRENT_TIMESTAMP │           │
  └──────────────┴──────────────────┴──────┴─────┴───────────────────┴───────────┘

  Each row has icons:  ✎ (Edit column)  🗑 (Drop column)
```

```sql
-- OR run in SQL tab:
DESCRIBE users;
SHOW COLUMNS FROM users;
SHOW CREATE TABLE users;   -- shows complete CREATE TABLE statement
```

### Adding a New Column

```
STEP 1: Click table → Structure tab
STEP 2: At the bottom, find:
  "Add [1] column(s) [After ▼] [created_at ▼]"
  → Set how many columns to add
  → Set where to add them (After which existing column)
STEP 3: Click "Go"
STEP 4: Fill in the new column definition → Save
```

```sql
-- Or via SQL tab:
ALTER TABLE users ADD COLUMN bio TEXT NULL AFTER email;
ALTER TABLE users ADD COLUMN points INT UNSIGNED NOT NULL DEFAULT 0 AFTER status;
```

### Modifying an Existing Column

```
STEP 1: Click table → Structure tab
STEP 2: Click ✎ (Edit/pencil) next to the column you want to change
STEP 3: Modify the definition → Save

phpMyAdmin runs:
  ALTER TABLE users MODIFY COLUMN name VARCHAR(200) NOT NULL;
```

---

## Common Mistakes When Creating Tables

```
❌ MISTAKE 1: Using VARCHAR for phone numbers
  WRONG:  phone INT          → "+95 9 123 456" can't be stored as INT
  RIGHT:  phone VARCHAR(20)  → stores any format: +959123456, 09-123-456

❌ MISTAKE 2: Using FLOAT for money
  WRONG:  price FLOAT        → 19.99 + 0.01 might equal 19.999999...
  RIGHT:  price DECIMAL(10,2)→ exact decimal arithmetic

❌ MISTAKE 3: Using TEXT everywhere
  WRONG:  name TEXT          → can't index it easily, wastes space
  RIGHT:  name VARCHAR(100)  → right size, indexable, default-able

❌ MISTAKE 4: Forgetting the character set
  WRONG:  Default charset (sometimes latin1 on old MySQL installs)
  RIGHT:  Always specify: DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
          Otherwise Burmese text and emoji get corrupted or stored as ???

❌ MISTAKE 5: No created_at column
  WRONG:  Forgetting to add timestamp columns
  RIGHT:  Every table should have created_at and usually updated_at

❌ MISTAKE 6: All columns nullable
  WRONG:  Making everything NULL "just in case"
  RIGHT:  Only nullable when absence of data is genuinely meaningful
          Default to NOT NULL

❌ MISTAKE 7: id as SIGNED INT
  WRONG:  id INT (signed) — wastes half the range on negative numbers
  RIGHT:  id INT UNSIGNED — IDs are never negative, doubles the range

❌ MISTAKE 8: Using hard DELETE instead of soft delete
  WRONG:  Deleting rows permanently from the DB
  RIGHT:  Use deleted_at TIMESTAMP NULL and mark deletion time
          (for important tables at least)

❌ MISTAKE 9: No indexes on foreign key columns
  WRONG:  user_id INT NOT NULL (no index)
  RIGHT:  user_id INT UNSIGNED NOT NULL, KEY orders_user_id (user_id)
          Without this index, "get all orders by user 42" scans the whole table

❌ MISTAKE 10: Storing passwords as plain text
  WRONG:  password VARCHAR(255) → "mypassword123"
  RIGHT:  password VARCHAR(255) → "$2y$10$hashedvalue..." (bcrypt hash)
          NEVER store plain text passwords — use password_hash() in PHP
```

---

## Quick Revision

- A **table** stores data in rows (records) and columns (fields). Plan your table design on paper before creating it.
- **Column Name:** lowercase, underscores not spaces or hyphens, descriptive, consistent. Never use MySQL reserved words.
- **Data Type:** most important decision — must match the data:
  - `INT UNSIGNED` for IDs and counts, `DECIMAL(10,2)` for money, `VARCHAR(n)` for most text, `TEXT` for long content, `TIMESTAMP` for timestamps, `ENUM` for fixed value sets.
- **Length/Values:** `VARCHAR(255)` for emails/URLs, `VARCHAR(100)` for names, `DECIMAL(10,2)` for prices, `ENUM('a','b','c')` for fixed value sets.
- **Default:** `CURRENT_TIMESTAMP` for `created_at`, a meaningful value for ENUM columns (`'active'`, `'user'`), `0` for numeric defaults. No default for required text columns.
- **NULL vs NOT NULL:** Prefer NOT NULL (unchecked). Only allow NULL when the absence of data is genuinely meaningful (optional phone, soft-delete timestamp, unverified email).
- **Index types:** `PRIMARY` (one per table, for `id`), `UNIQUE` (for email, slug, username), `INDEX` (for foreign keys and filter columns), `FULLTEXT` (for search columns).
- **AUTO_INCREMENT:** check the A_I checkbox for your `id` column — MySQL assigns the next number automatically. Never reuses deleted IDs.
- **UNSIGNED:** always add to integer IDs and counts — removes negative values and doubles the positive range.
- **The Standard 4 columns:** every table should have `id` (PK, AUTO_INCREMENT, UNSIGNED), `created_at` (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP), `updated_at` (TIMESTAMP, ON UPDATE CURRENT_TIMESTAMP), `deleted_at` (TIMESTAMP, NULL — for soft deletes).
- **Soft delete:** set `deleted_at = NOW()` instead of `DELETE FROM table`. Then filter with `WHERE deleted_at IS NULL`. Allows data recovery and audit trails.
- **Always set** `ENGINE=InnoDB` and `CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci` — InnoDB for transactions and foreign keys, `utf8mb4` for full Unicode (Burmese, emoji, all languages).
- **Never store** phone numbers as INT, money as FLOAT, or passwords as plain text.
- **Add indexes** on every column you filter/sort/JOIN on — especially foreign key columns like `user_id`, `category_id`.