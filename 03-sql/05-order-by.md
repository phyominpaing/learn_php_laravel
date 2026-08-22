# SQL — ORDER BY, ASC & DESC — Sorting Query Results

**ORDER BY** is the SQL clause that controls the **order in which rows are returned** from a query. Without it, MySQL returns rows in whatever internal order it finds them — which is unpredictable and unreliable. Every time you build a list, a table, a leaderboard, a feed, or any sorted output, ORDER BY is what makes it work correctly.

---

## Table of Contents

1. [Why Ordering Matters](#why-ordering-matters)
2. [Basic ORDER BY Syntax](#basic-order-by-syntax)
3. [ASC — Ascending Order](#asc--ascending-order)
4. [DESC — Descending Order](#desc--descending-order)
5. [ORDER BY on Different Data Types](#order-by-on-different-data-types)
6. [ORDER BY Multiple Columns](#order-by-multiple-columns)
7. [ORDER BY with WHERE](#order-by-with-where)
8. [ORDER BY with LIMIT](#order-by-with-limit)
9. [ORDER BY Column Position (Number)](#order-by-column-position-number)
10. [ORDER BY with Expressions and Functions](#order-by-with-expressions-and-functions)
11. [ORDER BY with NULL Values](#order-by-with-null-values)
12. [ORDER BY with CASE — Custom Sort Order](#order-by-with-case--custom-sort-order)
13. [ORDER BY in Real-World Scenarios](#order-by-in-real-world-scenarios)
14. [ORDER BY and Performance](#order-by-and-performance)
15. [ORDER BY in PHP with PDO](#order-by-in-php-with-pdo)
16. [Common Mistakes](#common-mistakes)
17. [Quick Revision](#quick-revision)

---

## Why Ordering Matters

Without ORDER BY, MySQL returns rows in the order it finds them internally — determined by physical storage, index usage, and the query optimizer. This order can change silently after data modifications, server restarts, or MySQL version upgrades.

```sql
-- Without ORDER BY: unpredictable
SELECT id, name, price FROM products;
-- Today returns: Shirt, Book, Laptop
-- Tomorrow: Laptop, Shirt, Book   (same data, different order!)
-- This is a BUG waiting to happen

-- With ORDER BY: always predictable
SELECT id, name, price FROM products ORDER BY name ASC;
-- Always returns: Book, Laptop, Shirt — guaranteed
```

```
Real consequences of missing ORDER BY:

❌ "Show newest orders first" — sometimes shows old ones first
❌ "Top 10 products" with LIMIT 10 — might not be the actual top 10
❌ Pagination breaks — same row appears on page 1 and page 2
❌ Reports are inconsistent — different result each day
❌ "First registered user" is sometimes wrong

✅ ORDER BY solves all of these — deterministic, reliable ordering
```

---

## Basic ORDER BY Syntax

```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition          -- optional
ORDER BY column_name direction;  -- direction = ASC or DESC

-- direction is optional — defaults to ASC if omitted
SELECT * FROM books ORDER BY price;       -- same as ORDER BY price ASC
SELECT * FROM books ORDER BY price ASC;   -- explicit ascending
SELECT * FROM books ORDER BY price DESC;  -- explicit descending
```

### The Full SELECT Clause Order

ORDER BY must always come near the end of the query — after WHERE and GROUP BY:

```sql
SELECT   columns        -- 1. what to return
FROM     table          -- 2. where from
WHERE    condition      -- 3. filter rows (optional)
GROUP BY column         -- 4. group rows (optional)
HAVING   condition      -- 5. filter groups (optional)
ORDER BY column DIR     -- 6. sort results ← HERE
LIMIT    n              -- 7. limit rows (optional)
OFFSET   m;             -- 8. skip rows (optional)
```

---

## ASC — Ascending Order

- **ASC** = **Ascending** = smallest to largest, oldest to newest, A to Z.
- This is the **default** — if you write `ORDER BY price`, MySQL assumes `ORDER BY price ASC`.
- Think of it as counting up: 1, 2, 3... or A, B, C...

```sql
-- Setup: use the bookstore database from previous notes
USE bookstore;

-- Numbers: smallest first (cheapest → most expensive)
SELECT title, price
FROM books
ORDER BY price ASC;
```

```
Result:
┌─────────────────────────────┬────────┐
│ title                       │ price  │
├─────────────────────────────┼────────┤
│ Animal Farm                 │  8.99  │ ← cheapest first
│ Carrie                      │ 11.99  │
│ The Alchemist               │ 11.99  │
│ 1984                        │ 12.99  │
│ To Kill a Mockingbird       │ 13.99  │
│ The Shining                 │ 13.99  │
│ The Grand Design            │ 13.99  │
│ Harry Potter PS             │ 14.99  │
│ Harry Potter CoS            │ 14.99  │
│ Harry Potter PoA            │ 14.99  │
│ A Brief History of Time     │ 15.99  │
│ 21 Lessons                  │ 15.99  │
│ It                          │ 16.99  │
│ Homo Deus                   │ 16.99  │
│ Sapiens                     │ 17.99  │
│ Eloquent JavaScript         │ 24.99  │
│ You Don't Know JS           │ 29.99  │
│ Clean Code                  │ 35.99  │
│ The Pragmatic Programmer    │ 39.99  │
│ Design Patterns             │ 44.99  │ ← most expensive last
└─────────────────────────────┴────────┘
```

```sql
-- Text: A to Z (alphabetical)
SELECT title, author
FROM books
ORDER BY title ASC;
-- Returns: 1984, 21 Lessons, A Brief History..., Animal Farm, Carrie...

-- Dates: oldest to newest
SELECT id, title, created_at
FROM books
ORDER BY created_at ASC;
-- Returns oldest records first

-- All direction variations (all mean ASC):
SELECT * FROM books ORDER BY price;        -- implicit ASC
SELECT * FROM books ORDER BY price ASC;    -- explicit ASC (recommended)
```

---

## DESC — Descending Order

- **DESC** = **Descending** = largest to smallest, newest to oldest, Z to A.
- Think of it as counting down: 100, 99, 98... or Z, Y, X...

```sql
-- Numbers: largest first (most expensive → cheapest)
SELECT title, price
FROM books
ORDER BY price DESC;
```

```
Result:
┌─────────────────────────────┬────────┐
│ title                       │ price  │
├─────────────────────────────┼────────┤
│ Design Patterns             │ 44.99  │ ← most expensive first
│ The Pragmatic Programmer    │ 39.99  │
│ Clean Code                  │ 35.99  │
│ You Don't Know JS           │ 29.99  │
│ Eloquent JavaScript         │ 24.99  │
│ Sapiens                     │ 17.99  │
│ It                          │ 16.99  │
│ Homo Deus                   │ 16.99  │
│ ...                         │ ...    │
│ Animal Farm                 │  8.99  │ ← cheapest last
└─────────────────────────────┴────────┘
```

```sql
-- Text: Z to A (reverse alphabetical)
SELECT title
FROM books
ORDER BY title DESC;
-- Returns: You Don't Know JS, To Kill a Mockingbird, The Shining...

-- Dates: newest to oldest (most common use case)
SELECT id, title, created_at
FROM books
ORDER BY created_at DESC;
-- Returns the most recently added books first

-- IDs: highest to lowest (effectively newest first if using AUTO_INCREMENT)
SELECT id, title
FROM books
ORDER BY id DESC;
-- Returns last inserted rows first
```

### ASC vs DESC at a Glance

| Direction | Keyword | Numbers | Text | Dates |
|---|---|---|---|---|
| Ascending | `ASC` (default) | 1, 2, 3... | A, B, C... | Oldest → Newest |
| Descending | `DESC` | 100, 99, 98... | Z, Y, X... | Newest → Oldest |

---

## ORDER BY on Different Data Types

ORDER BY behaves differently depending on the column's data type.

### Sorting Numbers

```sql
-- Integers — sorts numerically (not as text)
SELECT title, stock
FROM books
ORDER BY stock ASC;
-- 30, 40, 45, 55, 60, 65, 70, 75, 80, 85, 90, 100, 110, 120, 150, 180, 200

-- Decimals — also sorts numerically
SELECT title, price
FROM books
ORDER BY price DESC;
-- 44.99, 39.99, 35.99, 29.99, 24.99...

-- Ratings
SELECT title, rating
FROM books
ORDER BY rating DESC;
-- 4.9, 4.8, 4.8, 4.8, 4.7, 4.7, 4.6...
```

### Sorting Text (VARCHAR / TEXT)

```sql
-- Alphabetical sort (case-insensitive with utf8mb4_unicode_ci collation)
SELECT title
FROM books
ORDER BY title ASC;

-- Important: how MySQL sorts numbers stored as text (VARCHAR)
-- If stock were VARCHAR (it's not, but hypothetically):
-- Sorts as: '100', '200', '30', '40', '55'  ← WRONG (text sort!)
-- As INT:   30, 40, 55, 100, 200            ← RIGHT (numeric sort!)
-- Always store numbers as numeric types!

-- Sort by author name
SELECT author, title
FROM books
ORDER BY author ASC, title ASC;
-- George Orwell: 1984, Animal Farm
-- GoF: Design Patterns
-- Harper Lee: To Kill a Mockingbird
-- J.K. Rowling: Harry Potter CoS, Harry Potter PoA, Harry Potter PS
-- ...
```

### Sorting Dates and Timestamps

```sql
-- Newest first (most common)
SELECT title, created_at
FROM books
ORDER BY created_at DESC;

-- Oldest first
SELECT title, created_at
FROM books
ORDER BY created_at ASC;

-- Sort by DATE column (birthdays, deadlines)
SELECT name, birth_date
FROM users
ORDER BY birth_date ASC;   -- oldest person first
ORDER BY birth_date DESC;  -- youngest person first

-- Sort by year only (using YEAR() function)
SELECT title, published_at
FROM books
ORDER BY YEAR(published_at) DESC;  -- most recent year first
```

### Sorting ENUM Columns

```sql
-- ENUM sorts by the INTERNAL NUMBER of the value (order defined in ENUM)
-- genre ENUM('fiction','non-fiction','science','history','programming','horror')
-- Internal numbers:  1=fiction, 2=non-fiction, 3=science, 4=history, 5=programming, 6=horror

SELECT title, genre
FROM books
ORDER BY genre ASC;
-- Returns: fiction books first, then non-fiction, then science, then history...
-- This is the ORDER the ENUM was defined in, NOT alphabetical

-- To sort ENUM alphabetically, cast to CHAR:
SELECT title, genre
FROM books
ORDER BY CAST(genre AS CHAR) ASC;
-- Returns: fiction, history, horror, non-fiction, programming, science
-- Now truly alphabetical
```

---

## ORDER BY Multiple Columns

You can sort by more than one column — the second column is used as a **tiebreaker** when two rows have the same value in the first column.

```sql
-- Syntax:
ORDER BY column1 ASC, column2 DESC, column3 ASC

-- Each column can have its own direction independently
```

```sql
-- Sort by genre alphabetically, then by price cheapest first within each genre
SELECT title, genre, price
FROM books
ORDER BY genre ASC, price ASC;
```

```
Result — grouped by genre, sorted by price within each genre:
┌─────────────────────────────┬─────────────┬────────┐
│ title                       │ genre       │ price  │
├─────────────────────────────┼─────────────┼────────┤
│ Animal Farm                 │ fiction     │  8.99  │ ← fiction, cheapest
│ Carrie (horror)             │ fiction     │ 11.99  │ ← wait, wrong!
│ ...                         │             │        │
└─────────────────────────────┴─────────────┴────────┘
```

```sql
-- Let's get it right with our actual genres:
SELECT title, genre, price
FROM books
ORDER BY genre ASC, price ASC;
```

```
Result:
┌─────────────────────────────┬─────────────┬────────┐
│ title                       │ genre       │ price  │
├─────────────────────────────┼─────────────┼────────┤
│ Animal Farm                 │ fiction     │  8.99  │ ← fiction group, $8.99
│ The Alchemist               │ fiction     │ 11.99  │ ←   sorted by price
│ 1984                        │ fiction     │ 12.99  │ ←   within group
│ To Kill a Mockingbird       │ fiction     │ 13.99  │
│ Harry Potter PS             │ fiction     │ 14.99  │
│ Harry Potter CoS            │ fiction     │ 14.99  │
│ Harry Potter PoA            │ fiction     │ 14.99  │
├─────────────────────────────┼─────────────┼────────┤
│ Sapiens                     │ history     │ 17.99  │ ← history group
│ Homo Deus                   │ history     │ 16.99  │
│ 21 Lessons                  │ history     │ 15.99  │
├─────────────────────────────┼─────────────┼────────┤
│ Carrie                      │ horror      │ 11.99  │ ← horror group
│ The Shining                 │ horror      │ 13.99  │
│ It                          │ horror      │ 16.99  │
├─────────────────────────────┼─────────────┼────────┤
│ Eloquent JavaScript         │ programming │ 24.99  │ ← programming group
│ You Don't Know JS           │ programming │ 29.99  │
│ Clean Code                  │ programming │ 35.99  │
│ Design Patterns             │ programming │ 44.99  │
│ The Pragmatic Programmer    │ programming │ 39.99  │
├─────────────────────────────┼─────────────┼────────┤
│ The Grand Design            │ science     │ 13.99  │ ← science group
│ A Brief History of Time     │ science     │ 15.99  │
└─────────────────────────────┴─────────────┴────────┘
```

```sql
-- Mix directions — each column independent
-- Sort by rating descending (best first), then by price ascending (cheaper first for ties)
SELECT title, rating, price
FROM books
ORDER BY rating DESC, price ASC;

-- Sort by author name A-Z, then newest books first within each author
SELECT author, title, created_at
FROM books
ORDER BY author ASC, created_at DESC;

-- Sort by genre, then rating (best first), then price (cheapest first)
SELECT title, genre, rating, price
FROM books
ORDER BY genre ASC, rating DESC, price ASC;

-- Three-column sort: status first, then priority, then created_at
SELECT id, title, status, priority, created_at
FROM tasks
ORDER BY status ASC, priority DESC, created_at ASC;
```

---

## ORDER BY with WHERE

ORDER BY always comes AFTER WHERE. The filtering happens first, then the result is sorted.

```sql
-- Find all fiction books, sorted cheapest first
SELECT title, author, price
FROM books
WHERE genre = 'fiction'
ORDER BY price ASC;

-- Find all programming books priced under $40, highest rated first
SELECT title, price, rating
FROM books
WHERE genre = 'programming' AND price < 40.00
ORDER BY rating DESC;

-- Find books with stock below 60, sort by stock (lowest first)
SELECT title, stock
FROM books
WHERE stock < 60
ORDER BY stock ASC;

-- Find books matching a search, sorted by relevance (rating)
SELECT title, author, genre, rating
FROM books
WHERE author LIKE 'Stephen%'
ORDER BY rating DESC;
```

---

## ORDER BY with LIMIT

Always pair ORDER BY with LIMIT — LIMIT without ORDER BY gives unpredictable rows.

```sql
-- Top 5 most expensive books
SELECT title, price
FROM books
ORDER BY price DESC
LIMIT 5;

-- 3 cheapest fiction books
SELECT title, price
FROM books
WHERE genre = 'fiction'
ORDER BY price ASC
LIMIT 3;

-- The single best-rated book
SELECT title, author, rating
FROM books
ORDER BY rating DESC
LIMIT 1;

-- Newest 5 books added to the store
SELECT id, title, created_at
FROM books
ORDER BY created_at DESC
LIMIT 5;

-- Page 2 of books sorted by title (5 per page)
SELECT title, price
FROM books
ORDER BY title ASC
LIMIT 5 OFFSET 5;
```

---

## ORDER BY Column Position (Number)

Instead of repeating a column name, you can reference it by its position in the SELECT list. This is shorthand but generally discouraged — it breaks if column order changes.

```sql
-- These three are identical:
SELECT title, price, rating FROM books ORDER BY price ASC;
SELECT title, price, rating FROM books ORDER BY 2 ASC;    -- 2 = price (2nd column)
SELECT title, price, rating FROM books ORDER BY 2;        -- ASC is default

-- Column position examples:
SELECT id, title, author, price, rating
FROM books
ORDER BY 4 DESC;    -- ORDER BY price DESC (4th column)

SELECT id, title, author, price, rating
FROM books
ORDER BY 5 DESC, 4 ASC;  -- ORDER BY rating DESC, price ASC

-- ⚠️ Avoid positional ORDER BY in production code:
-- If someone adds a column to the SELECT, position 4 becomes something else
-- Hard to read — "ORDER BY 4" gives no context about what's being sorted
-- Use column names instead: ORDER BY price DESC, rating ASC
```

---

## ORDER BY with Expressions and Functions

You can ORDER BY the result of a calculation or function — not just raw column values.

```sql
-- Sort by the discounted price (price * 0.8), even though that column doesn't exist
SELECT title, price, ROUND(price * 0.8, 2) AS discounted_price
FROM books
ORDER BY price * 0.8 ASC;
-- OR using the alias:
ORDER BY discounted_price ASC;

-- Sort by string length (shortest title first)
SELECT title, LENGTH(title) AS title_length
FROM books
ORDER BY LENGTH(title) ASC;

-- Sort by the last word in the author's name (surname sort)
SELECT author, title
FROM books
ORDER BY SUBSTRING_INDEX(author, ' ', -1) ASC;
-- SUBSTRING_INDEX gets text after last space = surname
-- Orwell, Rowling, King, Hawking... sorted alphabetically

-- Sort by year of publication
SELECT title, published_at
FROM books
ORDER BY YEAR(published_at) DESC;

-- Sort by day of week (1=Sunday, 7=Saturday)
SELECT title, created_at, DAYOFWEEK(created_at) AS day
FROM books
ORDER BY DAYOFWEEK(created_at) ASC;

-- Sort by price range bucket (group cheap/medium/expensive together)
SELECT title, price,
  CASE
    WHEN price < 15  THEN 'budget'
    WHEN price < 30  THEN 'mid-range'
    ELSE             'premium'
  END AS price_tier
FROM books
ORDER BY price_tier ASC, price ASC;

-- Sort by absolute difference from a target price
-- "Which books are closest to $20?"
SELECT title, price, ABS(price - 20) AS distance_from_20
FROM books
ORDER BY ABS(price - 20) ASC;
```

---

## ORDER BY with NULL Values

NULL values have special behaviour in ORDER BY — understanding this prevents surprises.

```sql
-- In MySQL: NULL is treated as LOWER than any non-NULL value
-- ASC order:  NULL values appear FIRST (before any real value)
-- DESC order: NULL values appear LAST (after any real value)

-- Example with a nullable column (published_at)
SELECT title, published_at
FROM books
ORDER BY published_at ASC;
-- Books with NULL published_at appear FIRST, then actual dates ascending

SELECT title, published_at
FROM books
ORDER BY published_at DESC;
-- Dates from newest to oldest, then NULLs appear at the VERY END

-- Force NULLs to appear LAST in ASC order:
SELECT title, published_at
FROM books
ORDER BY
  (published_at IS NULL) ASC,   -- 0=has value (sorts first), 1=NULL (sorts last)
  published_at ASC;             -- then sort the non-NULL values

-- Force NULLs to appear FIRST in DESC order:
SELECT title, published_at
FROM books
ORDER BY
  (published_at IS NULL) DESC,  -- 1=NULL sorts first
  published_at DESC;

-- Alternative using ISNULL() function:
SELECT title, published_at
FROM books
ORDER BY ISNULL(published_at) ASC, published_at ASC;
-- ISNULL() returns 0 for non-NULL, 1 for NULL
-- Sorting by 0 first means non-NULLs come first
```

```
NULL ordering summary in MySQL:

  ASC:  NULL → then real values (lowest to highest)
  DESC: real values (highest to lowest) → then NULL

  To control it:
    NULLs last in ASC:  ORDER BY ISNULL(col) ASC, col ASC
    NULLs first in DESC: ORDER BY ISNULL(col) DESC, col DESC
```

---

## ORDER BY with CASE — Custom Sort Order

Sometimes you need to sort in a custom order that's not alphabetical or numerical. CASE in ORDER BY lets you define any arbitrary order.

```sql
-- Sort by a custom status priority:
-- 'urgent' first, then 'high', then 'medium', then 'low'
-- Alphabetical would give: high, low, medium, urgent (wrong!)

SELECT id, title, status
FROM tasks
ORDER BY
  CASE status
    WHEN 'urgent' THEN 1
    WHEN 'high'   THEN 2
    WHEN 'medium' THEN 3
    WHEN 'low'    THEN 4
    ELSE               5
  END ASC;

-- Sort books with a custom genre order (not alphabetical):
-- Show fiction first, then history, then science, then programming, then horror
SELECT title, genre, price
FROM books
ORDER BY
  CASE genre
    WHEN 'fiction'     THEN 1
    WHEN 'history'     THEN 2
    WHEN 'science'     THEN 3
    WHEN 'programming' THEN 4
    WHEN 'horror'      THEN 5
    ELSE                    6
  END ASC,
  price ASC;   -- within each genre, sort by price

-- Pin a specific record to the top, sort everything else normally
-- "Always show 'featured' books first, then sort rest by rating"
SELECT id, title, is_featured, rating
FROM books
ORDER BY
  is_featured DESC,  -- 1 (featured) comes before 0 (not featured)
  rating DESC;       -- then by rating within each group

-- Put a specific ID first, then sort by title
SELECT id, title
FROM books
ORDER BY
  CASE WHEN id = 3 THEN 0 ELSE 1 END ASC,  -- id=3 gets position 0 (first)
  title ASC;                                  -- rest alphabetically
```

---

## ORDER BY in Real-World Scenarios

### E-Commerce Product Listings

```sql
-- Default sort: featured products first, then by rating
SELECT id, name, price, rating, is_featured
FROM products
WHERE status = 'active' AND deleted_at IS NULL
ORDER BY is_featured DESC, rating DESC, id ASC;

-- Sort by "Best Sellers" (most orders)
SELECT p.id, p.name, p.price, COUNT(oi.id) AS times_ordered
FROM products p
LEFT JOIN order_items oi ON p.id = oi.product_id
GROUP BY p.id, p.name, p.price
ORDER BY times_ordered DESC;

-- Sort by "Price: Low to High"
SELECT id, name, price FROM products
WHERE status = 'active'
ORDER BY price ASC, name ASC;

-- Sort by "Price: High to Low"
SELECT id, name, price FROM products
WHERE status = 'active'
ORDER BY price DESC, name ASC;

-- Sort by "Newest Arrivals"
SELECT id, name, price, created_at FROM products
WHERE status = 'active'
ORDER BY created_at DESC;
```

### User Admin Panel

```sql
-- List all users, newest registrations first
SELECT id, name, email, role, status, created_at
FROM users
WHERE deleted_at IS NULL
ORDER BY created_at DESC;

-- List users alphabetically by name, active first
SELECT id, name, email, status
FROM users
ORDER BY
  CASE status WHEN 'active' THEN 0 ELSE 1 END ASC,
  name ASC;

-- Users who haven't logged in recently (oldest last_login first)
SELECT id, name, email, last_login_at
FROM users
WHERE status = 'active'
ORDER BY last_login_at ASC
LIMIT 20;
```

### News / Blog Feed

```sql
-- Latest posts (newest first)
SELECT id, title, author_id, published_at
FROM posts
WHERE status = 'published' AND published_at <= NOW()
ORDER BY published_at DESC;

-- Trending posts (most views in last 7 days)
SELECT id, title, views_count
FROM posts
WHERE published_at >= (NOW() - INTERVAL 7 DAY)
ORDER BY views_count DESC
LIMIT 10;

-- Posts sorted by popularity score
SELECT id, title,
  (views_count * 1 + comments_count * 3 + likes_count * 2) AS score
FROM posts
WHERE status = 'published'
ORDER BY score DESC
LIMIT 20;
```

### Leaderboard / Rankings

```sql
-- Top 10 highest scoring users
SELECT
  RANK() OVER (ORDER BY score DESC) AS rank_position,
  name, score, level
FROM users
ORDER BY score DESC
LIMIT 10;

-- Leaderboard with ties handled: same score = same rank
SELECT name, score,
  DENSE_RANK() OVER (ORDER BY score DESC) AS rank_position
FROM game_scores
ORDER BY rank_position ASC, name ASC;

-- Order history: user's orders newest first
SELECT id, total, status, created_at
FROM orders
WHERE user_id = 42 AND deleted_at IS NULL
ORDER BY created_at DESC;
```

---

## ORDER BY and Performance

### When Sorting is Fast

```sql
-- Fast: sorting by an indexed column
SELECT id, name, email FROM users ORDER BY id ASC;        -- PRIMARY KEY index
SELECT id, email FROM users ORDER BY email ASC;           -- has UNIQUE index
SELECT id, created_at FROM orders ORDER BY created_at DESC; -- has index

-- MySQL uses the index to return rows already sorted — NO extra sort step needed
-- EXPLAIN shows: "Using index" — fastest possible
```

### When Sorting is Slow (Filesort)

```sql
-- Slow: sorting by a non-indexed column
SELECT id, name, phone FROM users ORDER BY phone ASC;
-- No index on phone → MySQL must:
-- 1. Read ALL rows
-- 2. Load them into memory (or a temp file if too many)
-- 3. Sort them
-- 4. Return them
-- This is called a "filesort" — shows in EXPLAIN as "Using filesort"

-- Check if your ORDER BY is using an index:
EXPLAIN SELECT id, name FROM users ORDER BY name ASC;
-- If you see "Using filesort" → add an index on that column
CREATE INDEX users_name_index ON users (name);
```

### Composite Index for ORDER BY

```sql
-- If you always filter by status AND sort by created_at:
SELECT * FROM orders WHERE status = 'shipped' ORDER BY created_at DESC;

-- A composite index covers both:
CREATE INDEX orders_status_created ON orders (status, created_at);
-- MySQL can use this index for the WHERE and the ORDER BY together
-- Extremely fast — no filesort needed
```

> 💡 **Performance Rule:** If you frequently sort by a column (especially with LIMIT), add an index on it. For common filter+sort combinations, use a composite index covering both columns.

---

## ORDER BY in PHP with PDO

### Safe Dynamic Sorting

```php
<?php
// ⚠️ NEVER put ORDER BY column/direction directly from user input!
// SQL Injection risk:
$sort   = $_GET['sort'];                            // attacker sends: "id; DROP TABLE users"
$query  = "SELECT * FROM books ORDER BY $sort";     // ❌ DANGEROUS!

// ✅ SAFE: Whitelist allowed sort options
function getBooks(PDO $pdo, string $sortBy = 'created_at', string $direction = 'DESC'): array {

    // 1. Whitelist allowed column names
    $allowedColumns = ['title', 'price', 'rating', 'created_at', 'author', 'stock'];
    if (!in_array($sortBy, $allowedColumns, true)) {
        $sortBy = 'created_at';  // default if invalid
    }

    // 2. Whitelist allowed directions
    $direction = strtoupper($direction);
    if (!in_array($direction, ['ASC', 'DESC'], true)) {
        $direction = 'DESC';  // default if invalid
    }

    // 3. Safe to interpolate — both values are whitelisted
    $sql  = "SELECT id, title, author, price, rating
             FROM books
             WHERE deleted_at IS NULL
             ORDER BY $sortBy $direction";

    $stmt = $pdo->prepare($sql);
    $stmt->execute();
    return $stmt->fetchAll();
}

// Usage from URL: /books?sort=price&dir=asc
$books = getBooks(
    $pdo,
    sortBy:    $_GET['sort'] ?? 'created_at',
    direction: $_GET['dir']  ?? 'DESC'
);

foreach ($books as $book) {
    echo "{$book['title']} — \${$book['price']}\n";
}
?>
```

### Full Sortable Table with Pagination

```php
<?php
function getPagedBooks(
    PDO   $pdo,
    int   $page      = 1,
    int   $perPage   = 10,
    string $sortBy   = 'created_at',
    string $direction = 'DESC',
    string $genre    = ''
): array {

    // Whitelist sort columns
    $allowed = ['id', 'title', 'author', 'price', 'rating', 'stock', 'created_at'];
    $sortBy   = in_array($sortBy, $allowed, true) ? $sortBy : 'created_at';
    $direction = strtoupper($direction) === 'ASC' ? 'ASC' : 'DESC';

    // Pagination
    $page   = max(1, $page);
    $offset = ($page - 1) * $perPage;

    // WHERE clause
    $where  = "deleted_at IS NULL";
    $params = [];

    if (!empty($genre)) {
        $where .= " AND genre = :genre";
        $params[':genre'] = $genre;
    }

    // Count total
    $countStmt = $pdo->prepare("SELECT COUNT(*) FROM books WHERE $where");
    $countStmt->execute($params);
    $total = (int) $countStmt->fetchColumn();

    // Get data
    $stmt = $pdo->prepare(
        "SELECT id, title, author, genre, price, rating, stock
         FROM books
         WHERE $where
         ORDER BY $sortBy $direction
         LIMIT :limit OFFSET :offset"
    );

    foreach ($params as $key => $val) {
        $stmt->bindValue($key, $val);
    }
    $stmt->bindValue(':limit',  $perPage, PDO::PARAM_INT);
    $stmt->bindValue(':offset', $offset,  PDO::PARAM_INT);
    $stmt->execute();

    return [
        'data'         => $stmt->fetchAll(),
        'total'        => $total,
        'total_pages'  => (int) ceil($total / $perPage),
        'current_page' => $page,
        'sort_by'      => $sortBy,
        'direction'    => $direction,
    ];
}

// Usage
$result = getPagedBooks(
    pdo:       $pdo,
    page:      (int) ($_GET['page']  ?? 1),
    perPage:   10,
    sortBy:    $_GET['sort'] ?? 'created_at',
    direction: $_GET['dir']  ?? 'DESC',
    genre:     $_GET['genre'] ?? ''
);

echo "Page {$result['current_page']} of {$result['total_pages']}\n";
echo "Sorted by: {$result['sort_by']} {$result['direction']}\n\n";

foreach ($result['data'] as $book) {
    echo "[{$book['genre']}] {$book['title']} — \${$book['price']} (⭐{$book['rating']})\n";
}

// Sortable column links for HTML:
// /books?sort=price&dir=ASC   → sort by price ascending
// /books?sort=price&dir=DESC  → sort by price descending
// /books?sort=title&dir=ASC   → sort by title A-Z
?>
```

---

## Common Mistakes

```sql
-- ❌ MISTAKE 1: Forgetting ORDER BY with LIMIT
SELECT * FROM orders LIMIT 10;
-- Which 10 orders? No one knows — MySQL decides
-- ✅ Fix:
SELECT * FROM orders ORDER BY created_at DESC LIMIT 10;

────────────────────────────────────────────────────────────

-- ❌ MISTAKE 2: Using ORDER BY without LIMIT on a huge table
SELECT * FROM users ORDER BY name ASC;
-- If you have 1,000,000 users, MySQL must sort ALL of them
-- before returning — massive memory use and slow
-- ✅ Fix: Always LIMIT if you don't need all rows
SELECT * FROM users ORDER BY name ASC LIMIT 50;

────────────────────────────────────────────────────────────

-- ❌ MISTAKE 3: Expecting alphabetical sort for ENUM
SELECT * FROM products ORDER BY status ASC;
-- ENUM sorts by internal number, NOT alphabetically!
-- ✅ Fix: if you want alphabetical:
SELECT * FROM products ORDER BY CAST(status AS CHAR) ASC;

────────────────────────────────────────────────────────────

-- ❌ MISTAKE 4: Directly using user input in ORDER BY
$col = $_GET['sort'];
$sql = "SELECT * FROM users ORDER BY $col";  -- SQL INJECTION!
-- ✅ Fix: whitelist allowed columns:
$allowed = ['name', 'email', 'created_at'];
$col = in_array($_GET['sort'], $allowed, true) ? $_GET['sort'] : 'created_at';

────────────────────────────────────────────────────────────

-- ❌ MISTAKE 5: ORDER BY column not in SELECT (in DISTINCT queries)
SELECT DISTINCT genre FROM books ORDER BY price ASC;
-- ERROR: ORDER BY 'price' is not in SELECT list (with DISTINCT)
-- ✅ Fix: include price in SELECT or sort by selected column
SELECT DISTINCT genre FROM books ORDER BY genre ASC;

────────────────────────────────────────────────────────────

-- ❌ MISTAKE 6: Assuming NULL sorts the way you want
SELECT title, published_at FROM books ORDER BY published_at ASC;
-- NULLs appear FIRST — is that what you want?
-- ✅ Fix: explicitly push NULLs to the end
SELECT title, published_at FROM books
ORDER BY ISNULL(published_at) ASC, published_at ASC;
```

---

## Quick Revision

- **`ORDER BY column_name`** sorts query results. Without it, MySQL's row order is unpredictable.
- **`ASC`** (Ascending) = smallest to largest, A to Z, oldest to newest. This is the **default** — writing `ORDER BY price` is the same as `ORDER BY price ASC`.
- **`DESC`** (Descending) = largest to smallest, Z to A, newest to oldest. Must be written explicitly.
- **Clause order:** `SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT → OFFSET`. ORDER BY always comes after WHERE.
- **Multiple columns:** `ORDER BY genre ASC, price ASC` — genre is the primary sort, price is the tiebreaker within the same genre. Each column has its own independent direction.
- **With WHERE:** filtering happens first, then the filtered result is sorted.
- **With LIMIT:** always pair ORDER BY with LIMIT — `LIMIT 10` without `ORDER BY` returns 10 arbitrary rows.
- **Data types:** numbers sort numerically, text sorts alphabetically (case-insensitive with utf8mb4_unicode_ci), dates sort chronologically, ENUM sorts by internal index number (not alphabetically).
- **NULL values:** in ASC, NULLs appear first. In DESC, NULLs appear last. Control with `ORDER BY ISNULL(col) ASC, col ASC` to push NULLs to the end.
- **Custom order with CASE:** `ORDER BY CASE status WHEN 'urgent' THEN 1 WHEN 'high' THEN 2 ELSE 3 END ASC` — define any arbitrary sort sequence.
- **Performance:** sorting by indexed columns is fast (MySQL skips the sort step). Sorting by non-indexed columns causes a "filesort" — add an index to fix it.
- **In PHP:** NEVER put ORDER BY column/direction from user input directly into SQL — SQL injection risk. Always use a whitelist: `in_array($sort, $allowed, true)`.
- **Real-world uses:** newest first (`ORDER BY created_at DESC`), cheapest first (`ORDER BY price ASC`), top rated (`ORDER BY rating DESC`), alphabetical (`ORDER BY name ASC`), leaderboard (`ORDER BY score DESC`).