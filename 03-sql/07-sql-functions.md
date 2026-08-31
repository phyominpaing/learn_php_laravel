# SQL Functions — Aggregate, Scalar, Aliases & HAVING

**SQL functions** are built-in tools that process data and return a result — either summarizing an entire group of rows (aggregate functions like COUNT, SUM) or transforming a single value (scalar functions like ROUND, CONCAT). Mastering these is what lets you answer real business questions directly in SQL: "What is our total revenue?", "Who is our top customer?", "How many users signed up this month?"

---

## Table of Contents

1. [Two Types of SQL Functions](#two-types-of-sql-functions)
2. [Aggregate Functions — Summarizing Groups of Rows](#aggregate-functions--summarizing-groups-of-rows)
   - [COUNT()](#count)
   - [SUM()](#sum)
   - [AVG()](#avg)
   - [MIN()](#min)
   - [MAX()](#max)
   - [GROUP BY — The Partner of Aggregate Functions](#group-by--the-partner-of-aggregate-functions)
   - [HAVING — Filtering Groups](#having--filtering-groups)
3. [Scalar Functions — Transforming Single Values](#scalar-functions--transforming-single-values)
   - [ROUND(), FLOOR(), CEIL()](#round-floor-ceil)
   - [RAND()](#rand)
   - [NOW(), CURDATE(), CURTIME()](#now-curdate-curtime)
   - [YEAR(), MONTH(), DAY()](#year-month-day)
   - [CAST()](#cast)
   - [CONCAT()](#concat)
4. [AS — Aliases](#as--aliases)
   - [Column Aliases](#column-aliases)
   - [Table Aliases](#table-aliases)
5. [Combining Everything — Real Queries](#combining-everything--real-queries)
6. [SQL Functions in PHP with PDO](#sql-functions-in-php-with-pdo)
7. [Quick Revision](#quick-revision)

---

## Two Types of SQL Functions

```
AGGREGATE FUNCTIONS          SCALAR FUNCTIONS
(operate on many rows)       (operate on one value at a time)

COUNT()  → how many rows     ROUND()  → round a number
SUM()    → total of a column FLOOR()  → round down
AVG()    → average value     CEIL()   → round up
MIN()    → smallest value    RAND()   → random number
MAX()    → largest value     NOW()    → current datetime
                             YEAR()   → extract year
                             MONTH()  → extract month
                             DAY()    → extract day
                             CAST()   → convert data type
                             CONCAT() → join strings

Used with GROUP BY           Used in SELECT, WHERE, ORDER BY
Returns ONE row per group    Returns one value per row
```

---

## Aggregate Functions — Summarizing Groups of Rows

Aggregate functions **collapse multiple rows into a single summary value**. They answer questions like "how many?", "what's the total?", "what's the average?".

---

### COUNT()

- Counts the **number of rows** that match a condition.
- `COUNT(*)` — counts ALL rows including NULLs.
- `COUNT(column_name)` — counts only rows where that column is NOT NULL.
- `COUNT(DISTINCT column)` — counts unique values only.

```sql
-- Basic COUNT — how many books are in the table?
SELECT COUNT(*) FROM books;
-- Result: 20

-- COUNT with a column name (excludes NULLs in that column)
SELECT COUNT(published_at) FROM books;
-- If 5 books have NULL published_at → returns 15 (not 20)

-- COUNT with WHERE — how many fiction books?
SELECT COUNT(*) AS fiction_count
FROM books
WHERE genre = 'fiction';
-- Result: 7

-- COUNT with WHERE — how many books cost over $30?
SELECT COUNT(*) AS expensive_books
FROM books
WHERE price > 30;
-- Result: 4

-- COUNT DISTINCT — how many unique genres exist?
SELECT COUNT(DISTINCT genre) AS total_genres
FROM books;
-- Result: 5 (fiction, horror, science, history, programming)

-- COUNT DISTINCT — how many unique authors?
SELECT COUNT(DISTINCT author) AS total_authors FROM books;

-- COUNT in multiple WHERE conditions
SELECT COUNT(*) AS active_user_count
FROM users
WHERE status = 'active' AND deleted_at IS NULL;

-- COUNT with GROUP BY (most powerful) — books per genre
SELECT genre, COUNT(*) AS book_count
FROM books
GROUP BY genre
ORDER BY book_count DESC;
```

```
Result of COUNT by genre:
┌─────────────┬────────────┐
│ genre       │ book_count │
├─────────────┼────────────┤
│ fiction     │          7 │
│ programming │          5 │
│ horror      │          3 │
│ history     │          3 │
│ science     │          2 │
└─────────────┴────────────┘
```

---

### SUM()

- Adds up all values in a column.
- Ignores NULL values automatically.
- Only works on numeric columns.

```sql
-- Total value of all books in the bookstore
SELECT SUM(price) AS total_value FROM books;
-- Result: 374.83 (sum of all 20 book prices)

-- Total stock across all books
SELECT SUM(stock) AS total_stock FROM books;
-- Result: 1710 (sum of all stock quantities)

-- Total value of all fiction books
SELECT SUM(price) AS fiction_total
FROM books
WHERE genre = 'fiction';

-- Total stock of programming books
SELECT SUM(stock) AS programming_stock
FROM books
WHERE genre = 'programming';

-- SUM with GROUP BY — total stock per genre
SELECT genre, SUM(stock) AS total_stock
FROM books
GROUP BY genre
ORDER BY total_stock DESC;

-- Total revenue from delivered orders
SELECT SUM(total) AS total_revenue
FROM orders
WHERE status = 'delivered'
  AND deleted_at IS NULL;

-- Revenue by month
SELECT
  YEAR(created_at)  AS year,
  MONTH(created_at) AS month,
  SUM(total)        AS monthly_revenue
FROM orders
WHERE status NOT IN ('cancelled', 'refunded')
GROUP BY YEAR(created_at), MONTH(created_at)
ORDER BY year ASC, month ASC;

-- SUM of a calculated column: total inventory value (price × stock)
SELECT
  genre,
  SUM(price * stock) AS inventory_value
FROM books
GROUP BY genre
ORDER BY inventory_value DESC;
```

---

### AVG()

- Calculates the **arithmetic mean** (sum ÷ count) of a column.
- Ignores NULL values.
- Only works on numeric columns.

```sql
-- Average price of all books
SELECT AVG(price) AS avg_price FROM books;
-- Result: 18.74 (approximately)

-- Average price of programming books
SELECT AVG(price) AS avg_programming_price
FROM books
WHERE genre = 'programming';

-- Average rating of all books
SELECT AVG(rating) AS avg_rating FROM books;

-- Average with ROUND (AVG returns many decimal places)
SELECT ROUND(AVG(price), 2) AS avg_price FROM books;
-- Result: 18.74 (clean 2 decimal places)

-- AVG per genre
SELECT genre, ROUND(AVG(price), 2) AS avg_price
FROM books
GROUP BY genre
ORDER BY avg_price DESC;

-- AVG with GROUP BY — average order value per user
SELECT
  user_id,
  COUNT(*)              AS order_count,
  ROUND(AVG(total), 2) AS avg_order_value,
  SUM(total)            AS lifetime_value
FROM orders
WHERE status NOT IN ('cancelled', 'refunded')
  AND deleted_at IS NULL
GROUP BY user_id
ORDER BY lifetime_value DESC;
```

---

### MIN()

- Returns the **smallest value** in a column.
- Works on numbers, text (alphabetical), and dates (earliest).
- With WHERE: finds the minimum among matching rows only.

```sql
-- Syntax
SELECT MIN(column_name) FROM table_name WHERE condition;

-- Cheapest book in the entire store
SELECT MIN(price) AS cheapest_price FROM books;
-- Result: 8.99 (Animal Farm)

-- What IS the cheapest book? (get title too)
SELECT id, title, price
FROM books
WHERE price = (SELECT MIN(price) FROM books);
-- Result: Animal Farm — $8.99

-- Cheapest book in each genre
SELECT genre, MIN(price) AS cheapest
FROM books
GROUP BY genre
ORDER BY cheapest ASC;

-- Lowest stock level (to know what needs restocking most urgently)
SELECT MIN(stock) AS lowest_stock FROM books;

-- Earliest order date (when did we get our first order?)
SELECT MIN(created_at) AS first_order_date FROM orders;

-- Minimum rating (what's our worst-rated book?)
SELECT MIN(rating) AS worst_rating FROM books;

-- MIN with WHERE — cheapest fiction book
SELECT MIN(price) AS cheapest_fiction
FROM books
WHERE genre = 'fiction';
```

```
Result of MIN per genre:
┌─────────────┬──────────┐
│ genre       │ cheapest │
├─────────────┼──────────┤
│ fiction     │     8.99 │
│ horror      │    11.99 │
│ science     │    13.99 │
│ history     │    15.99 │
│ programming │    24.99 │
└─────────────┴──────────┘
```

---

### MAX()

- Returns the **largest value** in a column.
- Works on numbers, text (alphabetical), and dates (most recent).

```sql
-- Syntax
SELECT MAX(column_name) FROM table_name WHERE condition;

-- Most expensive book
SELECT MAX(price) AS most_expensive FROM books;
-- Result: 44.99 (Design Patterns)

-- What IS the most expensive book?
SELECT id, title, price
FROM books
WHERE price = (SELECT MAX(price) FROM books);
-- Result: Design Patterns — $44.99

-- Most expensive book per genre
SELECT genre, MAX(price) AS most_expensive
FROM books
GROUP BY genre
ORDER BY most_expensive DESC;

-- Highest-rated book
SELECT MAX(rating) AS best_rating FROM books;

-- Most stock in any single book
SELECT MAX(stock) AS highest_stock FROM books;

-- Latest order date
SELECT MAX(created_at) AS most_recent_order FROM orders;

-- MAX with WHERE — highest-priced programming book
SELECT MAX(price) AS most_expensive_programming
FROM books
WHERE genre = 'programming';

-- MAX and MIN together — price spread (range)
SELECT
  MIN(price) AS cheapest,
  MAX(price) AS most_expensive,
  MAX(price) - MIN(price) AS price_range
FROM books;
```

### All Aggregate Functions Together

```sql
-- Complete summary of each genre in one query
SELECT
  genre,
  COUNT(*)                    AS total_books,
  MIN(price)                  AS cheapest,
  MAX(price)                  AS most_expensive,
  ROUND(AVG(price), 2)        AS avg_price,
  SUM(stock)                  AS total_stock,
  ROUND(AVG(rating), 1)       AS avg_rating,
  MIN(rating)                 AS worst_rating,
  MAX(rating)                 AS best_rating
FROM books
GROUP BY genre
ORDER BY avg_price DESC;
```

```
Result:
┌─────────────┬─────────┬──────────┬────────────────┬───────────┬─────────────┬────────────┐
│ genre       │ total   │ cheapest │ most_expensive │ avg_price │ total_stock │ avg_rating │
├─────────────┼─────────┼──────────┼────────────────┼───────────┼─────────────┼────────────┤
│ programming │       5 │    24.99 │          44.99 │     35.19 │         270 │        4.6 │
│ history     │       3 │    15.99 │          17.99 │     16.99 │         200 │        4.6 │
│ science     │       2 │    13.99 │          15.99 │     14.99 │         160 │        4.6 │
│ horror      │       3 │    11.99 │          16.99 │     14.32 │         155 │        4.5 │
│ fiction     │       7 │     8.99 │          14.99 │     12.71 │         925 │        4.7 │
└─────────────┴─────────┴──────────┴────────────────┴───────────┴─────────────┴────────────┘
```

---

### GROUP BY — The Partner of Aggregate Functions

- **GROUP BY** groups rows that have the same value in specified columns into a single summary row.
- Every column in SELECT must either be in GROUP BY OR be inside an aggregate function.

```sql
-- Without GROUP BY → one result for the whole table
SELECT COUNT(*) FROM books;           -- 20 (all books)
SELECT AVG(price) FROM books;         -- 18.74 (all books average)

-- With GROUP BY → one result per group
SELECT genre, COUNT(*) FROM books GROUP BY genre;
-- One row per genre with that genre's count

-- GROUP BY multiple columns
SELECT genre, author, COUNT(*) AS book_count
FROM books
GROUP BY genre, author
ORDER BY genre, book_count DESC;
-- Groups by the combination of genre AND author

-- GROUP BY with ORDER BY
SELECT genre, SUM(stock) AS total_stock
FROM books
GROUP BY genre
ORDER BY total_stock DESC;

-- GROUP BY with WHERE (filter BEFORE grouping)
SELECT genre, COUNT(*) AS count, AVG(price) AS avg_price
FROM books
WHERE price < 20           -- filter rows BEFORE grouping
GROUP BY genre
ORDER BY count DESC;
-- Only counts/averages books under $20 per genre

-- GROUP BY with date parts
SELECT
  YEAR(created_at)  AS year,
  MONTH(created_at) AS month,
  COUNT(*)          AS new_users
FROM users
GROUP BY YEAR(created_at), MONTH(created_at)
ORDER BY year, month;
```

---

### HAVING — Filtering Groups

- **HAVING** filters the **result of GROUP BY** — it's like WHERE but for groups.
- `WHERE` filters individual rows BEFORE grouping.
- `HAVING` filters groups AFTER grouping.
- HAVING can use aggregate functions (COUNT, SUM, AVG, MIN, MAX) — WHERE cannot.

```sql
-- Show only genres with MORE than 2 books
SELECT genre, COUNT(*) AS book_count
FROM books
GROUP BY genre
HAVING COUNT(*) > 2;
-- Returns: fiction(7), programming(5), horror(3), history(3)
-- Excludes: science(2) — only 2 books, not MORE than 2

-- Show only genres with average price ABOVE $15
SELECT genre, ROUND(AVG(price), 2) AS avg_price
FROM books
GROUP BY genre
HAVING AVG(price) > 15;

-- Show genres where total stock is BELOW 200
SELECT genre, SUM(stock) AS total_stock
FROM books
GROUP BY genre
HAVING SUM(stock) < 200;

-- Show users who have placed MORE than 3 orders
SELECT user_id, COUNT(*) AS order_count
FROM orders
WHERE deleted_at IS NULL
GROUP BY user_id
HAVING COUNT(*) > 3
ORDER BY order_count DESC;

-- Show users whose total spending is OVER $200 (VIP customers)
SELECT
  user_id,
  COUNT(*)           AS orders,
  SUM(total)         AS total_spent
FROM orders
WHERE status NOT IN ('cancelled', 'refunded')
  AND deleted_at IS NULL
GROUP BY user_id
HAVING SUM(total) > 200
ORDER BY total_spent DESC;

-- Combining WHERE and HAVING in one query
SELECT
  genre,
  COUNT(*)              AS book_count,
  ROUND(AVG(price), 2) AS avg_price
FROM books
WHERE rating >= 4.5        -- WHERE: filter rows before grouping (only well-rated books)
GROUP BY genre
HAVING COUNT(*) >= 2       -- HAVING: filter groups after grouping (at least 2 books)
ORDER BY avg_price DESC;
```

```
WHERE vs HAVING — The Key Difference:

Query: "For each genre with at least 2 books, show the average price of books over $10"

SELECT genre, COUNT(*), ROUND(AVG(price),2)
FROM books
WHERE price > 10          ← WHERE filters rows FIRST (before grouping)
GROUP BY genre            ← then GROUP BY groups the filtered rows
HAVING COUNT(*) >= 2      ← HAVING filters groups LAST (after grouping)
ORDER BY 3 DESC;

Execution order:
1. FROM books              → access all rows
2. WHERE price > 10        → filter: keep only rows where price > 10
3. GROUP BY genre          → group the remaining rows by genre
4. SELECT (aggregates)     → calculate COUNT, AVG for each group
5. HAVING COUNT(*) >= 2    → filter: keep only groups with 2+ books
6. ORDER BY avg DESC       → sort the final groups
```

---

## Scalar Functions — Transforming Single Values

Scalar functions operate on **one value at a time** — they transform each row individually.

---

### ROUND(), FLOOR(), CEIL()

```sql
-- ROUND(number, decimal_places) — round to N decimal places
SELECT ROUND(18.567, 2);    -- 18.57 (rounds up at 0.005)
SELECT ROUND(18.564, 2);    -- 18.56 (rounds down)
SELECT ROUND(18.5, 0);      -- 19    (rounds to nearest integer)
SELECT ROUND(18.4, 0);      -- 18
SELECT ROUND(18.567, 0);    -- 19
SELECT ROUND(18.567, 1);    -- 18.6

-- Practical uses
SELECT title, ROUND(price, 2) AS price FROM books;
SELECT ROUND(AVG(price), 2) AS avg_price FROM books;
SELECT title, ROUND(price * 0.9, 2) AS discounted FROM books;  -- 10% off

-- FLOOR(number) — always rounds DOWN to the nearest integer (floor of a room)
SELECT FLOOR(18.9);    -- 18  (not 19 — always goes DOWN)
SELECT FLOOR(18.1);    -- 18
SELECT FLOOR(18.0);    -- 18
SELECT FLOOR(-18.1);   -- -19 (goes more negative — still goes DOWN)

-- Practical uses for FLOOR
SELECT FLOOR(rating) AS rating_group FROM books;  -- 4.9 → 4, 4.5 → 4
-- Good for grouping ratings into whole-number tiers

-- CEIL(number) / CEILING(number) — always rounds UP (ceiling of a room)
SELECT CEIL(18.1);    -- 19  (not 18 — always goes UP)
SELECT CEIL(18.9);    -- 19
SELECT CEIL(18.0);    -- 18  (already an integer — stays)
SELECT CEIL(-18.1);   -- -18 (goes less negative — still goes UP)
SELECT CEILING(4.1);  -- 5   (CEILING is alias of CEIL)

-- Practical use for CEIL — page count calculation
SELECT CEIL(COUNT(*) / 10) AS total_pages FROM books;
-- 20 books / 10 per page = CEIL(2.0) = 2 pages
-- If 21 books: CEIL(21/10) = CEIL(2.1) = 3 pages
```

```
ROUND vs FLOOR vs CEIL comparison:

  Value    ROUND(x,0)  FLOOR(x)  CEIL(x)
  ───────────────────────────────────────
   18.1       18          18        19
   18.5       19          18        19
   18.9       19          18        19
   18.0       18          18        18
  -18.1      -18         -19       -18
  -18.9      -19         -19       -18
```

---

### RAND()

- Returns a **random decimal number** between 0 (inclusive) and 1 (exclusive).
- Different result every time it's called.
- Seed with `RAND(n)` to get a repeatable random sequence.

```sql
-- Random number between 0 and 1
SELECT RAND();       -- e.g. 0.7382941283
SELECT RAND();       -- e.g. 0.1847362910  (different each time!)

-- Random integer between 1 and 100
SELECT FLOOR(RAND() * 100) + 1 AS random_number;
-- RAND() * 100 → 0 to 99.999
-- FLOOR()      → 0 to 99
-- +1           → 1 to 100

-- Random integer between min and max:
-- FLOOR(RAND() * (max - min + 1)) + min
SELECT FLOOR(RAND() * (50 - 10 + 1)) + 10 AS random_10_to_50;
-- Returns a random integer from 10 to 50

-- Random book ("book of the day" feature)
SELECT id, title, author, price
FROM books
ORDER BY RAND()
LIMIT 1;
-- Returns a completely random book
-- ⚠️ Slow on large tables (must sort ALL rows randomly)

-- Better approach for large tables: random by ID
SELECT id, title FROM books
WHERE id >= (SELECT FLOOR(RAND() * (SELECT MAX(id) FROM books)) + 1)
ORDER BY id ASC
LIMIT 1;

-- Seed RAND for repeatable results (useful in testing)
SELECT RAND(42);    -- Always returns the same number when seed=42
SELECT RAND(42);    -- Same number again — same seed = same result
SELECT RAND(100);   -- Different number (different seed)
```

---

### NOW(), CURDATE(), CURTIME()

```sql
-- NOW() — current date AND time (DATETIME format)
SELECT NOW();          -- 2026-06-28 14:30:00

-- CURDATE() — current date only (DATE format)
SELECT CURDATE();      -- 2026-06-28

-- CURTIME() — current time only (TIME format)
SELECT CURTIME();      -- 14:30:00

-- Practical uses:
-- Insert current timestamp
INSERT INTO logs (message, created_at) VALUES ('User logged in', NOW());

-- Compare against current time
SELECT * FROM events WHERE event_date >= CURDATE();     -- future events
SELECT * FROM sessions WHERE expires_at < NOW();        -- expired sessions
SELECT * FROM orders WHERE DATE(created_at) = CURDATE(); -- today's orders

-- Calculate how old records are
SELECT id, title, created_at,
  DATEDIFF(NOW(), created_at) AS days_since_added
FROM books
ORDER BY days_since_added DESC;

-- DATEDIFF — difference in days between two dates
SELECT DATEDIFF('2026-12-31', '2026-06-28');  -- 186 days

-- Related datetime functions:
SELECT SYSDATE();        -- like NOW() but evaluated per row (rarely needed)
SELECT UTC_TIMESTAMP();  -- current time in UTC regardless of server timezone
SELECT UNIX_TIMESTAMP(); -- current Unix timestamp (seconds since 1970)

-- Add/subtract time intervals
SELECT NOW() + INTERVAL 7 DAY;      -- 7 days from now
SELECT NOW() - INTERVAL 1 HOUR;     -- 1 hour ago
SELECT NOW() + INTERVAL 1 MONTH;    -- 1 month from now
SELECT NOW() + INTERVAL 1 YEAR;     -- 1 year from now
SELECT DATE_ADD(NOW(), INTERVAL 30 DAY);   -- same as + INTERVAL 30 DAY
SELECT DATE_SUB(NOW(), INTERVAL 3 MONTH); -- 3 months ago
```

---

### YEAR(), MONTH(), DAY()

- Extract specific components from a date or datetime value.
- Very useful for grouping and filtering by time periods.

```sql
-- YEAR() — extract the year
SELECT YEAR('2026-06-28');           -- 2026
SELECT YEAR(created_at) FROM books;  -- extracts year from each row

-- MONTH() — extract the month (1-12)
SELECT MONTH('2026-06-28');          -- 6
SELECT MONTH(NOW());                 -- current month number

-- DAY() / DAYOFMONTH() — extract day of month (1-31)
SELECT DAY('2026-06-28');            -- 28
SELECT DAYOFMONTH('2026-06-28');     -- 28 (same as DAY)

-- DAYOFWEEK() — day of week (1=Sunday, 7=Saturday)
SELECT DAYOFWEEK('2026-06-28');      -- 1 = Sunday

-- DAYNAME() — name of the day
SELECT DAYNAME('2026-06-28');        -- Sunday

-- MONTHNAME() — name of the month
SELECT MONTHNAME('2026-06-28');      -- June

-- WEEKOFYEAR() — week number of the year (1-53)
SELECT WEEKOFYEAR('2026-06-28');     -- 26

-- Practical uses: group orders by year and month
SELECT
  YEAR(created_at)      AS order_year,
  MONTH(created_at)     AS order_month,
  MONTHNAME(created_at) AS month_name,
  COUNT(*)              AS order_count,
  SUM(total)            AS revenue
FROM orders
GROUP BY YEAR(created_at), MONTH(created_at)
ORDER BY order_year ASC, order_month ASC;

-- Filter by specific year
SELECT * FROM orders WHERE YEAR(created_at) = 2026;

-- Filter by specific month (e.g., all June orders regardless of year)
SELECT * FROM orders WHERE MONTH(created_at) = 6;

-- Users registered THIS year
SELECT COUNT(*) FROM users WHERE YEAR(created_at) = YEAR(NOW());

-- Books by year added
SELECT YEAR(created_at) AS year, COUNT(*) AS books_added
FROM books
GROUP BY YEAR(created_at)
ORDER BY year DESC;

-- DATE_FORMAT — format dates as custom strings
SELECT DATE_FORMAT(created_at, '%d %M %Y')  AS formatted FROM books;
-- Returns: "28 June 2026"
SELECT DATE_FORMAT(NOW(), '%Y-%m')           AS year_month;
-- Returns: "2026-06"
SELECT DATE_FORMAT(NOW(), '%W, %d %M %Y')   AS friendly_date;
-- Returns: "Sunday, 28 June 2026"
```

---

### CAST()

- **Converts** a value from one data type to another.
- Essential when types don't match automatically, or when you need to control comparison behavior.

```sql
-- Syntax: CAST(value AS type)

-- Available target types: SIGNED, UNSIGNED, DECIMAL(m,d), CHAR, DATE, DATETIME, TIME, JSON

-- Convert string to integer
SELECT CAST('42' AS SIGNED);       -- 42 (integer)
SELECT CAST('42.99' AS SIGNED);    -- 42 (truncated — not rounded)
SELECT CAST('42abc' AS SIGNED);    -- 42 (reads until non-numeric)
SELECT CAST('abc' AS SIGNED);      -- 0  (no number found)

-- Convert string to decimal
SELECT CAST('42.99' AS DECIMAL(10,2));  -- 42.99

-- Convert number to string
SELECT CAST(42 AS CHAR);           -- '42'
SELECT CAST(3.14 AS CHAR);         -- '3.14'

-- Convert string to date
SELECT CAST('2026-06-28' AS DATE);        -- 2026-06-28
SELECT CAST('2026-06-28 14:30:00' AS DATETIME); -- 2026-06-28 14:30:00

-- Convert to UNSIGNED (remove sign)
SELECT CAST(-42 AS UNSIGNED);      -- 18446744073709551574 (wraps around!)
SELECT CAST(42 AS UNSIGNED);       -- 42

-- Practical: PDO often returns numbers as strings — fix comparison
SELECT id, stock FROM books WHERE CAST(stock AS UNSIGNED) > 100;

-- Practical: sort numbers stored as text correctly
-- If a column is VARCHAR but contains numbers, sorting is wrong:
-- '100', '20', '3' sorts as: '100', '20', '3' (text sort — wrong!)
-- Fix with CAST:
SELECT id, code FROM items ORDER BY CAST(code AS UNSIGNED) ASC;
-- Now sorts: 3, 20, 100 (numeric sort — correct!)

-- CONVERT() is similar to CAST — converts character set OR type
SELECT CONVERT('42', SIGNED);            -- same as CAST('42' AS SIGNED)
SELECT CONVERT(name USING utf8mb4) FROM books; -- convert character set

-- Practical: price stored in paise (cents), display in rupees
SELECT
  title,
  price_in_paise,
  CAST(price_in_paise AS DECIMAL(10,2)) / 100 AS price_in_rupees
FROM products;
```

---

### CONCAT()

- **Joins** two or more strings together into one.
- If ANY argument is NULL, the result is NULL (use CONCAT_WS to handle NULLs).
- Can concatenate columns, strings, and function results.

```sql
-- Basic CONCAT
SELECT CONCAT('Hello', ' ', 'World');      -- 'Hello World'
SELECT CONCAT('PHP', ' is ', 'great!');    -- 'PHP is great!'

-- CONCAT columns
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;
-- 'Phyo Min Paing'

-- CONCAT with other values
SELECT CONCAT(title, ' by ', author) AS book_info FROM books;
-- '1984 by George Orwell'
-- 'Design Patterns by GoF'

-- CONCAT with price formatting
SELECT CONCAT('$', price) AS formatted_price FROM books;
-- '$12.99', '$44.99'

-- CONCAT with functions
SELECT CONCAT(title, ' (', YEAR(created_at), ')') AS title_with_year
FROM books;
-- '1984 (2026)'

-- CONCAT with numbers — MySQL auto-converts numbers to strings
SELECT CONCAT('Stock: ', stock, ' units') AS stock_info FROM books;
-- 'Stock: 100 units'

-- CONCAT NULL issue:
SELECT CONCAT('Hello', NULL, 'World');  -- NULL! (any NULL = whole result NULL)

-- CONCAT_WS — Concat With Separator (handles NULLs gracefully)
-- First argument is the separator, NULLs in remaining args are SKIPPED
SELECT CONCAT_WS(' ', first_name, middle_name, last_name) AS full_name
FROM users;
-- If middle_name is NULL: 'Phyo Paing' (skipped, no double space)
-- If all filled: 'Phyo Min Paing'

SELECT CONCAT_WS(', ', city, state, country) AS address FROM users;
-- 'Thingangyun, Yangon, Myanmar'
-- If state is NULL: 'Thingangyun, Myanmar' (state skipped, no double comma)

SELECT CONCAT_WS(' | ', title, author, genre) AS book_summary FROM books;
-- '1984 | George Orwell | fiction'

-- More string functions
SELECT UPPER('hello world');          -- 'HELLO WORLD'
SELECT LOWER('HELLO WORLD');          -- 'hello world'
SELECT LENGTH('Hello');               -- 5 (byte length)
SELECT CHAR_LENGTH('Hello');          -- 5 (character length — better for Unicode)
SELECT SUBSTRING('Hello World', 1, 5);-- 'Hello' (start pos, length)
SELECT TRIM('  Hello World  ');       -- 'Hello World' (removes leading/trailing spaces)
SELECT LTRIM('  Hello');              -- 'Hello' (left trim)
SELECT RTRIM('Hello  ');              -- 'Hello' (right trim)
SELECT REPLACE('Hello World', 'World', 'MySQL'); -- 'Hello MySQL'
SELECT REVERSE('Hello');              -- 'olleH'
SELECT LEFT('Hello World', 5);        -- 'Hello' (first 5 chars)
SELECT RIGHT('Hello World', 5);       -- 'World' (last 5 chars)
SELECT LPAD('42', 5, '0');            -- '00042' (pad to length 5 with '0' on left)
SELECT RPAD('Hello', 8, '!');         -- 'Hello!!!' (pad to length 8 with '!' on right)
```

---

## AS — Aliases

**Aliases** give a column or table a temporary, more readable name in the query result. They do not change the actual column/table name in the database.

---

### Column Aliases

```sql
-- Syntax: column_name AS alias_name
-- The AS keyword is optional but recommended for clarity

-- Without alias — column name is the function call (ugly)
SELECT COUNT(*), AVG(price), SUM(stock) FROM books;
-- Column headers: COUNT(*) | AVG(price) | SUM(stock)

-- With aliases — meaningful column names
SELECT
  COUNT(*)    AS total_books,
  AVG(price)  AS average_price,
  SUM(stock)  AS total_stock
FROM books;
-- Column headers: total_books | average_price | total_stock

-- Aliases on calculated columns
SELECT
  title,
  price,
  price * 0.9                  AS discounted_price,
  price - (price * 0.9)        AS savings,
  CONCAT('$', price)           AS formatted_price,
  ROUND(price * 0.9, 2)        AS final_price
FROM books;

-- Aliases with spaces (must use backticks or quotes)
SELECT
  COUNT(*)   AS `Total Books`,         -- backtick quotes for spaces
  AVG(price) AS 'Average Price'        -- single quotes also work (non-standard)
FROM books;

-- You CAN use aliases in ORDER BY
SELECT title, price * 0.9 AS discounted
FROM books
ORDER BY discounted ASC;   -- use the alias in ORDER BY ✅

-- You CANNOT use aliases in WHERE (WHERE runs before SELECT)
SELECT title, price * 0.9 AS discounted
FROM books
WHERE discounted < 15;     -- ❌ Error! 'discounted' doesn't exist in WHERE
-- ✅ Fix: repeat the expression
WHERE price * 0.9 < 15;

-- You CAN use aliases in HAVING
SELECT genre, ROUND(AVG(price), 2) AS avg_price
FROM books
GROUP BY genre
HAVING avg_price > 15;    -- ✅ alias works in HAVING in MySQL
```

---

### Table Aliases

Table aliases make queries shorter and more readable, especially with JOINs.

```sql
-- Syntax: table_name AS alias (or just: table_name alias)

-- Without alias (verbose)
SELECT books.id, books.title, books.price
FROM books
WHERE books.genre = 'fiction';

-- With alias (cleaner)
SELECT b.id, b.title, b.price
FROM books AS b
WHERE b.genre = 'fiction';

-- Table aliases are essential for JOINs
SELECT
  u.id,
  u.name           AS customer_name,
  u.email,
  o.id             AS order_id,
  o.total,
  o.status,
  o.created_at     AS order_date
FROM users AS u
JOIN orders AS o ON u.id = o.user_id
WHERE u.status = 'active'
  AND o.deleted_at IS NULL
ORDER BY o.created_at DESC;

-- Self-join (REQUIRES aliases — same table twice)
SELECT
  e.name        AS employee,
  m.name        AS manager
FROM employees AS e
JOIN employees AS m ON e.manager_id = m.id;
-- Both e and m refer to the SAME employees table
-- Aliases distinguish which "copy" we mean

-- AS keyword is optional for tables
FROM books b          -- same as FROM books AS b
FROM users u          -- same as FROM users AS u
FROM orders o         -- same as FROM orders AS o
```

---

## Combining Everything — Real Queries

```sql
-- ─── SALES DASHBOARD QUERY ──────────────────────────────────────────────────
SELECT
  YEAR(o.created_at)        AS year,
  MONTHNAME(o.created_at)   AS month,
  COUNT(o.id)               AS total_orders,
  COUNT(DISTINCT o.user_id) AS unique_customers,
  SUM(o.total)              AS gross_revenue,
  ROUND(AVG(o.total), 2)    AS avg_order_value,
  MIN(o.total)              AS smallest_order,
  MAX(o.total)              AS largest_order
FROM orders AS o
WHERE o.status NOT IN ('cancelled', 'refunded')
  AND o.deleted_at IS NULL
  AND YEAR(o.created_at) = YEAR(NOW())
GROUP BY YEAR(o.created_at), MONTH(o.created_at)
HAVING COUNT(o.id) > 0
ORDER BY YEAR(o.created_at), MONTH(o.created_at);

-- ─── BOOK CATALOG REPORT ────────────────────────────────────────────────────
SELECT
  b.genre,
  COUNT(b.id)                                   AS book_count,
  CONCAT('$', MIN(b.price))                     AS price_from,
  CONCAT('$', MAX(b.price))                     AS price_to,
  CONCAT('$', ROUND(AVG(b.price), 2))           AS avg_price,
  SUM(b.stock)                                  AS total_stock,
  CONCAT('$', ROUND(SUM(b.price * b.stock), 2)) AS inventory_value,
  ROUND(AVG(b.rating), 1)                       AS avg_rating
FROM books AS b
GROUP BY b.genre
HAVING book_count >= 2         -- only genres with at least 2 books
ORDER BY inventory_value DESC;

-- ─── TOP CUSTOMERS LAST 30 DAYS ─────────────────────────────────────────────
SELECT
  u.id,
  CONCAT_WS(' ', u.first_name, u.last_name) AS customer_name,
  u.email,
  COUNT(o.id)                               AS orders_this_month,
  SUM(o.total)                              AS monthly_spend,
  ROUND(AVG(o.total), 2)                    AS avg_order
FROM users AS u
JOIN orders AS o ON u.id = o.user_id
WHERE o.created_at >= NOW() - INTERVAL 30 DAY
  AND o.status NOT IN ('cancelled', 'refunded')
  AND u.deleted_at IS NULL
GROUP BY u.id
HAVING monthly_spend > 100     -- only customers who spent over $100
ORDER BY monthly_spend DESC
LIMIT 10;

-- ─── GENRE PERFORMANCE (books added this year, highly rated) ────────────────
SELECT
  genre,
  COUNT(*)                  AS new_books,
  ROUND(AVG(rating), 2)     AS avg_rating,
  SUM(stock)                AS stock_added,
  ROUND(AVG(price), 2)      AS avg_price
FROM books
WHERE YEAR(created_at) = YEAR(NOW())
  AND rating >= 4.0
GROUP BY genre
HAVING new_books >= 1
ORDER BY avg_rating DESC;

-- ─── RESTOCK PRIORITY LIST ──────────────────────────────────────────────────
SELECT
  id,
  title,
  author,
  genre,
  stock                                            AS current_stock,
  CASE
    WHEN stock = 0           THEN 'OUT OF STOCK'
    WHEN stock BETWEEN 1 AND 20  THEN 'CRITICAL'
    WHEN stock BETWEEN 21 AND 50 THEN 'LOW'
    ELSE                          'OK'
  END                                              AS stock_status,
  CONCAT(ROUND(rating, 1), ' ⭐')               AS rating,
  CONCAT('$', price)                              AS price
FROM books
WHERE stock <= 50
ORDER BY stock ASC, rating DESC;
```

---

## SQL Functions in PHP with PDO

```php
<?php
// ─── FETCH AGGREGATE RESULTS ─────────────────────────────────────────────────

// Single aggregate value
$stmt = $pdo->query("SELECT COUNT(*) AS total FROM books WHERE deleted_at IS NULL");
$total = $stmt->fetchColumn();  // returns just the number
echo "Total books: $total";

// Multiple aggregate values in one query
$stmt = $pdo->query(
    "SELECT
       COUNT(*)              AS total,
       ROUND(AVG(price), 2) AS avg_price,
       MIN(price)            AS min_price,
       MAX(price)            AS max_price,
       SUM(stock)            AS total_stock
     FROM books
     WHERE deleted_at IS NULL"
);
$stats = $stmt->fetch();
echo "Books: {$stats['total']}, Avg: \${$stats['avg_price']}";

// Grouped results
$stmt = $pdo->query(
    "SELECT
       genre,
       COUNT(*)              AS count,
       ROUND(AVG(price), 2) AS avg_price,
       SUM(stock)            AS stock
     FROM books
     GROUP BY genre
     ORDER BY count DESC"
);
$genres = $stmt->fetchAll();
foreach ($genres as $genre) {
    echo "{$genre['genre']}: {$genre['count']} books, avg \${$genre['avg_price']}\n";
}

// ─── DYNAMIC HAVING FILTER ───────────────────────────────────────────────────
function getTopCustomers(PDO $pdo, float $minSpend = 100, int $days = 30): array {
    $stmt = $pdo->prepare(
        "SELECT
           u.id,
           CONCAT_WS(' ', u.first_name, u.last_name) AS name,
           u.email,
           COUNT(o.id)   AS order_count,
           SUM(o.total)  AS total_spent
         FROM users u
         JOIN orders o ON u.id = o.user_id
         WHERE o.created_at >= NOW() - INTERVAL :days DAY
           AND o.status NOT IN ('cancelled', 'refunded')
           AND u.deleted_at IS NULL
         GROUP BY u.id
         HAVING total_spent >= :min_spend
         ORDER BY total_spent DESC
         LIMIT 20"
    );
    $stmt->bindValue(':days',      $days,     PDO::PARAM_INT);
    $stmt->bindValue(':min_spend', $minSpend, PDO::PARAM_STR); // DECIMAL → PARAM_STR
    $stmt->execute();
    return $stmt->fetchAll();
}

$vipCustomers = getTopCustomers($pdo, minSpend: 200, days: 90);

// ─── SEARCH WITH CONCAT ──────────────────────────────────────────────────────
$stmt = $pdo->prepare(
    "SELECT
       id,
       CONCAT(title, ' — ', author)  AS display_name,
       CONCAT('\$', price)            AS formatted_price,
       genre
     FROM books
     WHERE (title LIKE :search OR author LIKE :search)
       AND deleted_at IS NULL
     ORDER BY rating DESC
     LIMIT 10"
);
$stmt->bindValue(':search', '%' . $_GET['q'] . '%');
$stmt->execute();
$results = $stmt->fetchAll();
?>
```

---

## Quick Revision

- **Two types:** Aggregate functions (collapse many rows → one summary) and Scalar functions (transform one value at a time).
- **`COUNT(*)`** — count all rows. **`COUNT(col)`** — count non-NULL values. **`COUNT(DISTINCT col)`** — count unique values.
- **`SUM(col)`** — total of all values. **`AVG(col)`** — arithmetic mean. Both ignore NULLs.
- **`MIN(col)`** — smallest value (numbers, text, dates). **`MAX(col)`** — largest value.
- **`GROUP BY`** — groups rows with same value into one summary row. Every SELECT column must be in GROUP BY or inside an aggregate function.
- **`HAVING`** — filters groups AFTER GROUP BY (like WHERE but for groups). Can use aggregate functions. WHERE cannot.
- **Execution order:** FROM → WHERE (filter rows) → GROUP BY (group) → SELECT (aggregate) → HAVING (filter groups) → ORDER BY → LIMIT.
- **`ROUND(n, d)`** — round to d decimal places. **`FLOOR(n)`** — always round down. **`CEIL(n)`** — always round up.
- **`RAND()`** — random float 0-1. `FLOOR(RAND() * 100) + 1` for random int 1-100. Slow for ORDER BY RAND() on large tables.
- **`NOW()`** — current date+time. **`CURDATE()`** — current date only. **`CURTIME()`** — current time only.
- **`YEAR(date)`**, **`MONTH(date)`**, **`DAY(date)`** — extract parts. **`MONTHNAME()`**, **`DAYNAME()`** — return text names. **`DATE_FORMAT(date, format)`** — custom formatting.
- **`CAST(val AS type)`** — convert data types: SIGNED, UNSIGNED, DECIMAL, CHAR, DATE, DATETIME. Use to fix sorting of numbers stored as text.
- **`CONCAT(a, b, c)`** — joins strings. Returns NULL if any arg is NULL. **`CONCAT_WS(sep, a, b, c)`** — joins with separator, skips NULLs.
- **`AS alias`** — gives a column or table a temporary readable name. Required for expressions and aggregate results. Aliases work in ORDER BY and HAVING but NOT in WHERE.
- **Table aliases** — `FROM books AS b` → use `b.column` everywhere. Essential for JOIN queries and self-joins.