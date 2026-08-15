# SQL Basic Statements — The Core Commands Every Developer Must Know

These are the **7 foundational SQL statements** that power every database-driven application. Master these and you can build any backend feature — user systems, product catalogs, order management, anything.

---

## Table of Contents

1. [Overview — The 7 Core SQL Statements](#overview--the-7-core-sql-statements)
2. [Setup — Our Practice Database](#setup--our-practice-database)
3. [`CREATE DATABASE`](#create-database)
4. [`CREATE TABLE`](#create-table)
5. [`INSERT INTO`](#insert-into)
6. [`SELECT`](#select)
   - [Select All Columns](#select-all-columns)
   - [Select Specific Columns](#select-specific-columns)
   - [WHERE Clause](#where-clause)
   - [ORDER BY](#order-by)
   - [LIMIT](#limit)
   - [Combining Clauses](#combining-clauses)
7. [`UPDATE`](#update)
8. [`DELETE`](#delete)
9. [`DROP TABLE`](#drop-table)
10. [`DROP DATABASE`](#drop-database)
11. [Putting It All Together — A Real Project Walkthrough](#putting-it-all-together--a-real-project-walkthrough)
12. [Quick Revision](#quick-revision)

---

## Overview — The 7 Core SQL Statements

```
Statement          Category    What it does
─────────────────────────────────────────────────────────────────
CREATE DATABASE    DDL         Creates a new database
CREATE TABLE       DDL         Creates a new table with columns
INSERT INTO        DML         Adds new rows of data
SELECT             DQL         Reads / retrieves data
UPDATE             DML         Modifies existing rows
DELETE             DML         Removes rows
DROP TABLE         DDL         Deletes a table and all its data
DROP DATABASE      DDL         Deletes an entire database
─────────────────────────────────────────────────────────────────
```

> 💡 **These 7 statements ARE the database.** Every feature in every web app — user registration, product browsing, order processing, admin dashboards — uses exactly these commands under the hood. PHP frameworks like Laravel wrap these in beautiful syntax, but underneath they generate exactly this SQL.

---

## Setup — Our Practice Database

Throughout this note, we'll build a simple **bookstore database** so you can see all statements working together with realistic data.

```
Our bookstore will have:
  → authors     (author information)
  → books       (book catalog)
  → customers   (people who buy books)
  → orders      (purchase records)

All examples will be hands-on — run them in phpMyAdmin's SQL tab
or in the MySQL command line: mysql -u root -p
```

---

## `CREATE DATABASE`

**What it does:** Creates a brand new, empty database on the MySQL server.

### Syntax

```sql
CREATE DATABASE database_name;

-- Full syntax with character set (always recommended):
CREATE DATABASE database_name
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

-- Safe version (no error if database already exists):
CREATE DATABASE IF NOT EXISTS database_name
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
```

### Examples

```sql
-- Example 1: Basic database creation
CREATE DATABASE bookstore;

-- Example 2: With character set (best practice — always do this)
CREATE DATABASE bookstore
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
-- utf8mb4 = full Unicode support (emoji, Burmese, Chinese, all languages)
-- utf8mb4_unicode_ci = case-insensitive comparisons (A = a when searching)

-- Example 3: Safe creation (no error if already exists)
CREATE DATABASE IF NOT EXISTS bookstore
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

-- Example 4: Multiple databases for different projects
CREATE DATABASE IF NOT EXISTS bookstore_dev   CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE DATABASE IF NOT EXISTS bookstore_test  CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE DATABASE IF NOT EXISTS bookstore_prod  CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### After Creating a Database — Switch to It

```sql
-- Tell MySQL "use this database for all following statements"
USE bookstore;

-- Confirm which database is active
SELECT DATABASE();
-- +------------+
-- | DATABASE() |
-- +------------+
-- | bookstore  |
-- +------------+

-- See all databases on this server
SHOW DATABASES;
-- +--------------------+
-- | Database           |
-- +--------------------+
-- | bookstore          |
-- | information_schema |
-- | mysql              |
-- | performance_schema |
-- | sys                |
-- +--------------------+
```

> ⚠️ **Warning:** `CREATE DATABASE` creates an empty database with no tables. It's like creating an empty folder. The database has no data until you create tables with `CREATE TABLE`.

---

## `CREATE TABLE`

**What it does:** Creates a new table inside the current database, defining its columns, data types, and constraints.

### Syntax

```sql
CREATE TABLE table_name (
    column1_name  datatype  [constraints],
    column2_name  datatype  [constraints],
    column3_name  datatype  [constraints],
    ...
    [table_constraints]
);
```

### Examples

```sql
-- First, make sure we're in our database
USE bookstore;

-- ─── Example 1: Simple table (minimal) ─────────────────────────────────────
CREATE TABLE authors (
    id    INT,
    name  VARCHAR(100),
    email VARCHAR(255)
);
-- This works but is missing best practices (no PRIMARY KEY, no NOT NULL, etc.)

-- ─── Example 2: Proper table with all best practices ───────────────────────
DROP TABLE IF EXISTS authors;   -- Remove the simple version first

CREATE TABLE authors (
    id         INT UNSIGNED    NOT NULL AUTO_INCREMENT,
    name       VARCHAR(100)    NOT NULL,
    email      VARCHAR(255)    NOT NULL,
    bio        TEXT            NULL,
    country    VARCHAR(100)    NULL     DEFAULT 'Unknown',
    is_active  TINYINT(1)      NOT NULL DEFAULT 1,
    created_at TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY authors_email_unique (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- ─── Example 3: Books table ────────────────────────────────────────────────
CREATE TABLE books (
    id           INT UNSIGNED    NOT NULL AUTO_INCREMENT,
    title        VARCHAR(255)    NOT NULL,
    author_id    INT UNSIGNED    NOT NULL,
    isbn         VARCHAR(20)     NOT NULL,
    price        DECIMAL(10, 2)  NOT NULL DEFAULT 0.00,
    stock        INT UNSIGNED    NOT NULL DEFAULT 0,
    genre        ENUM('fiction','non-fiction','science','history','biography','other')
                                 NOT NULL DEFAULT 'other',
    published_at DATE            NULL,
    description  TEXT            NULL,
    created_at   TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at   TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at   TIMESTAMP       NULL     DEFAULT NULL,
    PRIMARY KEY (id),
    UNIQUE KEY books_isbn_unique (isbn),
    KEY books_author_id_index (author_id),
    KEY books_genre_index (genre),
    FOREIGN KEY (author_id) REFERENCES authors(id) ON DELETE RESTRICT ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- ─── Example 4: Customers table ────────────────────────────────────────────
CREATE TABLE customers (
    id         INT UNSIGNED    NOT NULL AUTO_INCREMENT,
    name       VARCHAR(100)    NOT NULL,
    email      VARCHAR(255)    NOT NULL,
    phone      VARCHAR(20)     NULL,
    address    TEXT            NULL,
    created_at TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY customers_email_unique (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- ─── Example 5: Orders table ───────────────────────────────────────────────
CREATE TABLE orders (
    id          INT UNSIGNED    NOT NULL AUTO_INCREMENT,
    customer_id INT UNSIGNED    NOT NULL,
    book_id     INT UNSIGNED    NOT NULL,
    quantity    INT UNSIGNED    NOT NULL DEFAULT 1,
    total_price DECIMAL(10, 2)  NOT NULL,
    status      ENUM('pending','confirmed','shipped','delivered','cancelled')
                                NOT NULL DEFAULT 'pending',
    ordered_at  TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  TIMESTAMP       NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    KEY orders_customer_id_index (customer_id),
    KEY orders_book_id_index (book_id),
    KEY orders_status_index (status),
    FOREIGN KEY (customer_id) REFERENCES customers(id) ON DELETE RESTRICT,
    FOREIGN KEY (book_id)     REFERENCES books(id)     ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Verify Tables Were Created

```sql
-- See all tables in the current database
SHOW TABLES;
-- +--------------+
-- | Tables_in_bookstore |
-- +--------------+
-- | authors      |
-- | books        |
-- | customers    |
-- | orders       |
-- +--------------+

-- See the structure of a specific table
DESCRIBE authors;
-- +------------+--------------+------+-----+-------------------+-----------------------------+
-- | Field      | Type         | Null | Key | Default           | Extra                       |
-- +------------+--------------+------+-----+-------------------+-----------------------------+
-- | id         | int unsigned | NO   | PRI | NULL              | auto_increment              |
-- | name       | varchar(100) | NO   |     | NULL              |                             |
-- | email      | varchar(255) | NO   | UNI | NULL              |                             |
-- | bio        | text         | YES  |     | NULL              |                             |
-- | country    | varchar(100) | YES  |     | Unknown           |                             |
-- | is_active  | tinyint(1)   | NO   |     | 1                 |                             |
-- | created_at | timestamp    | NO   |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED           |
-- | updated_at | timestamp    | NO   |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED on update |
-- +------------+--------------+------+-----+-------------------+-----------------------------+

-- See the exact SQL that would recreate this table
SHOW CREATE TABLE authors\G
```

> 💡 **The `CREATE TABLE` checklist every time:**
> - ✅ Every table has an `id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY`
> - ✅ Every table has `created_at` and `updated_at` TIMESTAMP columns
> - ✅ Every column that must have data is `NOT NULL`
> - ✅ Tables end with `ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci`
> - ✅ Foreign keys reference the right table and column

---

## `INSERT INTO`

**What it does:** Adds one or more new rows of data into a table.

### Syntax

```sql
-- Single row, specify columns
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);

-- Single row, all columns in order (not recommended — fragile)
INSERT INTO table_name VALUES (value1, value2, value3, ...);

-- Multiple rows at once
INSERT INTO table_name (column1, column2)
VALUES
    (value1a, value2a),
    (value1b, value2b),
    (value1c, value2c);
```

### Examples

```sql
USE bookstore;

-- ─── Example 1: Insert ONE author ──────────────────────────────────────────
INSERT INTO authors (name, email, bio, country)
VALUES ('George Orwell', 'orwell@example.com', 'English novelist and essayist.', 'UK');
-- id is omitted → AUTO_INCREMENT assigns it (starts at 1)
-- is_active is omitted → DEFAULT 1 is used
-- created_at is omitted → DEFAULT CURRENT_TIMESTAMP is used
-- MySQL shows: 1 row affected, 1 row inserted. (id = 1)

-- ─── Example 2: Insert MULTIPLE authors at once (more efficient) ────────────
INSERT INTO authors (name, email, bio, country)
VALUES
    ('J.K. Rowling',    'jkr@example.com',    'British author of the Harry Potter series.', 'UK'),
    ('Yuval Noah Harari','yuval@example.com',  'Israeli historian and author of Sapiens.',   'Israel'),
    ('Agatha Christie', 'agatha@example.com',  'English mystery writer.',                    'UK'),
    ('Paulo Coelho',    'paulo@example.com',   'Brazilian lyricist and novelist.',            'Brazil'),
    ('Stephen King',    'stephen@example.com', 'American author of horror and fiction.',      'USA');
-- MySQL shows: 5 rows affected, 5 rows inserted.
-- IDs assigned: 2, 3, 4, 5, 6

-- ─── Example 3: Insert a book (references author_id) ───────────────────────
INSERT INTO books (title, author_id, isbn, price, stock, genre, published_at, description)
VALUES (
    '1984',
    1,                       -- George Orwell's id
    '978-0451524935',
    12.99,
    100,
    'fiction',
    '1949-06-08',
    'A dystopian novel set in a totalitarian society where the government, called Big Brother, controls every aspect of life.'
);

-- ─── Example 4: Insert multiple books ──────────────────────────────────────
INSERT INTO books (title, author_id, isbn, price, stock, genre, published_at)
VALUES
    ('Animal Farm',              1, '978-0451526342',  8.99,  75, 'fiction',     '1945-08-17'),
    ('Harry Potter and the PS',  2, '978-0439708180', 14.99, 200, 'fiction',     '1997-06-26'),
    ('Harry Potter and the CS',  2, '978-0439064873', 14.99, 180, 'fiction',     '1998-07-02'),
    ('Sapiens',                  3, '978-0062316097', 18.99,  60, 'history',     '2011-01-01'),
    ('Murder on the Orient Exp', 4, '978-0062073501',  9.99,  90, 'fiction',     '1934-01-01'),
    ('The Alchemist',            5, '978-0062315007', 11.99, 150, 'fiction',     '1988-01-01'),
    ('The Shining',              6, '978-0307743657', 13.99,  55, 'fiction',     '1977-01-28'),
    ('It',                       6, '978-1501156700', 16.99,  40, 'fiction',     '1986-09-15');

-- ─── Example 5: Insert customers ───────────────────────────────────────────
INSERT INTO customers (name, email, phone, address)
VALUES
    ('Phyo Min Paing',  'phyo@example.com',   '+95 9 123 456 789', 'Thingangyun, Yangon, Myanmar'),
    ('Alice Johnson',   'alice@example.com',   '+1 555 234 5678',   '123 Main St, New York, USA'),
    ('Bob Smith',       'bob@example.com',     '+44 20 1234 5678',  '456 Queen St, London, UK'),
    ('Charlie Brown',   'charlie@example.com', '+61 3 1234 5678',   '789 King St, Melbourne, AU'),
    ('Diana Prince',    'diana@example.com',   '+49 30 1234 5678',  '321 Berlin Str, Berlin, DE');

-- ─── Example 6: Insert orders ──────────────────────────────────────────────
INSERT INTO orders (customer_id, book_id, quantity, total_price, status)
VALUES
    (1, 1, 2, 25.98,  'delivered'),   -- Phyo bought 2x '1984'
    (1, 5, 1, 18.99,  'shipped'),     -- Phyo bought 1x 'Sapiens'
    (2, 3, 1, 14.99,  'delivered'),   -- Alice bought 'Harry Potter CS'
    (3, 7, 2, 27.98,  'pending'),     -- Bob bought 2x 'The Shining'
    (4, 6, 1, 11.99,  'confirmed'),   -- Charlie bought 'The Alchemist'
    (1, 8, 1, 16.99,  'pending'),     -- Phyo bought 'It'
    (5, 2, 3, 26.97,  'delivered'),   -- Diana bought 3x 'Animal Farm'
    (2, 4, 1, 14.99,  'shipped');     -- Alice bought 'Harry Potter PS'
```

### What Happens After INSERT

```sql
-- Verify inserts worked
SELECT * FROM authors;
SELECT * FROM books;
SELECT * FROM customers;
SELECT * FROM orders;

-- Get the ID of the last inserted row (useful in PHP)
SELECT LAST_INSERT_ID();
-- Returns the auto-incremented id of the most recent INSERT in this session

-- In PHP:
-- $lastId = $pdo->lastInsertId();
```

### INSERT Variations

```sql
-- INSERT IGNORE — skip if a UNIQUE constraint would be violated (no error)
INSERT IGNORE INTO authors (name, email)
VALUES ('George Orwell', 'orwell@example.com');
-- If orwell@example.com already exists → silently skip, no error

-- INSERT ... ON DUPLICATE KEY UPDATE — insert or update if unique key conflicts
INSERT INTO books (isbn, title, price, stock, author_id, genre)
VALUES ('978-0451524935', '1984', 13.99, 120, 1, 'fiction')
ON DUPLICATE KEY UPDATE
    price = VALUES(price),    -- update price to new value
    stock = VALUES(stock);    -- update stock to new value
-- If isbn '978-0451524935' already exists → UPDATE price and stock
-- If it doesn't exist → INSERT as normal

-- INSERT with SELECT (copy data from another table)
INSERT INTO archived_orders (customer_id, book_id, quantity, total_price, status, ordered_at)
SELECT customer_id, book_id, quantity, total_price, status, ordered_at
FROM orders
WHERE status = 'delivered' AND ordered_at < '2026-01-01';
```

---

## `SELECT`

**What it does:** Retrieves (reads) data from one or more tables. The most frequently used SQL statement.

### Syntax

```sql
SELECT column1, column2, ...
FROM table_name
[WHERE condition]
[ORDER BY column ASC|DESC]
[LIMIT number];
```

---

### Select All Columns

```sql
-- The asterisk (*) means "all columns"
SELECT * FROM authors;
-- Returns every column of every row

-- Output:
-- +----+------------------+----------------------+----------------------------+---------+-----------+---------------------+---------------------+
-- | id | name             | email                | bio                        | country | is_active | created_at          | updated_at          |
-- +----+------------------+----------------------+----------------------------+---------+-----------+---------------------+---------------------+
-- |  1 | George Orwell    | orwell@example.com   | English novelist...        | UK      |         1 | 2026-06-28 14:00:00 | 2026-06-28 14:00:00 |
-- |  2 | J.K. Rowling     | jkr@example.com      | British author...          | UK      |         1 | 2026-06-28 14:00:00 | 2026-06-28 14:00:00 |
-- ...

SELECT * FROM books;
SELECT * FROM customers;
SELECT * FROM orders;
```

> ⚠️ **Avoid `SELECT *` in production code.** It fetches columns you might not need (wasting bandwidth and memory), and if you add columns to the table later, your query results change unexpectedly. Always name the columns you need: `SELECT id, name, email FROM users;`

---

### Select Specific Columns

```sql
-- Select only the columns you need
SELECT name, email FROM authors;
-- +------------------+----------------------+
-- | name             | email                |
-- +------------------+----------------------+
-- | George Orwell    | orwell@example.com   |
-- | J.K. Rowling     | jkr@example.com      |
-- | Yuval Noah Harari| yuval@example.com    |
-- +------------------+----------------------+

SELECT title, price, stock FROM books;
-- +---------------------------------+-------+-------+
-- | title                           | price | stock |
-- +---------------------------------+-------+-------+
-- | 1984                            | 12.99 |   100 |
-- | Animal Farm                     |  8.99 |    75 |
-- | Harry Potter and the PS         | 14.99 |   200 |
-- +---------------------------------+-------+-------+

-- Column Aliases — rename columns in the output
SELECT
    name         AS author_name,
    country      AS author_country,
    is_active    AS active
FROM authors;
-- +------------------+----------------+--------+
-- | author_name      | author_country | active |
-- +------------------+----------------+--------+
-- | George Orwell    | UK             |      1 |
-- | J.K. Rowling     | UK             |      1 |
-- +------------------+----------------+--------+

-- Expressions in SELECT
SELECT
    title,
    price,
    price * 0.9     AS discounted_price,   -- calculate 10% off
    stock * price   AS inventory_value      -- total value in stock
FROM books;
-- +---------------------+-------+-----------------+-----------------+
-- | title               | price | discounted_price | inventory_value |
-- +---------------------+-------+-----------------+-----------------+
-- | 1984                | 12.99 | 11.691          | 1299.00         |
-- | Animal Farm         |  8.99 |  8.091          |  674.25         |
-- +---------------------+-------+-----------------+-----------------+
```

---

### WHERE Clause

**The WHERE clause filters which rows are returned.** Only rows where the condition is TRUE are included.

```sql
-- ─── Basic comparisons ─────────────────────────────────────────────────────
SELECT * FROM authors WHERE country = 'UK';
-- Returns only George Orwell, J.K. Rowling, Agatha Christie

SELECT * FROM books WHERE price < 10.00;
-- Returns books under $10

SELECT * FROM books WHERE price >= 12.00;
-- Returns books $12 or more

SELECT * FROM orders WHERE status = 'pending';
-- Returns all pending orders

SELECT * FROM orders WHERE status != 'cancelled';
-- Returns all orders that are NOT cancelled

-- ─── Multiple conditions with AND ──────────────────────────────────────────
-- AND: BOTH conditions must be true
SELECT * FROM books WHERE price < 15.00 AND stock > 50;
-- Books that are cheap AND well-stocked

SELECT * FROM authors WHERE country = 'UK' AND is_active = 1;
-- Active UK authors

SELECT * FROM orders WHERE customer_id = 1 AND status = 'delivered';
-- Phyo's delivered orders

-- ─── Multiple conditions with OR ───────────────────────────────────────────
-- OR: at least ONE condition must be true
SELECT * FROM orders WHERE status = 'pending' OR status = 'confirmed';
-- Orders that are pending OR confirmed

SELECT * FROM authors WHERE country = 'UK' OR country = 'USA';
-- Authors from UK OR USA

-- ─── Combining AND and OR (use parentheses to be explicit) ─────────────────
SELECT * FROM books
WHERE genre = 'fiction'
  AND (price < 10.00 OR stock > 150);
-- Fiction books that are either cheap or well-stocked

-- ─── NOT — negates a condition ─────────────────────────────────────────────
SELECT * FROM books WHERE NOT genre = 'fiction';
-- Same as: WHERE genre != 'fiction'

SELECT * FROM authors WHERE NOT country = 'UK';
-- Non-UK authors

-- ─── IN — match any value in a list ────────────────────────────────────────
SELECT * FROM authors WHERE country IN ('UK', 'USA', 'Australia');
-- Authors from UK, USA, or Australia

SELECT * FROM orders WHERE status IN ('pending', 'confirmed', 'shipped');
-- Active (not yet delivered/cancelled) orders

SELECT * FROM books WHERE id IN (1, 3, 5, 7);
-- Specific books by ID

-- ─── NOT IN ────────────────────────────────────────────────────────────────
SELECT * FROM orders WHERE status NOT IN ('cancelled', 'delivered');
-- Orders still in progress

-- ─── BETWEEN — range (inclusive on both ends) ──────────────────────────────
SELECT * FROM books WHERE price BETWEEN 10.00 AND 15.00;
-- Books priced $10 to $15 (including 10 and 15)

SELECT * FROM orders WHERE ordered_at BETWEEN '2026-01-01' AND '2026-06-30';
-- Orders placed in the first half of 2026

SELECT * FROM authors WHERE id BETWEEN 2 AND 5;
-- Authors with id 2, 3, 4, or 5

-- ─── LIKE — pattern matching ───────────────────────────────────────────────
-- % = wildcard for any number of characters
-- _ = wildcard for exactly ONE character
SELECT * FROM authors WHERE name LIKE 'J%';
-- Names starting with "J": J.K. Rowling

SELECT * FROM books WHERE title LIKE '%Harry Potter%';
-- Titles containing "Harry Potter"

SELECT * FROM customers WHERE email LIKE '%@example.com';
-- All example.com email addresses

SELECT * FROM authors WHERE name LIKE '%.K.%';
-- Names containing ".K.": J.K. Rowling

SELECT * FROM books WHERE isbn LIKE '978-%';
-- ISBNs starting with 978 (all modern ISBNs)

SELECT * FROM authors WHERE name LIKE '______%';
-- Names with 6 or more characters (6 underscores + %)

-- ─── IS NULL and IS NOT NULL ───────────────────────────────────────────────
SELECT * FROM authors WHERE bio IS NULL;
-- Authors with no bio entered

SELECT * FROM books WHERE deleted_at IS NULL;
-- Books that have NOT been soft-deleted (active books)

SELECT * FROM books WHERE deleted_at IS NOT NULL;
-- Books that HAVE been soft-deleted

SELECT * FROM customers WHERE phone IS NOT NULL;
-- Customers who provided a phone number
```

---

### ORDER BY

**ORDER BY sorts the result rows by one or more columns.**

```sql
-- Ascending order (A→Z, 0→9) — default
SELECT name, country FROM authors ORDER BY name ASC;
SELECT name, country FROM authors ORDER BY name;    -- ASC is the default

-- Descending order (Z→A, 9→0)
SELECT title, price FROM books ORDER BY price DESC;
-- Most expensive books first

-- Sort by multiple columns (primary sort, then secondary)
SELECT name, country FROM authors ORDER BY country ASC, name ASC;
-- First sorted by country alphabetically
-- Within the same country, sorted by name alphabetically

SELECT status, ordered_at FROM orders ORDER BY status ASC, ordered_at DESC;
-- Grouped by status, newest first within each status

-- Sort by column position (1 = first column, 2 = second, etc.)
SELECT name, price FROM books ORDER BY 2 DESC;  -- ORDER BY price DESC
-- Useful for aliases

-- Sort by an alias
SELECT title, price * 0.9 AS sale_price FROM books ORDER BY sale_price ASC;
```

---

### LIMIT

**LIMIT restricts how many rows are returned.**

```sql
-- Get the first 5 rows
SELECT * FROM books LIMIT 5;

-- Get the top 3 most expensive books
SELECT title, price FROM books ORDER BY price DESC LIMIT 3;
-- +------------------------------+-------+
-- | title                        | price |
-- +------------------------------+-------+
-- | Sapiens                      | 18.99 |
-- | It                           | 16.99 |
-- | Harry Potter and the CS      | 14.99 |
-- +------------------------------+-------+

-- LIMIT with OFFSET — for pagination
-- LIMIT rows_per_page OFFSET skip_this_many_rows
SELECT * FROM books ORDER BY id ASC LIMIT 3 OFFSET 0;   -- Page 1 (rows 1-3)
SELECT * FROM books ORDER BY id ASC LIMIT 3 OFFSET 3;   -- Page 2 (rows 4-6)
SELECT * FROM books ORDER BY id ASC LIMIT 3 OFFSET 6;   -- Page 3 (rows 7-9)

-- Alternative syntax: LIMIT offset, count
SELECT * FROM books LIMIT 0, 3;   -- Page 1
SELECT * FROM books LIMIT 3, 3;   -- Page 2
SELECT * FROM books LIMIT 6, 3;   -- Page 3

-- Pagination formula:
-- offset = (current_page - 1) * rows_per_page
-- PHP example:
-- $page = 2;  $perPage = 3;
-- $offset = ($page - 1) * $perPage;  // = 3
-- "SELECT * FROM books LIMIT $perPage OFFSET $offset"
```

---

### Combining Clauses

In a SELECT, the order of clauses is always:

```
SELECT → FROM → WHERE → ORDER BY → LIMIT
```

```sql
-- Full example combining all clauses:

-- "Get the titles and prices of all fiction books
--  that cost between $10 and $20,
--  ordered by price from cheapest to most expensive,
--  showing only the first 3 results"
SELECT
    title,
    price,
    genre
FROM books
WHERE genre = 'fiction'
  AND price BETWEEN 10.00 AND 20.00
ORDER BY price ASC
LIMIT 3;

-- Result:
-- +---------------------------+-------+---------+
-- | title                     | price | genre   |
-- +---------------------------+-------+---------+
-- | 1984                      | 12.99 | fiction |
-- | The Shining               | 13.99 | fiction |
-- | Harry Potter and the PS   | 14.99 | fiction |
-- +---------------------------+-------+---------+

-- ─────────────────────────────────────────────────────────────────────────────

-- "Get the name and email of UK authors
--  whose bio is not null,
--  sorted alphabetically by name"
SELECT name, email, country
FROM authors
WHERE country = 'UK'
  AND bio IS NOT NULL
ORDER BY name ASC;

-- ─────────────────────────────────────────────────────────────────────────────

-- "Get the 5 most recent pending or confirmed orders"
SELECT id, customer_id, book_id, quantity, total_price, status, ordered_at
FROM orders
WHERE status IN ('pending', 'confirmed')
ORDER BY ordered_at DESC
LIMIT 5;
```

### Useful SELECT Functions

```sql
-- COUNT — count rows
SELECT COUNT(*) AS total_books FROM books;                          -- all books
SELECT COUNT(*) AS fiction_count FROM books WHERE genre = 'fiction'; -- fiction only

-- SUM — add up values
SELECT SUM(total_price) AS total_revenue FROM orders WHERE status = 'delivered';
SELECT SUM(stock * price) AS inventory_value FROM books;

-- AVG — average value
SELECT AVG(price) AS average_price FROM books;
SELECT AVG(total_price) AS avg_order_value FROM orders;

-- MIN / MAX — smallest and largest
SELECT MIN(price) AS cheapest FROM books;
SELECT MAX(price) AS most_expensive FROM books;

-- All aggregate functions together
SELECT
    COUNT(*)       AS total_books,
    AVG(price)     AS avg_price,
    MIN(price)     AS min_price,
    MAX(price)     AS max_price,
    SUM(stock)     AS total_stock
FROM books;

-- DISTINCT — remove duplicate values
SELECT DISTINCT country FROM authors;
-- +--------+
-- | country|
-- +--------+
-- | UK     |
-- | Israel |
-- | Brazil |
-- | USA    |
-- +--------+

SELECT DISTINCT genre FROM books;
-- Shows each genre only once

SELECT DISTINCT status FROM orders;
-- Shows each unique order status

-- GROUP BY — group rows and aggregate per group
SELECT genre, COUNT(*) AS book_count, AVG(price) AS avg_price
FROM books
GROUP BY genre;
-- +-----------+------------+-----------+
-- | genre     | book_count | avg_price |
-- +-----------+------------+-----------+
-- | fiction   |          7 |  12.7700  |
-- | history   |          1 |  18.9900  |
-- +-----------+------------+-----------+

SELECT country, COUNT(*) AS author_count
FROM authors
GROUP BY country
ORDER BY author_count DESC;

-- HAVING — filter groups (like WHERE but for GROUP BY results)
SELECT genre, COUNT(*) AS book_count
FROM books
GROUP BY genre
HAVING book_count > 1;   -- only genres with more than 1 book
-- (cannot use WHERE for this — WHERE filters rows BEFORE grouping)
```

---

## `UPDATE`

**What it does:** Modifies existing rows in a table.

### Syntax

```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

> ⚠️ **ALWAYS include WHERE in UPDATE.** Without WHERE, EVERY row in the table is updated.

### Examples

```sql
-- ─── Example 1: Update a single row by ID ──────────────────────────────────
UPDATE authors
SET country = 'England'
WHERE id = 1;
-- Changes George Orwell's country from 'UK' to 'England'
-- MySQL shows: 1 row affected

-- ─── Example 2: Update multiple columns at once ─────────────────────────────
UPDATE books
SET price = 11.99, stock = 120
WHERE id = 1;
-- Updates both price and stock for book id=1

-- ─── Example 3: Update based on a condition (not just by ID) ───────────────
UPDATE orders
SET status = 'confirmed'
WHERE status = 'pending' AND ordered_at < '2026-06-01';
-- Confirm all orders that have been pending since before June 2026

-- ─── Example 4: Update multiple rows matching a condition ───────────────────
UPDATE books
SET price = price * 0.90   -- apply 10% discount to the current price
WHERE genre = 'fiction';
-- Reduces ALL fiction book prices by 10%

UPDATE books
SET stock = stock + 50
WHERE author_id = 2;
-- Restock all J.K. Rowling books by 50 units

-- ─── Example 5: Update with calculation ────────────────────────────────────
UPDATE orders
SET total_price = quantity * (
    SELECT price FROM books WHERE books.id = orders.book_id
)
WHERE status = 'pending';
-- Recalculate total_price based on current book prices for pending orders

-- ─── Example 6: Soft delete (set deleted_at instead of DELETE) ─────────────
UPDATE books
SET deleted_at = NOW()
WHERE id = 8;
-- "Deletes" the book logically — it still exists but is hidden

-- Restore a soft-deleted book:
UPDATE books
SET deleted_at = NULL
WHERE id = 8;

-- ─── Example 7: Update using CASE (conditional update) ─────────────────────
UPDATE books
SET price = CASE genre
    WHEN 'fiction'     THEN price * 0.95   -- 5% off fiction
    WHEN 'non-fiction' THEN price * 0.90   -- 10% off non-fiction
    WHEN 'history'     THEN price * 0.85   -- 15% off history
    ELSE price                              -- no change for others
END;
-- Different discounts for different genres in one statement
```

### Verify Updates

```sql
-- Always verify an UPDATE with a SELECT first:

-- STEP 1: Check what rows will be affected
SELECT * FROM books WHERE genre = 'fiction';
-- See the rows — do they look right?

-- STEP 2: Run the UPDATE
UPDATE books SET price = price * 0.90 WHERE genre = 'fiction';

-- STEP 3: Verify the result
SELECT title, genre, price FROM books WHERE genre = 'fiction';
```

---

## `DELETE`

**What it does:** Removes rows from a table permanently.

### Syntax

```sql
DELETE FROM table_name
WHERE condition;
```

> ⚠️ **ALWAYS include WHERE in DELETE.** Without WHERE, EVERY row in the table is deleted. This cannot be undone (without a backup or transaction).

### Examples

```sql
-- ─── Example 1: Delete a single specific row ────────────────────────────────
DELETE FROM orders WHERE id = 6;
-- Deletes only the order with id = 6

-- ─── Example 2: Delete by a specific condition ─────────────────────────────
DELETE FROM orders WHERE status = 'cancelled';
-- Deletes all cancelled orders

-- ─── Example 3: Delete by multiple conditions ──────────────────────────────
DELETE FROM orders
WHERE customer_id = 3
  AND status = 'pending';
-- Deletes only Bob's pending orders (not his delivered ones)

-- ─── Example 4: Delete with IN ─────────────────────────────────────────────
DELETE FROM books WHERE id IN (7, 8);
-- Deletes 'The Shining' and 'It'

-- ─── Example 5: Delete old data (cleanup task) ─────────────────────────────
DELETE FROM orders
WHERE status = 'delivered'
  AND ordered_at < (NOW() - INTERVAL 1 YEAR);
-- Delete delivered orders older than 1 year

-- ─── Example 6: TRUNCATE — delete ALL rows, fast (resets AUTO_INCREMENT) ────
TRUNCATE TABLE orders;
-- Instantly empties the entire orders table
-- AUTO_INCREMENT resets to 1
-- Much faster than DELETE for large tables

-- ─── Example 7: The SAFE pattern — verify before deleting ──────────────────
-- STEP 1: Select to see what you're about to delete
SELECT id, status, ordered_at FROM orders
WHERE status = 'cancelled';

-- STEP 2: Looks right? Then delete
DELETE FROM orders WHERE status = 'cancelled';

-- STEP 3: Verify
SELECT COUNT(*) AS cancelled_orders FROM orders WHERE status = 'cancelled';
-- Should return 0
```

### DELETE with a LIMIT (safety net)

```sql
-- Useful for deleting in batches to avoid locking large tables
DELETE FROM orders
WHERE status = 'delivered'
ORDER BY ordered_at ASC
LIMIT 100;   -- delete only 100 at a time
-- Run this repeatedly until no more rows match
```

### The Soft Delete Alternative

```sql
-- Instead of permanently deleting, mark as deleted:
UPDATE books SET deleted_at = NOW() WHERE id = 5;

-- Then always filter in queries:
SELECT * FROM books WHERE deleted_at IS NULL;

-- Why? So you can recover data, see history, and prevent orphaned references.
```

---

## `DROP TABLE`

**What it does:** Permanently deletes an entire table — both its structure (columns, indexes, constraints) and all its data.

### Syntax

```sql
DROP TABLE table_name;

-- Safe version (no error if table doesn't exist):
DROP TABLE IF EXISTS table_name;

-- Drop multiple tables at once:
DROP TABLE IF EXISTS table1, table2, table3;
```

### Examples

```sql
-- ─── Example 1: Drop a single table ────────────────────────────────────────
DROP TABLE authors;
-- ⚠️ All data in 'authors' is permanently gone
-- ⚠️ If 'books' has a FOREIGN KEY referencing 'authors', this will FAIL
--    (referential integrity prevents dropping the parent table)

-- ─── Example 2: Safe drop (no error if table doesn't exist) ─────────────────
DROP TABLE IF EXISTS authors;
-- No error if 'authors' doesn't exist — safe to run in scripts

-- ─── Example 3: Drop multiple tables ───────────────────────────────────────
-- When dropping tables that reference each other via foreign keys,
-- drop the CHILD tables first, then the PARENT tables

-- orders references customers and books → drop orders first
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS books;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS authors;

-- ─── Example 4: Drop to rebuild (development workflow) ──────────────────────
-- Sometimes you want to completely rebuild a table during development
DROP TABLE IF EXISTS users;
CREATE TABLE users (
    id         INT UNSIGNED NOT NULL AUTO_INCREMENT,
    name       VARCHAR(100) NOT NULL,
    -- ... your new design
    PRIMARY KEY (id)
);

-- ─── Example 5: DROP vs TRUNCATE vs DELETE ──────────────────────────────────

-- DELETE FROM table WHERE condition:  Remove specific rows (can be rolled back)
DELETE FROM orders WHERE status = 'cancelled';

-- TRUNCATE TABLE table:              Remove ALL rows (resets AUTO_INCREMENT, faster)
TRUNCATE TABLE orders;

-- DROP TABLE table:                  Remove the ENTIRE TABLE (data + structure gone)
DROP TABLE orders;
```

### DROP TABLE Comparison

| Command | Removes Data | Removes Structure | Resets AUTO_INCREMENT | Rollback Possible |
|---|---|---|---|---|
| `DELETE FROM` (no WHERE) | ✅ Yes | ❌ No | ❌ No | ✅ Yes |
| `TRUNCATE TABLE` | ✅ Yes | ❌ No | ✅ Yes | ❌ No |
| `DROP TABLE` | ✅ Yes | ✅ Yes | N/A | ❌ No |

> ⚠️ **Warning:** In production, never `DROP TABLE` without a fresh backup. Even in development, double-check you're on the right database (check `SELECT DATABASE();` first!).

---

## `DROP DATABASE`

**What it does:** Permanently deletes an entire database — every table, every row, every index, every view, everything inside it. Gone instantly.

### Syntax

```sql
DROP DATABASE database_name;

-- Safe version:
DROP DATABASE IF EXISTS database_name;
```

### Examples

```sql
-- ─── Example 1: Basic drop ──────────────────────────────────────────────────
DROP DATABASE bookstore;
-- ⚠️ EVERYTHING inside bookstore is permanently deleted
-- ⚠️ Cannot be undone

-- ─── Example 2: Safe drop ────────────────────────────────────────────────────
DROP DATABASE IF EXISTS bookstore;
-- No error if 'bookstore' doesn't exist

-- ─── Example 3: Development workflow — drop and recreate ───────────────────
DROP DATABASE IF EXISTS bookstore;
CREATE DATABASE bookstore CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE bookstore;
-- Now create your tables fresh...

-- ─── Example 4: Verify before dropping ──────────────────────────────────────
-- ALWAYS check which database you're on before dropping!
SELECT DATABASE();              -- what's currently selected?
SHOW TABLES;                    -- see what's inside
SELECT COUNT(*) FROM authors;   -- how much data?

-- Then if you're sure:
DROP DATABASE bookstore;
```

> ⚠️ **Extreme Warning — The Most Dangerous SQL Command:**
> `DROP DATABASE` on a production server is catastrophic. A single typo can destroy years of data in milliseconds. Production best practices:
> - Remove DROP DATABASE permission from all application users
> - Root user should only connect from localhost
> - Always have recent backups
> - Some teams disable DROP DATABASE at the user permission level entirely

---

## Putting It All Together — A Real Project Walkthrough

Let's run through the complete lifecycle of our bookstore database from scratch.

```sql
-- ══════════════════════════════════════════════════════════════════════════
-- PHASE 1: CREATE THE DATABASE
-- ══════════════════════════════════════════════════════════════════════════

CREATE DATABASE IF NOT EXISTS bookstore
  CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE bookstore;

-- ══════════════════════════════════════════════════════════════════════════
-- PHASE 2: CREATE THE TABLES
-- ══════════════════════════════════════════════════════════════════════════

CREATE TABLE authors (
    id         INT UNSIGNED NOT NULL AUTO_INCREMENT,
    name       VARCHAR(100) NOT NULL,
    email      VARCHAR(255) NOT NULL,
    country    VARCHAR(100) NULL DEFAULT 'Unknown',
    created_at TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY authors_email_unique (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE books (
    id         INT UNSIGNED   NOT NULL AUTO_INCREMENT,
    title      VARCHAR(255)   NOT NULL,
    author_id  INT UNSIGNED   NOT NULL,
    price      DECIMAL(10,2)  NOT NULL DEFAULT 0.00,
    stock      INT UNSIGNED   NOT NULL DEFAULT 0,
    genre      ENUM('fiction','non-fiction','history','other') NOT NULL DEFAULT 'other',
    created_at TIMESTAMP      NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP      NULL DEFAULT NULL,
    PRIMARY KEY (id),
    KEY books_author_id_index (author_id),
    FOREIGN KEY (author_id) REFERENCES authors(id) ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- ══════════════════════════════════════════════════════════════════════════
-- PHASE 3: INSERT DATA
-- ══════════════════════════════════════════════════════════════════════════

INSERT INTO authors (name, email, country) VALUES
    ('George Orwell', 'orwell@example.com', 'UK'),
    ('J.K. Rowling',  'jkr@example.com',   'UK'),
    ('Stephen King',  'king@example.com',   'USA');

INSERT INTO books (title, author_id, price, stock, genre) VALUES
    ('1984',              1, 12.99, 100, 'fiction'),
    ('Animal Farm',       1,  8.99,  75, 'fiction'),
    ('Harry Potter PS',   2, 14.99, 200, 'fiction'),
    ('The Shining',       3, 13.99,  55, 'fiction'),
    ('It',                3, 16.99,  40, 'fiction');

-- ══════════════════════════════════════════════════════════════════════════
-- PHASE 4: QUERY THE DATA
-- ══════════════════════════════════════════════════════════════════════════

-- Show all books
SELECT id, title, price, stock FROM books;

-- Show only fiction books under $15
SELECT title, price, genre
FROM books
WHERE genre = 'fiction' AND price < 15.00
ORDER BY price ASC;

-- Count books per author
SELECT author_id, COUNT(*) AS book_count
FROM books
GROUP BY author_id;

-- ══════════════════════════════════════════════════════════════════════════
-- PHASE 5: UPDATE DATA
-- ══════════════════════════════════════════════════════════════════════════

-- Restock '1984'
UPDATE books SET stock = 150 WHERE id = 1;

-- Apply a 5% price increase to all books
UPDATE books SET price = ROUND(price * 1.05, 2);

-- Verify changes
SELECT title, price, stock FROM books ORDER BY id;

-- ══════════════════════════════════════════════════════════════════════════
-- PHASE 6: DELETE DATA (soft delete)
-- ══════════════════════════════════════════════════════════════════════════

-- Soft delete 'It' (out of print)
UPDATE books SET deleted_at = NOW() WHERE id = 5;

-- All queries now exclude soft-deleted books:
SELECT * FROM books WHERE deleted_at IS NULL;

-- ══════════════════════════════════════════════════════════════════════════
-- PHASE 7: CLEANUP (development only)
-- ══════════════════════════════════════════════════════════════════════════

DROP TABLE IF EXISTS books;
DROP TABLE IF EXISTS authors;
DROP DATABASE IF EXISTS bookstore;
```

---

## Quick Revision

- **`CREATE DATABASE name CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;`** — creates a new database. Always specify utf8mb4 for full Unicode support. Use `IF NOT EXISTS` to prevent errors.
- **`CREATE TABLE`** — defines a table's columns, types, and constraints. Every table needs: `id INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY`, `created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP`, `updated_at TIMESTAMP ON UPDATE CURRENT_TIMESTAMP`, and `ENGINE=InnoDB DEFAULT CHARSET=utf8mb4`.
- **`INSERT INTO table (cols) VALUES (vals)`** — adds rows. Omit `id`, `created_at`, `updated_at` — MySQL fills them automatically. Insert multiple rows by listing multiple `(val, val)` groups.
- **`SELECT cols FROM table`** — reads data. `*` returns all columns (avoid in production). Always name the columns you need. Clauses in order: `SELECT → FROM → WHERE → ORDER BY → LIMIT`.
- **`WHERE`** filters rows. Operators: `=`, `!=`, `<`, `>`, `<=`, `>=`, `AND`, `OR`, `NOT`, `IN (...)`, `NOT IN (...)`, `BETWEEN a AND b`, `LIKE '%pattern%'`, `IS NULL`, `IS NOT NULL`.
- **`ORDER BY col ASC/DESC`** sorts results. Multiple columns: `ORDER BY col1 ASC, col2 DESC`.
- **`LIMIT n OFFSET m`** — return n rows starting from position m. Pagination formula: `OFFSET = (page - 1) * per_page`.
- **`UPDATE table SET col=val WHERE condition`** — modifies existing rows. **ALWAYS use WHERE** — without it, every row is updated.
- **`DELETE FROM table WHERE condition`** — removes rows permanently. **ALWAYS use WHERE** — without it, every row is deleted. Prefer soft delete: `UPDATE SET deleted_at = NOW()`.
- **`DROP TABLE table_name`** — deletes the table AND all data, permanently. Can't be undone. Drop children before parents (foreign key order).
- **`DROP DATABASE name`** — deletes everything in the database instantly. The most dangerous command. Always backup first.
- **Safe delete pattern:** SELECT first → check rows → DELETE. Then verify with SELECT COUNT(*).
- **The golden WHERE rule:** Before running UPDATE or DELETE, run the equivalent SELECT and verify the rows that will be affected. Then run the UPDATE/DELETE.