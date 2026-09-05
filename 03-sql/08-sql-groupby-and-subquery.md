# SQL — GROUP BY Deep Dive & Subqueries

**GROUP BY** and **subqueries** are two of the most powerful concepts in SQL that separate beginners from intermediate developers. GROUP BY lets you summarize and analyze data in groups. Subqueries let you use the result of one query inside another — making it possible to answer complex questions that a single flat query cannot.

---

## Table of Contents

1. [GROUP BY — Deep Dive](#group-by--deep-dive)
   - [What GROUP BY Does](#what-group-by-does)
   - [Single Column Grouping](#single-column-grouping)
   - [Multiple Column Grouping](#multiple-column-grouping)
   - [GROUP BY with Aggregate Functions](#group-by-with-aggregate-functions)
   - [GROUP BY with HAVING](#group-by-with-having)
   - [GROUP BY with ORDER BY](#group-by-with-order-by)
   - [GROUP BY with WHERE and HAVING Together](#group-by-with-where-and-having-together)
   - [GROUP BY with Expressions](#group-by-with-expressions)
   - [ROLLUP — Subtotals and Grand Totals](#rollup--subtotals-and-grand-totals)
2. [Subqueries](#subqueries)
   - [What is a Subquery?](#what-is-a-subquery)
   - [Subquery in WHERE](#subquery-in-where)
   - [Subquery with MIN and MAX](#subquery-with-min-and-max)
   - [Subquery with IN](#subquery-with-in)
   - [Subquery with NOT IN](#subquery-with-not-in)
   - [Subquery with EXISTS](#subquery-with-exists)
   - [Subquery in SELECT (Scalar Subquery)](#subquery-in-select-scalar-subquery)
   - [Subquery in FROM (Derived Table)](#subquery-in-from-derived-table)
   - [Correlated Subquery](#correlated-subquery)
   - [Nested Subqueries](#nested-subqueries)
3. [Subquery vs JOIN — When to Use Which](#subquery-vs-join--when-to-use-which)
4. [Real-World Complex Queries](#real-world-complex-queries)
5. [GROUP BY and Subqueries in PHP](#group-by-and-subqueries-in-php)
6. [Common Mistakes](#common-mistakes)
7. [Quick Revision](#quick-revision)

---

## GROUP BY — Deep Dive

---

### What GROUP BY Does

- **GROUP BY** collapses many rows that share the same value in a column into **one summary row**.
- Without GROUP BY, aggregate functions return ONE result for the whole table.
- With GROUP BY, aggregate functions return ONE result PER GROUP.

```
Without GROUP BY:
  SELECT COUNT(*) FROM books;
  → Returns: 20  (one number for the entire table)

With GROUP BY genre:
  SELECT genre, COUNT(*) FROM books GROUP BY genre;
  → Returns: one row PER genre, each with its own count

Visual of what GROUP BY does:

BEFORE GROUP BY:                 AFTER GROUP BY genre:
┌────┬─────────────────┬────────┐  ┌─────────────┬───────┐
│ id │ title           │ genre  │  │ genre       │ count │
├────┼─────────────────┼────────┤  ├─────────────┼───────┤
│  1 │ 1984            │fiction │  │ fiction     │     7 │
│  2 │ Animal Farm     │fiction │  │ programming │     5 │
│  3 │ Harry Potter PS │fiction │  │ horror      │     3 │
│  6 │ The Shining     │horror  │  │ history     │     3 │
│  7 │ It              │horror  │  │ science     │     2 │
│  8 │ Carrie          │horror  │  └─────────────┴───────┘
│ 14 │ Clean Code      │program │
│ ...│ ...             │...     │
└────┴─────────────────┴────────┘
All 20 rows → collapsed into 5 group rows
```

### The GROUP BY Rule

> Every column in your SELECT that is NOT inside an aggregate function MUST appear in GROUP BY.

```sql
-- ❌ WRONG — title is in SELECT but not in GROUP BY (and not aggregated)
SELECT genre, title, COUNT(*) FROM books GROUP BY genre;
-- MySQL may return unpredictable title values (which title from the group?)
-- With ONLY_FULL_GROUP_BY mode (default in MySQL 8) → ERROR

-- ✅ CORRECT — only genre is in SELECT (not aggregated), and it's in GROUP BY
SELECT genre, COUNT(*) FROM books GROUP BY genre;

-- ✅ CORRECT — genre is in GROUP BY, count is aggregated
SELECT genre, COUNT(*) AS count, AVG(price) AS avg_price FROM books GROUP BY genre;

-- ✅ CORRECT — both genre and author in GROUP BY
SELECT genre, author, COUNT(*) AS count FROM books GROUP BY genre, author;
```

---

### Single Column Grouping

```sql
-- Setup: use the bookstore database
USE bookstore;

-- Group by genre — count books per genre
SELECT genre, COUNT(*) AS book_count
FROM books
GROUP BY genre;

-- Group by author — count books per author
SELECT author, COUNT(*) AS books_written
FROM books
GROUP BY author
ORDER BY books_written DESC;

-- Group by rating — how many books at each rating
SELECT rating, COUNT(*) AS count
FROM books
GROUP BY rating
ORDER BY rating DESC;

-- Group by stock level bucket (using CASE inside GROUP BY)
SELECT
  CASE
    WHEN stock = 0            THEN 'Out of Stock'
    WHEN stock BETWEEN 1  AND 50  THEN 'Low'
    WHEN stock BETWEEN 51 AND 100 THEN 'Medium'
    ELSE                          'High'
  END AS stock_level,
  COUNT(*) AS book_count
FROM books
GROUP BY stock_level
ORDER BY book_count DESC;
```

---

### Multiple Column Grouping

- When you GROUP BY multiple columns, MySQL creates a group for each **unique combination** of those columns.

```sql
-- Group by genre AND author — unique genre+author combinations
SELECT genre, author, COUNT(*) AS book_count
FROM books
GROUP BY genre, author
ORDER BY genre ASC, book_count DESC;
```

```
Result:
┌─────────────┬────────────────────┬────────────┐
│ genre       │ author             │ book_count │
├─────────────┼────────────────────┼────────────┤
│ fiction     │ J.K. Rowling       │          3 │ ← 3 Harry Potter books
│ fiction     │ George Orwell      │          2 │ ← 1984 + Animal Farm
│ fiction     │ Harper Lee         │          1 │
│ fiction     │ Paulo Coelho       │          1 │
│ horror      │ Stephen King       │          3 │ ← 3 King books
│ history     │ Yuval Noah Harari  │          3 │ ← 3 Harari books
│ programming │ Kyle Simpson       │          1 │
│ programming │ Robert Martin      │          1 │
│ ...         │ ...                │            │
└─────────────┴────────────────────┴────────────┘
```

```sql
-- Group by year AND month (time-based grouping)
SELECT
  YEAR(created_at)      AS year,
  MONTH(created_at)     AS month,
  MONTHNAME(created_at) AS month_name,
  COUNT(*)              AS books_added
FROM books
GROUP BY YEAR(created_at), MONTH(created_at)
ORDER BY year ASC, month ASC;

-- Group by genre AND price tier
SELECT
  genre,
  CASE
    WHEN price < 15    THEN 'Budget'
    WHEN price < 30    THEN 'Mid-Range'
    ELSE               'Premium'
  END        AS price_tier,
  COUNT(*)   AS count
FROM books
GROUP BY genre, price_tier
ORDER BY genre, price_tier;

-- Orders grouped by user AND status
SELECT user_id, status, COUNT(*) AS count, SUM(total) AS total
FROM orders
GROUP BY user_id, status
ORDER BY user_id, count DESC;
```

---

### GROUP BY with Aggregate Functions

```sql
-- The full power: multiple aggregates per group
SELECT
  genre,
  COUNT(*)                    AS total_books,
  MIN(price)                  AS cheapest,
  MAX(price)                  AS most_expensive,
  ROUND(AVG(price), 2)        AS avg_price,
  SUM(stock)                  AS total_stock,
  ROUND(AVG(rating), 1)       AS avg_rating
FROM books
GROUP BY genre
ORDER BY total_books DESC;
```

```
Result:
┌─────────────┬─────────┬──────────┬────────────────┬───────────┬─────────────┬────────────┐
│ genre       │ total   │ cheapest │ most_expensive │ avg_price │ total_stock │ avg_rating │
├─────────────┼─────────┼──────────┼────────────────┼───────────┼─────────────┼────────────┤
│ fiction     │       7 │     8.99 │          14.99 │     12.71 │         925 │        4.7 │
│ programming │       5 │    24.99 │          44.99 │     35.19 │         270 │        4.6 │
│ horror      │       3 │    11.99 │          16.99 │     14.32 │         155 │        4.5 │
│ history     │       3 │    15.99 │          17.99 │     16.99 │         200 │        4.6 │
│ science     │       2 │    13.99 │          15.99 │     14.99 │         160 │        4.6 │
└─────────────┴─────────┴──────────┴────────────────┴───────────┴─────────────┴────────────┘
```

---

### GROUP BY with HAVING

- **WHERE** filters rows BEFORE grouping.
- **HAVING** filters groups AFTER grouping.

```sql
-- Only genres with MORE than 2 books
SELECT genre, COUNT(*) AS book_count
FROM books
GROUP BY genre
HAVING COUNT(*) > 2;
-- Returns: fiction(7), programming(5), horror(3), history(3)
-- Excludes: science(2) — only 2 books, not more than 2

-- Only genres where average price is ABOVE $15
SELECT genre, ROUND(AVG(price), 2) AS avg_price
FROM books
GROUP BY genre
HAVING AVG(price) > 15;

-- Only authors who wrote MORE than 1 book
SELECT author, COUNT(*) AS book_count
FROM books
GROUP BY author
HAVING COUNT(*) > 1
ORDER BY book_count DESC;

-- HAVING with alias (works in MySQL)
SELECT genre, COUNT(*) AS cnt, ROUND(AVG(price), 2) AS avg_p
FROM books
GROUP BY genre
HAVING cnt > 2 AND avg_p < 20;
-- cnt and avg_p are aliases — MySQL allows these in HAVING

-- VIP customers: ordered more than 3 times
SELECT user_id, COUNT(*) AS order_count, SUM(total) AS total_spent
FROM orders
WHERE deleted_at IS NULL
GROUP BY user_id
HAVING COUNT(*) >= 3
ORDER BY total_spent DESC;
```

---

### GROUP BY with ORDER BY

ORDER BY comes AFTER GROUP BY and sorts the final grouped result.

```sql
-- Sort genres by number of books (most books first)
SELECT genre, COUNT(*) AS count
FROM books
GROUP BY genre
ORDER BY count DESC;

-- Sort by average price (most expensive first)
SELECT genre, ROUND(AVG(price), 2) AS avg_price
FROM books
GROUP BY genre
ORDER BY avg_price DESC;

-- Sort by multiple columns after grouping
SELECT
  genre,
  COUNT(*)              AS count,
  ROUND(AVG(price), 2) AS avg_price
FROM books
GROUP BY genre
ORDER BY count DESC, avg_price ASC;
-- Sort by count first, then by price for ties
```

---

### GROUP BY with WHERE and HAVING Together

```sql
-- WHERE filters individual rows BEFORE grouping
-- HAVING filters groups AFTER grouping
-- Both can be used in the same query

-- Among books rated 4.5+, show genres with more than 1 such book
SELECT
  genre,
  COUNT(*) AS highly_rated_count,
  ROUND(AVG(price), 2) AS avg_price
FROM books
WHERE rating >= 4.5           -- ← WHERE filters rows first (before grouping)
GROUP BY genre                -- ← then group the filtered rows
HAVING COUNT(*) > 1           -- ← HAVING filters groups last (after grouping)
ORDER BY highly_rated_count DESC;

-- Among orders NOT cancelled, show users who spent over $100 total
SELECT
  user_id,
  COUNT(*)     AS order_count,
  SUM(total)   AS total_spent
FROM orders
WHERE status NOT IN ('cancelled', 'refunded')   -- WHERE filters before GROUP
  AND deleted_at IS NULL
GROUP BY user_id
HAVING SUM(total) > 100                          -- HAVING filters after GROUP
ORDER BY total_spent DESC;
```

```
Execution order to remember:

1. FROM books              → get all rows
2. WHERE rating >= 4.5     → filter: keep only rows where rating >= 4.5
3. GROUP BY genre          → group the remaining rows by genre
4. COUNT(*), AVG(price)    → calculate aggregates for each group
5. HAVING COUNT(*) > 1     → filter: keep only groups with count > 1
6. ORDER BY                → sort the final groups
7. LIMIT                   → cut to N rows
```

---

### GROUP BY with Expressions

You can GROUP BY the result of a function or expression.

```sql
-- Group by year
SELECT YEAR(created_at) AS year, COUNT(*) AS books_added
FROM books
GROUP BY YEAR(created_at)
ORDER BY year;

-- Group by price tier (expression in GROUP BY)
SELECT
  CASE
    WHEN price < 15    THEN 'Budget (< $15)'
    WHEN price < 30    THEN 'Mid-Range ($15-$29)'
    ELSE                    'Premium ($30+)'
  END AS price_tier,
  COUNT(*) AS count,
  ROUND(AVG(rating), 2) AS avg_rating
FROM books
GROUP BY price_tier
ORDER BY MIN(price);

-- Group by first letter of title (alphabetical grouping)
SELECT
  LEFT(title, 1) AS first_letter,
  COUNT(*)       AS count
FROM books
GROUP BY LEFT(title, 1)
ORDER BY first_letter;

-- Group orders by day of week (which day do we get most orders?)
SELECT
  DAYNAME(created_at)  AS day_of_week,
  COUNT(*)             AS order_count,
  SUM(total)           AS revenue
FROM orders
GROUP BY DAYNAME(created_at), DAYOFWEEK(created_at)
ORDER BY DAYOFWEEK(created_at);
```

---

### ROLLUP — Subtotals and Grand Totals

`WITH ROLLUP` adds subtotal and grand total rows automatically.

```sql
-- Genre summary with grand total at the bottom
SELECT
  COALESCE(genre, 'GRAND TOTAL') AS genre,
  COUNT(*)                        AS total_books,
  SUM(stock)                      AS total_stock,
  ROUND(AVG(price), 2)            AS avg_price
FROM books
GROUP BY genre WITH ROLLUP;
```

```
Result:
┌─────────────┬─────────────┬─────────────┬───────────┐
│ genre       │ total_books │ total_stock │ avg_price │
├─────────────┼─────────────┼─────────────┼───────────┤
│ fiction     │           7 │         925 │     12.71 │
│ history     │           3 │         200 │     16.99 │
│ horror      │           3 │         155 │     14.32 │
│ programming │           5 │         270 │     35.19 │
│ science     │           2 │         160 │     14.99 │
├─────────────┼─────────────┼─────────────┼───────────┤
│ GRAND TOTAL │          20 │        1710 │     18.74 │ ← auto-added!
└─────────────┴─────────────┴─────────────┴───────────┘
```

---

## Subqueries

---

### What is a Subquery?

- A **subquery** (also called a nested query or inner query) is a **SELECT statement written inside another SQL statement**.
- The inner query (subquery) runs FIRST and its result is used by the outer query.
- Subqueries are wrapped in parentheses `( )`.
- They can appear in WHERE, SELECT, FROM, and HAVING clauses.

```sql
-- Structure of a subquery:
SELECT columns
FROM table
WHERE column = (SELECT column FROM other_table WHERE condition);
--              ↑─────────────────────────────────────────────↑
--              This is the SUBQUERY — runs first

-- The outer query uses the subquery's result as a value

-- Example: find the student with the least money
SELECT * FROM students WHERE money = (SELECT MIN(money) FROM students);
-- Step 1: inner query runs → SELECT MIN(money) FROM students → returns 500
-- Step 2: outer query runs → SELECT * FROM students WHERE money = 500
```

```
Think of it like a variable in PHP:

PHP:
  $minMoney = getMinMoneyFromDB();    // inner query result
  $students = getStudentsWith($minMoney); // outer query uses it

SQL:
  SELECT * FROM students
  WHERE money = (SELECT MIN(money) FROM students);
  -- same concept but written as one SQL statement
```

---

### Subquery in WHERE

The most common use — the subquery returns a single value that the outer query compares against.

```sql
-- ─── THE CLASSIC EXAMPLES ────────────────────────────────────────────────────

-- Find the book with the LOWEST price
SELECT id, title, author, price
FROM books
WHERE price = (SELECT MIN(price) FROM books);
-- Inner: SELECT MIN(price) FROM books → 8.99
-- Outer: SELECT ... WHERE price = 8.99
-- Result: Animal Farm — $8.99

-- Find the book with the HIGHEST price
SELECT id, title, author, price
FROM books
WHERE price = (SELECT MAX(price) FROM books);
-- Result: Design Patterns — $44.99

-- Find the best-rated book(s)
SELECT id, title, rating
FROM books
WHERE rating = (SELECT MAX(rating) FROM books);
-- Returns ALL books tied for the top rating (could be multiple)

-- Find the student with the least money (the exact example from your question)
SELECT * FROM students
WHERE money = (SELECT MIN(money) FROM students);
-- Inner query finds the minimum money value
-- Outer query returns ALL students who have exactly that amount

-- Find the cheapest book in fiction genre
SELECT id, title, price
FROM books
WHERE genre = 'fiction'
  AND price = (SELECT MIN(price) FROM books WHERE genre = 'fiction');
-- Note: the subquery also filters by fiction

-- Find users who joined after the average join date
SELECT id, name, created_at
FROM users
WHERE created_at > (SELECT AVG(created_at) FROM users);
-- AVG on timestamps returns a numeric average timestamp

-- Find books priced ABOVE the store's average
SELECT id, title, price
FROM books
WHERE price > (SELECT AVG(price) FROM books)
ORDER BY price ASC;
-- Inner: AVG(price) → 18.74
-- Outer: WHERE price > 18.74

-- Find books priced BELOW the average of THEIR OWN GENRE
-- (covered in Correlated Subqueries section below)

-- Find orders above the user's own average order value
SELECT id, user_id, total
FROM orders o
WHERE total > (
  SELECT AVG(total)
  FROM orders
  WHERE user_id = o.user_id   -- correlated: references outer query's user_id
);
```

---

### Subquery with MIN and MAX

```sql
-- ─── MIN SUBQUERY PATTERNS ────────────────────────────────────────────────────

-- Book with minimum price
SELECT id, title, author, price, genre
FROM books
WHERE price = (SELECT MIN(price) FROM books);

-- Book with minimum price IN EACH genre (need correlated subquery)
SELECT b.id, b.title, b.genre, b.price
FROM books AS b
WHERE b.price = (
  SELECT MIN(b2.price)
  FROM books AS b2
  WHERE b2.genre = b.genre   -- match the genre of the outer row
);
-- Returns the cheapest book from EACH genre

-- User with the most orders
SELECT id, name
FROM users
WHERE id = (
  SELECT user_id
  FROM orders
  WHERE deleted_at IS NULL
  GROUP BY user_id
  ORDER BY COUNT(*) DESC
  LIMIT 1
);

-- Product with lowest stock (needs restocking most urgently)
SELECT id, title, stock
FROM books
WHERE stock = (SELECT MIN(stock) FROM books WHERE stock > 0);
-- Lowest stock that isn't zero

-- ─── MAX SUBQUERY PATTERNS ────────────────────────────────────────────────────

-- Most expensive book in the programming genre
SELECT id, title, price
FROM books
WHERE genre = 'programming'
  AND price = (SELECT MAX(price) FROM books WHERE genre = 'programming');

-- The user who spent the most
SELECT id, name, email
FROM users
WHERE id = (
  SELECT user_id
  FROM orders
  WHERE status = 'delivered' AND deleted_at IS NULL
  GROUP BY user_id
  ORDER BY SUM(total) DESC
  LIMIT 1
);

-- The month with the highest sales
SELECT MONTHNAME(created_at) AS best_month, SUM(total) AS revenue
FROM orders
WHERE YEAR(created_at) = YEAR(NOW())
  AND status NOT IN ('cancelled', 'refunded')
GROUP BY MONTH(created_at)
HAVING SUM(total) = (
  SELECT MAX(monthly_total)
  FROM (
    SELECT SUM(total) AS monthly_total
    FROM orders
    WHERE YEAR(created_at) = YEAR(NOW())
      AND status NOT IN ('cancelled', 'refunded')
    GROUP BY MONTH(created_at)
  ) AS monthly_totals
);
```

---

### Subquery with IN

When the subquery returns **multiple values** (a list), use `IN` instead of `=`.

```sql
-- Find all books by authors who have written more than one book
SELECT id, title, author, genre
FROM books
WHERE author IN (
  SELECT author           -- inner query returns a list of authors
  FROM books
  GROUP BY author
  HAVING COUNT(*) > 1    -- only authors with more than 1 book
)
ORDER BY author, title;
-- Inner returns: ['J.K. Rowling', 'Stephen King', 'George Orwell', 'Yuval Noah Harari', 'Stephen Hawking']
-- Outer returns: all books by those authors

-- Find users who have placed at least one order
SELECT id, name, email
FROM users
WHERE id IN (
  SELECT DISTINCT user_id
  FROM orders
  WHERE deleted_at IS NULL
);

-- Find books in genres that have more than 3 books
SELECT id, title, genre, price
FROM books
WHERE genre IN (
  SELECT genre
  FROM books
  GROUP BY genre
  HAVING COUNT(*) > 3
)
ORDER BY genre, price;
-- Inner: genres with > 3 books → [fiction, programming]
-- Outer: all books in those genres

-- Find users who ordered a specific product
SELECT DISTINCT u.id, u.name, u.email
FROM users AS u
WHERE u.id IN (
  SELECT o.user_id
  FROM orders AS o
  JOIN order_items AS oi ON o.id = oi.order_id
  WHERE oi.product_id = 5
    AND o.deleted_at IS NULL
);
-- Who bought book #5?

-- Find genres that contain at least one book under $10
SELECT id, title, genre, price
FROM books
WHERE genre IN (
  SELECT genre
  FROM books
  WHERE price < 10
);
-- Returns ALL books in genres that have at least one cheap book
```

---

### Subquery with NOT IN

Finds rows that are NOT in the subquery's result.

```sql
-- Find users who have NEVER placed an order
SELECT id, name, email, created_at
FROM users
WHERE id NOT IN (
  SELECT DISTINCT user_id
  FROM orders
  WHERE deleted_at IS NULL
    AND user_id IS NOT NULL  -- ← IMPORTANT! See warning below
)
AND deleted_at IS NULL
ORDER BY created_at DESC;

-- Find books that have NEVER been ordered
SELECT id, title, author, price
FROM books
WHERE id NOT IN (
  SELECT DISTINCT product_id
  FROM order_items
  WHERE product_id IS NOT NULL   -- ← always add IS NOT NULL!
);

-- Find genres with NO programming books
SELECT DISTINCT genre
FROM books
WHERE genre NOT IN (
  SELECT genre FROM books WHERE author = 'Stephen King'
);
-- Genres that have no Stephen King books
```

> ⚠️ **Critical Warning: NOT IN + NULL = disaster**

```sql
-- If the subquery returns even ONE NULL value, NOT IN returns NO rows at all!

-- This might return 0 rows even when you expect results:
SELECT * FROM users
WHERE id NOT IN (SELECT user_id FROM orders);
-- If any order has user_id = NULL → returns 0 rows!
-- Why? Because "id NOT IN (..., NULL, ...)" is unknown for every id

-- ✅ Always add WHERE col IS NOT NULL to the subquery:
SELECT * FROM users
WHERE id NOT IN (
  SELECT user_id FROM orders WHERE user_id IS NOT NULL
);

-- OR: use NOT EXISTS instead (naturally handles NULLs):
SELECT u.id, u.name
FROM users AS u
WHERE NOT EXISTS (
  SELECT 1 FROM orders AS o
  WHERE o.user_id = u.id AND o.deleted_at IS NULL
);
```

---

### Subquery with EXISTS

- `EXISTS` checks if the subquery returns **any rows** (TRUE if at least one row, FALSE if no rows).
- More efficient than IN for large datasets.
- Naturally handles NULLs (unlike NOT IN).

```sql
-- Find users who HAVE placed an order (EXISTS)
SELECT u.id, u.name, u.email
FROM users AS u
WHERE EXISTS (
  SELECT 1               -- "1" because we only care IF rows exist, not what they are
  FROM orders AS o
  WHERE o.user_id = u.id
    AND o.deleted_at IS NULL
);
-- For each user, the inner query checks: "does any order exist for this user?"
-- If YES → EXISTS is TRUE → include this user
-- If NO  → EXISTS is FALSE → exclude this user

-- Find users who have NEVER ordered (NOT EXISTS)
SELECT u.id, u.name, u.email
FROM users AS u
WHERE NOT EXISTS (
  SELECT 1
  FROM orders AS o
  WHERE o.user_id = u.id
    AND o.deleted_at IS NULL
);

-- Find books that have been ordered at least once
SELECT b.id, b.title, b.price
FROM books AS b
WHERE EXISTS (
  SELECT 1
  FROM order_items AS oi
  WHERE oi.product_id = b.id
);

-- Find genres that have at least one book under $10
SELECT DISTINCT genre
FROM books AS b
WHERE EXISTS (
  SELECT 1
  FROM books AS inner_b
  WHERE inner_b.genre = b.genre
    AND inner_b.price < 10
);
```

```
EXISTS vs IN for performance:
  Small result sets → either is fine
  Large result sets → EXISTS is usually faster
  NULLs in data    → EXISTS is safer (no NULL surprises)

EXISTS stops as soon as it finds ONE matching row.
IN collects ALL matching values then checks against them.
```

---

### Subquery in SELECT (Scalar Subquery)

A subquery inside the SELECT clause returns **one value per row** — it runs once for each row of the outer query.

```sql
-- Show each book's price AND the store's average price alongside it
SELECT
  id,
  title,
  price,
  (SELECT ROUND(AVG(price), 2) FROM books) AS store_avg_price,
  price - (SELECT ROUND(AVG(price), 2) FROM books) AS diff_from_avg
FROM books
ORDER BY diff_from_avg DESC;
-- Each row shows: this book's price, the overall average, and the difference

-- Show each book's price AND its genre's average price
SELECT
  b.id,
  b.title,
  b.genre,
  b.price,
  (
    SELECT ROUND(AVG(b2.price), 2)
    FROM books AS b2
    WHERE b2.genre = b.genre   -- correlated: matches current row's genre
  ) AS genre_avg_price
FROM books AS b
ORDER BY b.genre, b.price;

-- Show each user and how many orders they have
SELECT
  u.id,
  u.name,
  u.email,
  (SELECT COUNT(*) FROM orders WHERE user_id = u.id AND deleted_at IS NULL)
    AS order_count
FROM users AS u
WHERE u.deleted_at IS NULL
ORDER BY order_count DESC;

-- Show the total number of books alongside each book
SELECT
  id,
  title,
  price,
  (SELECT COUNT(*) FROM books) AS total_in_store
FROM books
LIMIT 5;
-- Each row shows: book info + "20" (total books) — same for all rows
```

---

### Subquery in FROM (Derived Table)

A subquery in the FROM clause creates a **temporary table** (called a derived table or inline view) that the outer query treats like a real table.

```sql
-- The subquery creates a temporary result that FROM treats as a table
SELECT * FROM (subquery) AS alias_name;
-- The alias is REQUIRED when using a subquery in FROM

-- Find genres with above-average book counts
SELECT genre, book_count
FROM (
  SELECT genre, COUNT(*) AS book_count
  FROM books
  GROUP BY genre
) AS genre_counts               -- ← alias is mandatory
WHERE book_count > (
  SELECT AVG(book_count)
  FROM (
    SELECT COUNT(*) AS book_count FROM books GROUP BY genre
  ) AS inner_counts
);

-- Calculate statistics on grouped data
SELECT
  AVG(books_per_author) AS avg_books_per_author,
  MAX(books_per_author) AS most_prolific_author_count,
  MIN(books_per_author) AS least_prolific_count
FROM (
  SELECT author, COUNT(*) AS books_per_author
  FROM books
  GROUP BY author
) AS author_counts;
-- Can't do AVG(COUNT(*)) directly — put it in a subquery first!

-- Top 3 authors by book count, then get all their books
SELECT b.title, b.price, b.genre, top_authors.author
FROM books AS b
JOIN (
  SELECT author, COUNT(*) AS cnt
  FROM books
  GROUP BY author
  ORDER BY cnt DESC
  LIMIT 3
) AS top_authors ON b.author = top_authors.author
ORDER BY top_authors.cnt DESC, b.title;

-- Pagination with filtered data: page 2 of books above average price
SELECT *
FROM (
  SELECT id, title, price, ROW_NUMBER() OVER (ORDER BY price DESC) AS row_num
  FROM books
  WHERE price > (SELECT AVG(price) FROM books)
) AS above_avg
WHERE row_num BETWEEN 6 AND 10;  -- rows 6-10 (page 2 of 5-per-page)
```

---

### Correlated Subquery

A **correlated subquery** references a column from the OUTER query — it runs once for EACH row of the outer query.

```sql
-- Find books priced BELOW their genre's average
-- (For each book, compare its price against its own genre's average)
SELECT b.id, b.title, b.genre, b.price,
  ROUND((
    SELECT AVG(b2.price)
    FROM books AS b2
    WHERE b2.genre = b.genre   -- ← references b.genre from outer query
  ), 2) AS genre_avg
FROM books AS b
WHERE b.price < (
  SELECT AVG(b2.price)
  FROM books AS b2
  WHERE b2.genre = b.genre     -- ← correlates with outer query's current row
)
ORDER BY b.genre, b.price;
-- Returns books that are cheaper than their genre's average

-- For each user, show their latest order
SELECT u.id, u.name,
  (SELECT MAX(o.created_at) FROM orders AS o WHERE o.user_id = u.id) AS last_order_date,
  (SELECT o2.total FROM orders AS o2 WHERE o2.user_id = u.id
     ORDER BY o2.created_at DESC LIMIT 1) AS last_order_total
FROM users AS u
WHERE u.deleted_at IS NULL;

-- Find the most expensive book IN EACH genre (correlated)
SELECT b.id, b.title, b.genre, b.price
FROM books AS b
WHERE b.price = (
  SELECT MAX(b2.price)
  FROM books AS b2
  WHERE b2.genre = b.genre     -- correlates: same genre as current outer row
);
-- Returns the priciest book from EACH genre (all genres represented)
```

```
How a correlated subquery executes:

Outer query reads row 1: genre = 'fiction', price = 12.99
  → Inner query runs: SELECT AVG(price) FROM books WHERE genre = 'fiction' → 12.71
  → Comparison: 12.99 < 12.71? NO → row excluded

Outer query reads row 2: genre = 'fiction', price = 8.99
  → Inner query runs: SELECT AVG(price) FROM books WHERE genre = 'fiction' → 12.71
  → Comparison: 8.99 < 12.71? YES → row included

Outer query reads row 3: genre = 'programming', price = 35.99
  → Inner query runs: SELECT AVG(price) FROM books WHERE genre = 'programming' → 35.19
  → Comparison: 35.99 < 35.19? NO → row excluded

...runs for EVERY row in the outer query
```

---

### Nested Subqueries

Subqueries can be nested inside other subqueries — but keep it to 2-3 levels maximum for readability.

```sql
-- Find authors who wrote books in genres that have above-average book counts
SELECT DISTINCT author
FROM books
WHERE genre IN (
  -- Level 2: genres with above-average book count
  SELECT genre
  FROM books
  GROUP BY genre
  HAVING COUNT(*) > (
    -- Level 3: the average book count per genre
    SELECT AVG(genre_count)
    FROM (
      -- Level 4: count per genre
      SELECT genre, COUNT(*) AS genre_count
      FROM books
      GROUP BY genre
    ) AS counts
  )
);

-- Step by step:
-- L4: count per genre: fiction=7, programming=5, horror=3, history=3, science=2
-- L3: average of those counts: (7+5+3+3+2)/5 = 4
-- L2: genres above avg(4): fiction(7), programming(5)
-- L1: authors who wrote fiction or programming books
```

---

## Subquery vs JOIN — When to Use Which

```
SUBQUERY: Better when...
  ✅ You need a single value for comparison (MIN, MAX, AVG)
  ✅ The question is "does something exist?" (EXISTS)
  ✅ You want to filter by aggregated data (users with > 3 orders)
  ✅ The logic reads naturally as "find rows where X is Y"
  ✅ The subquery returns data from a completely different table
  ✅ You're finding rows that DON'T exist in another table (NOT EXISTS)

JOIN: Better when...
  ✅ You need COLUMNS from multiple tables in your result
  ✅ Performance matters on large datasets (JOINs are usually faster)
  ✅ Many-to-many relationships
  ✅ You need all matched rows (not just whether they match)

Example: "Find users and their order count"
  -- JOIN approach (returns users WITH order data):
  SELECT u.name, COUNT(o.id) AS order_count
  FROM users u LEFT JOIN orders o ON u.id = o.user_id
  GROUP BY u.id;

  -- Subquery approach (also works, but slightly different):
  SELECT name,
    (SELECT COUNT(*) FROM orders WHERE user_id = u.id) AS order_count
  FROM users u;
```

---

## Real-World Complex Queries

```sql
-- ─── 1. FIND THE CHEAPEST BOOK PER GENRE ────────────────────────────────────
SELECT b.id, b.title, b.genre, b.price
FROM books AS b
WHERE b.price = (
  SELECT MIN(price) FROM books WHERE genre = b.genre
)
ORDER BY b.genre;

-- ─── 2. BOOKS MORE EXPENSIVE THAN THEIR GENRE AVERAGE ───────────────────────
SELECT b.title, b.genre, b.price,
  ROUND((SELECT AVG(price) FROM books WHERE genre = b.genre), 2) AS genre_avg
FROM books b
WHERE b.price > (SELECT AVG(price) FROM books WHERE genre = b.genre)
ORDER BY b.genre, b.price DESC;

-- ─── 3. USERS WHO HAVEN'T ORDERED RECENTLY (last 30 days) ───────────────────
SELECT u.id, u.name, u.email,
  (SELECT MAX(created_at) FROM orders WHERE user_id = u.id) AS last_order_date
FROM users u
WHERE u.status = 'active'
  AND u.deleted_at IS NULL
  AND NOT EXISTS (
    SELECT 1 FROM orders o
    WHERE o.user_id = u.id
      AND o.created_at >= NOW() - INTERVAL 30 DAY
      AND o.deleted_at IS NULL
  )
ORDER BY last_order_date ASC;

-- ─── 4. TOP GENRE BY REVENUE THIS MONTH ─────────────────────────────────────
SELECT b.genre, SUM(oi.qty * oi.price) AS monthly_revenue
FROM order_items AS oi
JOIN books AS b ON oi.product_id = b.id
JOIN orders AS o ON oi.order_id = o.id
WHERE MONTH(o.created_at) = MONTH(NOW())
  AND YEAR(o.created_at)  = YEAR(NOW())
  AND o.status NOT IN ('cancelled', 'refunded')
GROUP BY b.genre
ORDER BY monthly_revenue DESC
LIMIT 1;

-- ─── 5. GENRES WITH ABOVE-AVERAGE RATING AND BELOW-AVERAGE PRICE ────────────
SELECT genre,
  ROUND(AVG(rating), 2) AS avg_rating,
  ROUND(AVG(price), 2)  AS avg_price
FROM books
GROUP BY genre
HAVING AVG(rating) > (SELECT AVG(rating) FROM books)
   AND AVG(price)  < (SELECT AVG(price)  FROM books)
ORDER BY avg_rating DESC;

-- ─── 6. RANK BOOKS WITHIN THEIR GENRE BY PRICE ──────────────────────────────
SELECT
  b.title,
  b.genre,
  b.price,
  (
    SELECT COUNT(*)
    FROM books AS b2
    WHERE b2.genre = b.genre AND b2.price <= b.price
  ) AS price_rank_in_genre   -- 1 = cheapest in genre
FROM books AS b
ORDER BY b.genre, price_rank_in_genre;
```

---

## GROUP BY and Subqueries in PHP

```php
<?php
// ─── GROUP BY QUERY ──────────────────────────────────────────────────────────
function getGenreStats(PDO $pdo): array {
    $stmt = $pdo->query(
        "SELECT
           genre,
           COUNT(*)              AS total_books,
           ROUND(AVG(price), 2) AS avg_price,
           MIN(price)            AS min_price,
           MAX(price)            AS max_price,
           SUM(stock)            AS total_stock
         FROM books
         GROUP BY genre
         HAVING COUNT(*) > 0
         ORDER BY total_books DESC"
    );
    return $stmt->fetchAll();
}

$stats = getGenreStats($pdo);
foreach ($stats as $row) {
    echo "{$row['genre']}: {$row['total_books']} books, avg \${$row['avg_price']}\n";
}


// ─── SUBQUERY — cheapest book overall ─────────────────────────────────────────
function getCheapestBook(PDO $pdo): array {
    $stmt = $pdo->query(
        "SELECT id, title, author, price
         FROM books
         WHERE price = (SELECT MIN(price) FROM books)"
    );
    return $stmt->fetch();
}

$cheapest = getCheapestBook($pdo);
echo "Cheapest: {$cheapest['title']} at \${$cheapest['price']}";


// ─── SUBQUERY — books above average price ─────────────────────────────────────
function getBooksAboveAverage(PDO $pdo): array {
    $stmt = $pdo->query(
        "SELECT id, title, price,
           (SELECT ROUND(AVG(price), 2) FROM books) AS store_avg
         FROM books
         WHERE price > (SELECT AVG(price) FROM books)
         ORDER BY price ASC"
    );
    return $stmt->fetchAll();
}


// ─── SUBQUERY with parameter ─────────────────────────────────────────────────
function getCheapestInGenre(PDO $pdo, string $genre): ?array {
    $stmt = $pdo->prepare(
        "SELECT id, title, price
         FROM books
         WHERE genre = :genre
           AND price = (
             SELECT MIN(price) FROM books WHERE genre = :genre
           )"
    );
    $stmt->execute([':genre' => $genre]);
    return $stmt->fetch() ?: null;
}

$cheapFiction = getCheapestInGenre($pdo, 'fiction');
if ($cheapFiction) {
    echo "Cheapest fiction: {$cheapFiction['title']} at \${$cheapFiction['price']}";
}


// ─── EXISTS SUBQUERY ──────────────────────────────────────────────────────────
function getUsersWithoutOrders(PDO $pdo): array {
    $stmt = $pdo->query(
        "SELECT id, name, email, created_at
         FROM users
         WHERE deleted_at IS NULL
           AND NOT EXISTS (
             SELECT 1 FROM orders
             WHERE user_id = users.id AND deleted_at IS NULL
           )
         ORDER BY created_at DESC"
    );
    return $stmt->fetchAll();
}

$newUsers = getUsersWithoutOrders($pdo);
echo count($newUsers) . " users have never ordered\n";


// ─── DERIVED TABLE (subquery in FROM) ─────────────────────────────────────────
function getAuthorStats(PDO $pdo): array {
    $stmt = $pdo->query(
        "SELECT
           author_stats.author,
           author_stats.book_count,
           author_stats.avg_price,
           author_stats.avg_rating
         FROM (
           SELECT
             author,
             COUNT(*)              AS book_count,
             ROUND(AVG(price), 2) AS avg_price,
             ROUND(AVG(rating), 1) AS avg_rating
           FROM books
           GROUP BY author
         ) AS author_stats
         WHERE author_stats.book_count > 1
         ORDER BY author_stats.book_count DESC"
    );
    return $stmt->fetchAll();
}
?>
```

---

## Common Mistakes

```sql
-- ❌ MISTAKE 1: Non-aggregated column not in GROUP BY
SELECT genre, title, COUNT(*) FROM books GROUP BY genre;
-- "title" is in SELECT but not in GROUP BY and not aggregated
-- In MySQL 8 ONLY_FULL_GROUP_BY mode → ERROR
-- ✅ Fix:
SELECT genre, COUNT(*) FROM books GROUP BY genre;
-- OR include title in GROUP BY if you need it:
SELECT genre, title, COUNT(*) FROM books GROUP BY genre, title;

────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 2: Using WHERE instead of HAVING for aggregate conditions
SELECT genre, COUNT(*) FROM books WHERE COUNT(*) > 2 GROUP BY genre;
-- ERROR: Can't use aggregate functions in WHERE
-- ✅ Fix:
SELECT genre, COUNT(*) FROM books GROUP BY genre HAVING COUNT(*) > 2;

────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 3: Subquery returns multiple rows when = is used
SELECT * FROM books WHERE price = (SELECT price FROM books WHERE genre = 'fiction');
-- ERROR: Subquery returns more than 1 row
-- ✅ Fix: use IN for multiple results
SELECT * FROM books WHERE price IN (SELECT price FROM books WHERE genre = 'fiction');
-- OR use LIMIT 1 to get one value
SELECT * FROM books WHERE price = (SELECT MIN(price) FROM books WHERE genre = 'fiction');

────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 4: NOT IN with possible NULLs
SELECT * FROM users WHERE id NOT IN (SELECT user_id FROM orders);
-- If any order has user_id = NULL → returns 0 rows!
-- ✅ Fix: always add IS NOT NULL
SELECT * FROM users WHERE id NOT IN (
  SELECT user_id FROM orders WHERE user_id IS NOT NULL
);
-- OR use NOT EXISTS instead

────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 5: Forgetting alias for derived table
SELECT * FROM (SELECT genre, COUNT(*) FROM books GROUP BY genre);
-- ERROR: Every derived table must have its own alias
-- ✅ Fix:
SELECT * FROM (SELECT genre, COUNT(*) AS cnt FROM books GROUP BY genre) AS genre_counts;

────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 6: Correlated subquery in WHERE using alias from SELECT
SELECT id, title, price,
  ROUND(AVG(price) OVER (PARTITION BY genre), 2) AS genre_avg
FROM books
WHERE price < genre_avg;    -- ERROR: alias from SELECT not available in WHERE
-- ✅ Fix: use a subquery or window function differently
SELECT * FROM (
  SELECT id, title, price, genre,
    ROUND(AVG(price) OVER (PARTITION BY genre), 2) AS genre_avg
  FROM books
) AS books_with_avg
WHERE price < genre_avg;
```

---

## Quick Revision

- **GROUP BY** collapses rows with the same value into one summary row. Every SELECT column must be in GROUP BY or inside an aggregate function.
- **Single column GROUP BY:** `GROUP BY genre` — one result per unique genre.
- **Multiple column GROUP BY:** `GROUP BY genre, author` — one result per unique genre+author combination.
- **Execution order:** FROM → WHERE (filter rows) → GROUP BY (group) → SELECT (aggregate) → HAVING (filter groups) → ORDER BY → LIMIT.
- **WHERE vs HAVING:** WHERE filters individual rows before grouping. HAVING filters groups after grouping. HAVING can use aggregate functions; WHERE cannot.
- **WITH ROLLUP:** adds automatic subtotal/grand total rows to GROUP BY results.
- A **subquery** is a SELECT inside another SQL statement, wrapped in `()`. The inner query runs first, the outer query uses its result.
- **Subquery with `=`:** inner must return exactly ONE value. Use for MIN, MAX, single comparisons.
- **Subquery with `IN`:** inner returns a list of values. Use for "find rows matching any of these".
- **NOT IN + NULL = disaster:** if the subquery list contains any NULL, NOT IN returns 0 rows. Always add `WHERE col IS NOT NULL` to the subquery, or use NOT EXISTS instead.
- **EXISTS / NOT EXISTS:** checks if any rows match (TRUE/FALSE). Naturally handles NULLs. Usually faster than IN for large datasets.
- **Scalar subquery (in SELECT):** returns one value per outer row. Runs once per row — can be slow on large tables.
- **Derived table (in FROM):** subquery creates a temporary table. Must have an alias: `FROM (SELECT ...) AS alias`.
- **Correlated subquery:** references the outer query's current row (via table alias). Runs once PER ROW of the outer query.
- **Subquery vs JOIN:** use subquery for single-value comparisons and EXISTS checks. Use JOIN when you need columns from multiple tables in the result.
- **In PHP:** subqueries work exactly like any other SQL in PDO. Use named parameters for any user-supplied values in either the outer or inner query.