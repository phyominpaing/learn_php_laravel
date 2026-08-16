# SQL — LIMIT & OFFSET — Pagination & Controlled Data Fetching

**LIMIT and OFFSET** are two of the most practical SQL clauses you'll use every single day as a backend developer. They control *how many rows* MySQL returns and *where to start returning from* — making them the foundation of pagination, "top N" queries, and performance optimization.

---

## Table of Contents

1. [What is LIMIT?](#what-is-limit)
2. [What is OFFSET?](#what-is-offset)
3. [LIMIT and OFFSET Together — Pagination](#limit-and-offset-together--pagination)
4. [The Shorthand Syntax — LIMIT x, y](#the-shorthand-syntax--limit-x-y)
5. [Pagination — The Real-World Formula](#pagination--the-real-world-formula)
6. [LIMIT with ORDER BY — Always Pair Them](#limit-with-order-by--always-pair-them)
7. [LIMIT in UPDATE and DELETE](#limit-in-update-and-delete)
8. [Performance and Best Practices](#performance-and-best-practices)
9. [LIMIT and OFFSET in PHP with PDO](#limit-and-offset-in-php-with-pdo)
10. [Building a Complete Pagination System in PHP](#building-a-complete-pagination-system-in-php)
11. [Common Mistakes](#common-mistakes)
12. [Quick Revision](#quick-revision)

---

## What is LIMIT?

- `LIMIT` tells MySQL to return **at most N rows** from the result — no matter how many rows match your query.
- Without LIMIT, MySQL returns **every single matching row** — which could be thousands or millions of rows.
- Syntax: `LIMIT number`

```sql
-- Practice setup — run this first
CREATE DATABASE IF NOT EXISTS bookstore
  CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE bookstore;

CREATE TABLE IF NOT EXISTS books (
  id         INT UNSIGNED  NOT NULL AUTO_INCREMENT,
  title      VARCHAR(255)  NOT NULL,
  author     VARCHAR(100)  NOT NULL,
  genre      VARCHAR(50)   NOT NULL,
  price      DECIMAL(10,2) NOT NULL,
  stock      INT UNSIGNED  NOT NULL DEFAULT 0,
  rating     DECIMAL(3,1)  NOT NULL DEFAULT 0.0,
  created_at TIMESTAMP     NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

INSERT INTO books (title, author, genre, price, stock, rating) VALUES
  ('1984',                    'George Orwell',    'fiction',     12.99, 100, 4.8),
  ('Animal Farm',             'George Orwell',    'fiction',      8.99,  75, 4.5),
  ('Harry Potter PS',         'J.K. Rowling',     'fiction',     14.99, 200, 4.9),
  ('Harry Potter CoS',        'J.K. Rowling',     'fiction',     14.99, 180, 4.7),
  ('Harry Potter PoA',        'J.K. Rowling',     'fiction',     14.99, 150, 4.8),
  ('The Shining',             'Stephen King',     'horror',      13.99,  55, 4.6),
  ('It',                      'Stephen King',     'horror',      16.99,  40, 4.5),
  ('Carrie',                  'Stephen King',     'horror',      11.99,  60, 4.3),
  ('A Brief History of Time', 'Stephen Hawking',  'science',     15.99,  90, 4.7),
  ('The Grand Design',        'Stephen Hawking',  'science',     13.99,  70, 4.4),
  ('Sapiens',                 'Yuval Noah Harari','history',     17.99,  80, 4.8),
  ('Homo Deus',               'Yuval Noah Harari','history',     16.99,  65, 4.5),
  ('21 Lessons',              'Yuval Noah Harari','history',     15.99,  55, 4.4),
  ('Clean Code',              'Robert Martin',    'programming', 35.99,  45, 4.7),
  ('The Pragmatic Programmer','David Thomas',     'programming', 39.99,  40, 4.8),
  ('Design Patterns',         'GoF',              'programming', 44.99,  30, 4.6),
  ('You Don\'t Know JS',      'Kyle Simpson',     'programming', 29.99,  70, 4.5),
  ('Eloquent JavaScript',     'Marijn Haverbeke', 'programming', 24.99,  85, 4.4),
  ('The Alchemist',           'Paulo Coelho',     'fiction',     11.99, 120, 4.6),
  ('To Kill a Mockingbird',   'Harper Lee',       'fiction',     13.99, 110, 4.8);
-- 20 books total
```

### Basic LIMIT Examples

```sql
-- Without LIMIT: returns ALL 20 books
SELECT id, title, price FROM books;
-- Returns: 20 rows

-- With LIMIT 5: returns only the FIRST 5 rows
SELECT id, title, price FROM books LIMIT 5;
-- Returns: 5 rows (rows 1-5)

-- With LIMIT 1: returns only the VERY FIRST matching row
SELECT id, title, price FROM books LIMIT 1;
-- Returns: 1 row

-- With LIMIT 10: returns first 10 rows
SELECT id, title, price FROM books LIMIT 10;
-- Returns: 10 rows (rows 1-10)

-- If LIMIT is larger than available rows, returns all available rows (no error)
SELECT id, title, price FROM books LIMIT 1000;
-- Returns: 20 rows (only 20 books exist — LIMIT 1000 just means "up to 1000")
```

```
Without LIMIT:             With LIMIT 5:
┌────┬───────────────────┐  ┌────┬───────────────────┐
│ id │ title             │  │ id │ title             │
├────┼───────────────────┤  ├────┼───────────────────┤
│  1 │ 1984              │  │  1 │ 1984              │ ← returned
│  2 │ Animal Farm       │  │  2 │ Animal Farm       │ ← returned
│  3 │ Harry Potter PS   │  │  3 │ Harry Potter PS   │ ← returned
│  4 │ Harry Potter CoS  │  │  4 │ Harry Potter CoS  │ ← returned
│  5 │ Harry Potter PoA  │  │  5 │ Harry Potter PoA  │ ← returned
│  6 │ The Shining       │  │  6 │ The Shining       │ ← STOPPED HERE
│  7 │ It                │  └────┴───────────────────┘
│  8 │ Carrie            │
│ ...│ ...               │
│ 20 │ Mockingbird       │
└────┴───────────────────┘
20 rows                      5 rows
```

> 💡 **Why LIMIT matters for performance:** If a table has 5 million rows and you only need the first 10, using `LIMIT 10` tells MySQL to STOP after finding 10 matching rows — it doesn't read the other 4,999,990 rows. Massively faster and uses far less memory.

---

## What is OFFSET?

- `OFFSET` tells MySQL **how many rows to skip** before starting to return results.
- Used together with LIMIT to implement pagination.
- Syntax: `LIMIT number OFFSET skip_count`
- Think of it as: "Skip the first N rows, then give me the next M rows."

```sql
-- OFFSET 0: skip 0 rows → start from the very beginning (same as no OFFSET)
SELECT id, title FROM books LIMIT 5 OFFSET 0;
-- Returns: rows 1, 2, 3, 4, 5

-- OFFSET 5: skip the first 5 rows → start from row 6
SELECT id, title FROM books LIMIT 5 OFFSET 5;
-- Returns: rows 6, 7, 8, 9, 10

-- OFFSET 10: skip the first 10 rows → start from row 11
SELECT id, title FROM books LIMIT 5 OFFSET 10;
-- Returns: rows 11, 12, 13, 14, 15

-- OFFSET 15: skip the first 15 rows → start from row 16
SELECT id, title FROM books LIMIT 5 OFFSET 15;
-- Returns: rows 16, 17, 18, 19, 20

-- OFFSET 20: skip the first 20 rows → no rows left (returns empty result)
SELECT id, title FROM books LIMIT 5 OFFSET 20;
-- Returns: 0 rows (empty)
```

```
All 20 books:
┌────┬───────────────────┐
│ id │ title             │
├────┼───────────────────┤
│  1 │ 1984              │ ─┐ OFFSET 0 → SKIP these 0 rows
│  2 │ Animal Farm       │  │ LIMIT 5  → RETURN these 5
│  3 │ Harry Potter PS   │  │
│  4 │ Harry Potter CoS  │  │
│  5 │ Harry Potter PoA  │ ─┘
├────┼───────────────────┤
│  6 │ The Shining       │ ─┐ OFFSET 5 → SKIP these 5 rows (rows 1-5 above)
│  7 │ It                │  │ LIMIT 5  → RETURN these 5
│  8 │ Carrie            │  │
│  9 │ Brief History     │  │
│ 10 │ The Grand Design  │ ─┘
├────┼───────────────────┤
│ 11 │ Sapiens           │ ─┐ OFFSET 10 → SKIP first 10 rows
│ 12 │ Homo Deus         │  │ LIMIT 5   → RETURN these 5
│ 13 │ 21 Lessons        │  │
│ 14 │ Clean Code        │  │
│ 15 │ Pragmatic Prog.   │ ─┘
├────┼───────────────────┤
│ 16 │ Design Patterns   │ ─┐ OFFSET 15 → SKIP first 15 rows
│ 17 │ You Don't Know JS │  │ LIMIT 5   → RETURN these 5
│ 18 │ Eloquent JS       │  │
│ 19 │ The Alchemist     │  │
│ 20 │ Mockingbird       │ ─┘
└────┴───────────────────┘
```

---

## LIMIT and OFFSET Together — Pagination

This is the most common use — showing data in pages (like Google's search results, a product listing page, a news feed).

```sql
-- Page 1: first 5 books (skip 0)
SELECT id, title, author, price
FROM books
LIMIT 5 OFFSET 0;
-- Results: books 1-5

-- Page 2: next 5 books (skip 5)
SELECT id, title, author, price
FROM books
LIMIT 5 OFFSET 5;
-- Results: books 6-10

-- Page 3: next 5 books (skip 10)
SELECT id, title, author, price
FROM books
LIMIT 5 OFFSET 10;
-- Results: books 11-15

-- Page 4: next 5 books (skip 15)
SELECT id, title, author, price
FROM books
LIMIT 5 OFFSET 15;
-- Results: books 16-20

-- Page 5: next 5 books (skip 20) — no books left
SELECT id, title, author, price
FROM books
LIMIT 5 OFFSET 20;
-- Results: empty (0 rows)
```

```
The Pagination Pattern:

Page 1:  LIMIT 5 OFFSET 0    → shows items 1–5
Page 2:  LIMIT 5 OFFSET 5    → shows items 6–10
Page 3:  LIMIT 5 OFFSET 10   → shows items 11–15
Page 4:  LIMIT 5 OFFSET 15   → shows items 16–20

Formula:
  OFFSET = (page_number - 1) × items_per_page

  Page 1:  (1 - 1) × 5 =  0  → OFFSET 0
  Page 2:  (2 - 1) × 5 =  5  → OFFSET 5
  Page 3:  (3 - 1) × 5 = 10  → OFFSET 10
  Page 4:  (4 - 1) × 5 = 15  → OFFSET 15
```

---

## The Shorthand Syntax — LIMIT x, y

MySQL supports a shorthand syntax where you can write both values in LIMIT directly:

```sql
-- Standard syntax (recommended — clearest to read):
SELECT * FROM books LIMIT 5 OFFSET 10;

-- Shorthand syntax (LIMIT offset, count):
SELECT * FROM books LIMIT 10, 5;
--                         ↑   ↑
--                     offset  count (number of rows)

-- These two are IDENTICAL:
SELECT * FROM books LIMIT 5 OFFSET 10;  -- "return 5 rows, skip first 10"
SELECT * FROM books LIMIT 10, 5;        -- "skip 10, return 5"

-- More shorthand examples:
SELECT * FROM books LIMIT 0, 5;   -- same as LIMIT 5 OFFSET 0  → rows 1-5
SELECT * FROM books LIMIT 5, 5;   -- same as LIMIT 5 OFFSET 5  → rows 6-10
SELECT * FROM books LIMIT 10, 5;  -- same as LIMIT 5 OFFSET 10 → rows 11-15
SELECT * FROM books LIMIT 15, 5;  -- same as LIMIT 5 OFFSET 15 → rows 16-20
```

> ⚠️ **Which syntax to use?** Use the `LIMIT n OFFSET m` form — it reads naturally in English ("give me 5 rows, offset by 10"). The shorthand `LIMIT 10, 5` is confusing because the numbers are reversed (offset first, count second) — easy to mix up. The standard `LIMIT n OFFSET m` is also compatible with other databases (PostgreSQL, SQLite).

---

## Pagination — The Real-World Formula

This is the exact formula you'll use every time you build a paginated list in PHP.

```sql
-- Variables (these come from your PHP code or URL params):
-- $page     = current page number (1, 2, 3, ...)
-- $per_page = number of items per page (e.g. 10, 20, 25, 50)

-- Formula:
-- OFFSET = ($page - 1) * $per_page

-- Example: 10 items per page, user is on page 3:
-- OFFSET = (3 - 1) * 10 = 20
SELECT id, title, author, price, genre
FROM books
ORDER BY id ASC
LIMIT 10 OFFSET 20;
-- Returns books 21-30 (page 3 of a 10-per-page list)
```

### Pagination with Real Filters

In practice, pagination is combined with filtering and sorting:

```sql
-- Show page 2 of fiction books, sorted by price (cheapest first)
-- 5 books per page
-- Page 2: OFFSET = (2-1) * 5 = 5
SELECT id, title, author, price
FROM books
WHERE genre = 'fiction'
ORDER BY price ASC
LIMIT 5 OFFSET 5;

-- Show page 1 of programming books, sorted by rating (highest first)
SELECT id, title, author, price, rating
FROM books
WHERE genre = 'programming'
ORDER BY rating DESC
LIMIT 5 OFFSET 0;

-- Show page 3 of ALL books ordered by newest first
-- 10 per page → OFFSET = (3-1) * 10 = 20
SELECT id, title, author, price, created_at
FROM books
ORDER BY created_at DESC
LIMIT 10 OFFSET 20;
```

### Getting the Total Count (for "Page X of Y")

To show "Page 2 of 4", you also need the total number of matching rows:

```sql
-- Step 1: Get total count (same WHERE conditions, no LIMIT)
SELECT COUNT(*) AS total FROM books WHERE genre = 'fiction';
-- Result: total = 8 (8 fiction books)

-- Step 2: Get the page data
SELECT id, title, price
FROM books
WHERE genre = 'fiction'
ORDER BY price ASC
LIMIT 5 OFFSET 0;

-- In PHP, calculate total pages:
-- total_pages = ceil(total / per_page) = ceil(8 / 5) = 2 pages
```

```
With 8 fiction books, 5 per page:

Page 1: LIMIT 5 OFFSET 0 → books 1-5 of fiction  ✅
Page 2: LIMIT 5 OFFSET 5 → books 6-8 of fiction  ✅ (only 3 returned — that's fine)
Page 3: LIMIT 5 OFFSET 10 → 0 rows returned       ← no more pages

Total pages = CEIL(8 / 5) = CEIL(1.6) = 2 pages
```

---

## LIMIT with ORDER BY — Always Pair Them

This is a **critical rule** that beginners often miss:

> **Without ORDER BY, LIMIT returns rows in an unpredictable order.**

MySQL can return rows in any order it wants when you don't specify ORDER BY — typically the physical storage order, but this can change after updates, deletions, and optimizer changes. If you use LIMIT without ORDER BY, you might get different rows each time.

```sql
-- ❌ UNPREDICTABLE — MySQL can return any 5 rows
SELECT id, title, price FROM books LIMIT 5;
-- The 5 rows you get might change between queries — not reliable!

-- ✅ PREDICTABLE — always returns the 5 cheapest books
SELECT id, title, price
FROM books
ORDER BY price ASC
LIMIT 5;

-- ✅ Most recent 5 books (newest first)
SELECT id, title, price, created_at
FROM books
ORDER BY created_at DESC
LIMIT 5;

-- ✅ Top 5 highest-rated books
SELECT id, title, author, rating
FROM books
ORDER BY rating DESC
LIMIT 5;

-- ✅ 5 most-stocked books
SELECT id, title, stock
FROM books
ORDER BY stock DESC
LIMIT 5;

-- ✅ The single cheapest book in the store
SELECT id, title, price
FROM books
ORDER BY price ASC
LIMIT 1;

-- ✅ The single most expensive book
SELECT id, title, price
FROM books
ORDER BY price DESC
LIMIT 1;

-- ✅ The 3 newest programming books added
SELECT id, title, price, created_at
FROM books
WHERE genre = 'programming'
ORDER BY created_at DESC
LIMIT 3;
```

### Practical "Top N" Queries

```sql
-- Top 3 most expensive books
SELECT title, author, price
FROM books
ORDER BY price DESC
LIMIT 3;
-- Results: Design Patterns (44.99), Pragmatic Programmer (39.99), Clean Code (35.99)

-- Top 5 highest-rated books
SELECT title, author, rating
FROM books
ORDER BY rating DESC, title ASC   -- ties broken alphabetically
LIMIT 5;

-- Cheapest book per genre (one from each genre)
-- Note: This needs a different approach — MIN() function
SELECT genre, MIN(price) AS lowest_price
FROM books
GROUP BY genre
ORDER BY lowest_price ASC;

-- Most recent entry in the database (the last book added)
SELECT id, title, author, created_at
FROM books
ORDER BY created_at DESC
LIMIT 1;

-- Random book (useful for "book of the day" feature)
SELECT id, title, author
FROM books
ORDER BY RAND()
LIMIT 1;
-- ⚠️ Note: ORDER BY RAND() is very slow on large tables — avoid in production
--           with millions of rows. Use a different approach (see performance section).
```

---

## LIMIT in UPDATE and DELETE

LIMIT works with UPDATE and DELETE too — not just SELECT. This is very useful for safe batch operations.

```sql
-- ─── LIMIT with UPDATE ─────────────────────────────────────────────────────

-- Apply a discount to only the FIRST 5 most expensive books (test/gradual rollout)
UPDATE books
SET price = ROUND(price * 0.9, 2)   -- 10% discount
ORDER BY price DESC
LIMIT 5;
-- Only the 5 most expensive books get discounted, not all books

-- Restock the 3 books with lowest stock
UPDATE books
SET stock = stock + 100
ORDER BY stock ASC
LIMIT 3;

-- ─── LIMIT with DELETE ─────────────────────────────────────────────────────

-- Delete only 100 old records at a time (batch delete — avoids locking the table)
DELETE FROM order_logs
WHERE created_at < '2020-01-01'
ORDER BY created_at ASC
LIMIT 100;
-- After running this, run it again to delete the next 100, and so on
-- This is called "batch deletion" — much safer than deleting millions at once

-- Real PHP pattern for batch deletion:
-- do {
--   $deleted = $pdo->exec("DELETE FROM logs WHERE old = 1 ORDER BY id ASC LIMIT 1000");
-- } while ($deleted > 0);
```

> 💡 **Why batch delete with LIMIT?** Deleting millions of rows in one query:
> - Holds a write lock on the table for a long time (blocks all other queries)
> - Creates a massive transaction log entry
> - Can cause MySQL to run out of memory
>
> Deleting in batches of 500-1000 rows at a time keeps locks short and the server healthy.

---

## Performance and Best Practices

### OFFSET Gets Slower as It Gets Larger

This is the biggest performance problem with OFFSET that every backend developer must understand.

```sql
-- Fast: OFFSET is small — MySQL skips a few rows quickly
SELECT * FROM books ORDER BY id ASC LIMIT 10 OFFSET 0;    -- very fast
SELECT * FROM books ORDER BY id ASC LIMIT 10 OFFSET 100;  -- fast
SELECT * FROM books ORDER BY id ASC LIMIT 10 OFFSET 1000; -- ok

-- SLOW: Large OFFSET — MySQL reads and discards thousands of rows
SELECT * FROM books ORDER BY id ASC LIMIT 10 OFFSET 100000;  -- slow!
SELECT * FROM books ORDER BY id ASC LIMIT 10 OFFSET 1000000; -- very slow!
```

```
Why large OFFSET is slow:

  LIMIT 10 OFFSET 1000000

  MySQL CANNOT just "jump to row 1,000,001"
  MySQL must:
    1. Read rows 1 through 1,000,000  (all million rows)
    2. Discard them all
    3. Return rows 1,000,001 to 1,000,010

  The work is proportional to the OFFSET value.
  OFFSET 1,000,000 = reading 1 million rows just to throw them away!
```

### The Cursor/Keyset Pagination Solution

For large datasets, use **keyset pagination** (also called cursor-based pagination) instead of OFFSET:

```sql
-- Traditional OFFSET pagination (slow for large pages):
SELECT id, title, price FROM books ORDER BY id ASC LIMIT 10 OFFSET 50000;

-- Keyset Pagination (fast — uses index):
-- Instead of OFFSET, remember the last ID you saw and use WHERE
SELECT id, title, price
FROM books
WHERE id > 50010   -- last_id from previous page
ORDER BY id ASC
LIMIT 10;

-- Why is this fast?
-- WHERE id > 50010 uses the PRIMARY KEY index → jumps directly to id=50011
-- No rows are read and discarded!
-- Works at the same speed whether you're on page 1 or page 1,000,000
```

```
Traditional Pagination:                Keyset Pagination:
Page 1: LIMIT 10 OFFSET 0             Page 1: WHERE id > 0 LIMIT 10
Page 2: LIMIT 10 OFFSET 10            Page 2: WHERE id > 10 LIMIT 10
Page 3: LIMIT 10 OFFSET 20            Page 3: WHERE id > 20 LIMIT 10
Page 100: LIMIT 10 OFFSET 990         Page 100: WHERE id > 990 LIMIT 10
Page 100000: LIMIT 10 OFFSET 999990   Page 100000: WHERE id > 999990 LIMIT 10
  ← slow (reads 999,990 rows)           ← always fast (uses index)

Drawback of keyset: Can't jump to arbitrary page number
("go to page 47"). You must navigate sequentially.
Best for: infinite scroll, "load more" buttons, API cursors.
OFFSET is still fine for typical pagination with <1000 pages.
```

### When Standard OFFSET is Fine

```
OFFSET pagination is perfectly acceptable when:
  ✅ Total rows in the table are under ~100,000
  ✅ Users navigate through a small number of pages (< 100 pages)
  ✅ It's an admin interface not user-facing
  ✅ The dataset is filtered heavily (WHERE clause reduces rows significantly)

Use keyset/cursor pagination when:
  → Table has millions of rows
  → Users can scroll to very deep pages
  → API pagination where clients navigate many pages
  → Performance at scale matters
```

---

## LIMIT and OFFSET in PHP with PDO

Here is how to use LIMIT and OFFSET safely in PHP:

```php
<?php
// ─── IMPORTANT: LIMIT and OFFSET values MUST be bound as integers ──────────

// WRONG — string binding for LIMIT causes issues with some PDO drivers
$stmt = $pdo->prepare("SELECT * FROM books LIMIT :limit OFFSET :offset");
$stmt->bindValue(':limit',  $limit,  PDO::PARAM_STR);  // ❌ Don't use STR
$stmt->bindValue(':offset', $offset, PDO::PARAM_STR);  // ❌ Don't use STR

// CORRECT — bind as integer
$stmt = $pdo->prepare("SELECT * FROM books LIMIT :limit OFFSET :offset");
$stmt->bindValue(':limit',  (int) $limit,  PDO::PARAM_INT);  // ✅
$stmt->bindValue(':offset', (int) $offset, PDO::PARAM_INT);  // ✅
$stmt->execute();
$books = $stmt->fetchAll();

// ─── Simple pagination function ─────────────────────────────────────────────
function getBooks(PDO $pdo, int $page = 1, int $perPage = 10): array {
    $page    = max(1, $page);       // page can never be less than 1
    $perPage = min(100, max(1, $perPage)); // per page: between 1 and 100

    $offset = ($page - 1) * $perPage;

    $stmt = $pdo->prepare(
        "SELECT id, title, author, price, genre
         FROM books
         WHERE deleted_at IS NULL
         ORDER BY created_at DESC
         LIMIT :limit OFFSET :offset"
    );
    $stmt->bindValue(':limit',  $perPage, PDO::PARAM_INT);
    $stmt->bindValue(':offset', $offset,  PDO::PARAM_INT);
    $stmt->execute();

    return $stmt->fetchAll();
}

// Usage:
$books = getBooks($pdo, page: 2, perPage: 5);

// ─── Count total rows (for "Page X of Y") ───────────────────────────────────
function getBooksCount(PDO $pdo): int {
    $stmt = $pdo->query("SELECT COUNT(*) FROM books WHERE deleted_at IS NULL");
    return (int) $stmt->fetchColumn();
}

// Usage:
$total      = getBooksCount($pdo);  // 20 books
$perPage    = 5;
$totalPages = (int) ceil($total / $perPage);  // ceil(20/5) = 4 pages
$page       = (int) ($_GET['page'] ?? 1);
$page       = max(1, min($page, $totalPages)); // clamp between 1 and totalPages

$books = getBooks($pdo, $page, $perPage);

echo "Showing page $page of $totalPages ($total total books)\n";
foreach ($books as $book) {
    echo "- {$book['title']} by {$book['author']} — \${$book['price']}\n";
}
?>
```

---

## Building a Complete Pagination System in PHP

A full, real-world pagination implementation:

```php
<?php
// ─── URL: /books?page=2&genre=fiction&sort=price_asc ────────────────────────

class BookPaginator {
    private PDO $pdo;

    public function __construct(PDO $pdo) {
        $this->pdo = $pdo;
    }

    public function paginate(array $filters = [], int $page = 1, int $perPage = 10): array {
        // Sanitize inputs
        $page    = max(1, (int) $page);
        $perPage = max(1, min(100, (int) $perPage));
        $offset  = ($page - 1) * $perPage;

        // Build WHERE conditions dynamically
        $wheres = ["deleted_at IS NULL"];
        $params = [];

        if (!empty($filters['genre'])) {
            $wheres[] = "genre = :genre";
            $params[':genre'] = $filters['genre'];
        }

        if (!empty($filters['min_price'])) {
            $wheres[] = "price >= :min_price";
            $params[':min_price'] = (float) $filters['min_price'];
        }

        if (!empty($filters['max_price'])) {
            $wheres[] = "price <= :max_price";
            $params[':max_price'] = (float) $filters['max_price'];
        }

        if (!empty($filters['search'])) {
            $wheres[] = "(title LIKE :search OR author LIKE :search)";
            $params[':search'] = '%' . $filters['search'] . '%';
        }

        $whereSQL = implode(" AND ", $wheres);

        // Build ORDER BY
        $sortOptions = [
            'price_asc'    => 'price ASC',
            'price_desc'   => 'price DESC',
            'rating_desc'  => 'rating DESC',
            'title_asc'    => 'title ASC',
            'newest'       => 'created_at DESC',
        ];
        $sort    = $filters['sort'] ?? 'newest';
        $orderBy = $sortOptions[$sort] ?? 'created_at DESC';

        // Get total count (same WHERE, no LIMIT)
        $countStmt = $this->pdo->prepare(
            "SELECT COUNT(*) FROM books WHERE $whereSQL"
        );
        $countStmt->execute($params);
        $total = (int) $countStmt->fetchColumn();

        // Get data (with LIMIT + OFFSET)
        $dataStmt = $this->pdo->prepare(
            "SELECT id, title, author, genre, price, rating
             FROM books
             WHERE $whereSQL
             ORDER BY $orderBy
             LIMIT :limit OFFSET :offset"
        );

        // Bind filter params
        foreach ($params as $key => $value) {
            $dataStmt->bindValue($key, $value);
        }
        // Bind pagination params as integers
        $dataStmt->bindValue(':limit',  $perPage, PDO::PARAM_INT);
        $dataStmt->bindValue(':offset', $offset,  PDO::PARAM_INT);
        $dataStmt->execute();
        $data = $dataStmt->fetchAll();

        $totalPages = (int) ceil($total / $perPage);

        return [
            'data'        => $data,           // the actual rows
            'total'       => $total,           // total matching rows
            'per_page'    => $perPage,         // rows per page
            'current_page'=> $page,            // current page number
            'total_pages' => $totalPages,      // how many pages total
            'from'        => $offset + 1,      // first row number on this page
            'to'          => min($offset + $perPage, $total), // last row number
            'has_prev'    => $page > 1,        // is there a previous page?
            'has_next'    => $page < $totalPages, // is there a next page?
            'prev_page'   => $page > 1 ? $page - 1 : null,
            'next_page'   => $page < $totalPages ? $page + 1 : null,
        ];
    }
}

// ─── Usage ───────────────────────────────────────────────────────────────────
$paginator = new BookPaginator($pdo);

$result = $paginator->paginate(
    filters: [
        'genre'     => $_GET['genre']  ?? '',
        'search'    => $_GET['search'] ?? '',
        'sort'      => $_GET['sort']   ?? 'newest',
        'min_price' => $_GET['min']    ?? '',
        'max_price' => $_GET['max']    ?? '',
    ],
    page:    (int) ($_GET['page'] ?? 1),
    perPage: 5
);

// Result structure:
// [
//   'data'         => [...5 book rows...],
//   'total'        => 20,
//   'per_page'     => 5,
//   'current_page' => 2,
//   'total_pages'  => 4,
//   'from'         => 6,
//   'to'           => 10,
//   'has_prev'     => true,
//   'has_next'     => true,
//   'prev_page'    => 1,
//   'next_page'    => 3,
// ]

// Display
echo "Showing {$result['from']}-{$result['to']} of {$result['total']} books\n";
echo "Page {$result['current_page']} of {$result['total_pages']}\n\n";

foreach ($result['data'] as $book) {
    echo "- [{$book['genre']}] {$book['title']} by {$book['author']} — \${$book['price']}\n";
}

if ($result['has_prev']) echo "\n← Previous: ?page={$result['prev_page']}";
if ($result['has_next']) echo "\nNext: ?page={$result['next_page']} →";
?>
```

---

## Common Mistakes

```sql
-- ❌ MISTAKE 1: Using LIMIT without ORDER BY
SELECT * FROM books LIMIT 5;
-- The 5 rows returned are unpredictable — order may change between queries
-- ALWAYS pair LIMIT with ORDER BY

-- ✅ Fix:
SELECT * FROM books ORDER BY id ASC LIMIT 5;

────────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 2: Page number starting from 0 instead of 1
$page   = 0;  // user is on page 0?? Pages should start at 1
$offset = ($page - 1) * 10;  // = -10 → NEGATIVE OFFSET → MySQL error!

-- ✅ Fix: Always enforce page >= 1
$page   = max(1, (int) $_GET['page']);
$offset = ($page - 1) * $perPage;  // minimum offset is 0

────────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 3: Not validating per_page (user sets per_page=999999)
$perPage = (int) $_GET['per_page'];  // user sends 999999
SELECT * FROM books LIMIT 999999;   // loads nearly the entire table!

-- ✅ Fix: Always cap per_page
$perPage = max(1, min(100, (int) ($_GET['per_page'] ?? 10)));
// Now per_page is always between 1 and 100

────────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 4: Forgetting to count total rows
-- Showing "Page 2 of ?" because you forgot to run COUNT(*)
-- Users can't see how many pages there are

-- ✅ Fix: Always run COUNT(*) with the same WHERE clause (but no LIMIT)
SELECT COUNT(*) FROM books WHERE genre = 'fiction';  -- get total
SELECT * FROM books WHERE genre = 'fiction' LIMIT 5 OFFSET 5;  -- get page data

────────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 5: Binding LIMIT/OFFSET as strings in PDO
$stmt->bindValue(':limit', $limit);  // defaults to PDO::PARAM_STR
-- MySQL receives: LIMIT '10' instead of LIMIT 10
-- Some PDO configurations refuse this (error) or add quotes to the SQL

-- ✅ Fix: Always bind as PDO::PARAM_INT
$stmt->bindValue(':limit',  (int) $limit,  PDO::PARAM_INT);
$stmt->bindValue(':offset', (int) $offset, PDO::PARAM_INT);

────────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 6: Not handling the last page correctly
-- Table has 7 books, showing 5 per page:
-- Page 2 query: LIMIT 5 OFFSET 5 → returns only 2 rows (not an error!)
-- Some developers expect exactly 5 rows and break when fewer come back

-- ✅ Fix: Handle partial last pages gracefully
$books = $stmt->fetchAll();  // might return 2 rows on last page — that's fine!
// Don't check count($books) == $perPage to detect last page
// Instead check: $currentPage >= $totalPages
```

---

## Quick Revision

- **`LIMIT n`** — returns at most `n` rows. If fewer rows match, returns all of them. No error if n > available rows.
- **`OFFSET m`** — skips the first `m` rows before returning. Used with LIMIT to paginate.
- **`LIMIT n OFFSET m`** — the standard, readable form: "return n rows, skip the first m."
- **Shorthand `LIMIT m, n`** — same as above but arguments reversed (offset first, count second). Confusing — prefer the standard form.
- **Pagination formula:** `OFFSET = (page - 1) × per_page`. Page 1 → OFFSET 0. Page 2 → OFFSET 10 (if per_page=10). Page 3 → OFFSET 20.
- **Always pair LIMIT with ORDER BY** — without ORDER BY, the rows returned are in unpredictable order and can differ between runs.
- **Always run a COUNT(\*) query** (with the same WHERE but no LIMIT) to get the total number of rows for "Page X of Y" display.
- **LIMIT also works with UPDATE and DELETE** — useful for safe batch operations: `DELETE FROM logs WHERE old = 1 ORDER BY id ASC LIMIT 1000`.
- **Large OFFSET is slow** — `LIMIT 10 OFFSET 1000000` forces MySQL to read and discard 1 million rows. For huge datasets, use keyset/cursor pagination: `WHERE id > last_seen_id LIMIT 10`.
- **In PHP PDO** — always bind LIMIT and OFFSET values as `PDO::PARAM_INT`, never as strings.
- **Always sanitize pagination inputs** — enforce `page >= 1` and cap `per_page` to a reasonable maximum (e.g., 100) to prevent abuse.
- **Total pages formula:** `ceil(total_rows / per_page)` — use PHP's `ceil()` to round up (7 rows ÷ 5 per page = 1.4 → 2 pages).
- **The complete SELECT clause order:** `SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT → OFFSET`.