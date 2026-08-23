# SQL Conditions — WHERE Clause Operators & Filters

The **WHERE clause** is the most powerful tool in SQL for filtering data. Without it, every query returns ALL rows — but in real applications you almost always need specific rows: "give me this user", "show me only active products", "find orders between these dates". WHERE conditions are how you express exactly that.

---

## Table of Contents

1. [What is a Condition?](#what-is-a-condition)
2. [Comparison Operators](#comparison-operators)
   - [= Equal](#-equal)
   - [<> and != Not Equal](#-and--not-equal)
   - [> Greater Than](#-greater-than)
   - [>= Greater Than or Equal](#-greater-than-or-equal)
   - [< Less Than](#-less-than)
   - [<= Less Than or Equal](#-less-than-or-equal)
3. [Logical Operators](#logical-operators)
   - [AND](#and)
   - [OR](#or)
   - [NOT](#not)
   - [Combining AND, OR, NOT](#combining-and-or-not)
4. [BETWEEN](#between)
5. [LIKE — Pattern Matching](#like--pattern-matching)
   - [% Wildcard](#-wildcard)
   - [_ Wildcard](#_-wildcard)
   - [All LIKE Patterns](#all-like-patterns)
6. [IN — Match a List of Values](#in--match-a-list-of-values)
   - [NOT IN](#not-in)
7. [IS NULL and IS NOT NULL](#is-null-and-is-not-null)
8. [Operator Precedence](#operator-precedence)
9. [Combining Everything — Real World Queries](#combining-everything--real-world-queries)
10. [Conditions in PHP with PDO](#conditions-in-php-with-pdo)
11. [Common Mistakes](#common-mistakes)
12. [Quick Revision](#quick-revision)

---

## What is a Condition?

- A **condition** is an expression that evaluates to either **TRUE**, **FALSE**, or **NULL**.
- MySQL returns a row only when the WHERE condition evaluates to **TRUE** for that row.
- Conditions use operators to compare column values against other values.

```sql
-- Full WHERE syntax:
SELECT columns FROM table WHERE condition;

-- A condition compares something:
WHERE price = 12.99          -- is price exactly 12.99?
WHERE stock > 50             -- is stock more than 50?
WHERE genre = 'fiction'      -- is genre the word fiction?
WHERE deleted_at IS NULL     -- is deleted_at empty (not deleted)?

-- MySQL tests each row — if TRUE → include it, if FALSE/NULL → skip it
```

```
How MySQL evaluates WHERE row by row:

books table:
┌────┬─────────────────┬─────────┬────────┐
│ id │ title           │ genre   │ price  │
├────┼─────────────────┼─────────┼────────┤
│  1 │ 1984            │ fiction │ 12.99  │ ← WHERE price < 15 → TRUE  ✅ include
│  2 │ Design Patterns │ program.│ 44.99  │ ← WHERE price < 15 → FALSE ❌ skip
│  3 │ Animal Farm     │ fiction │  8.99  │ ← WHERE price < 15 → TRUE  ✅ include
│  4 │ Clean Code      │ program.│ 35.99  │ ← WHERE price < 15 → FALSE ❌ skip
└────┴─────────────────┴─────────┴────────┘

Result of: SELECT * FROM books WHERE price < 15;
→ Returns rows 1 and 3 only.
```

---

## Comparison Operators

These are the basic building blocks — each one compares a column value to another value.

---

### = Equal

- Returns TRUE when the column value is **exactly equal** to the given value.
- Works on numbers, text, dates — any data type.
- For text: **case-insensitive** with MySQL's default utf8mb4_unicode_ci collation.

```sql
-- Equal on numbers
SELECT id, title, price FROM books WHERE price = 12.99;
-- Finds books that cost EXACTLY $12.99

SELECT id, title, stock FROM books WHERE stock = 100;
-- Finds books with EXACTLY 100 in stock

-- Equal on text (case-insensitive by default)
SELECT id, title, genre FROM books WHERE genre = 'fiction';
SELECT id, title, genre FROM books WHERE genre = 'Fiction';  -- same result!
SELECT id, title, genre FROM books WHERE genre = 'FICTION';  -- same result!

-- Equal on a specific ID (most common use case in apps)
SELECT id, title, author, price FROM books WHERE id = 5;

-- Equal on date
SELECT id, title, created_at FROM books WHERE DATE(created_at) = '2026-01-15';

-- Equal comparing two columns (not just column vs value)
SELECT id, price, sale_price FROM products WHERE price = sale_price;
-- Finds products where the sale price equals the regular price
```

> ⚠️ **Critical Rule:** NEVER use `= NULL` to check for NULL values. `column = NULL` always returns FALSE (NULL is not equal to anything, even itself). Use `IS NULL` instead. This is one of the most common SQL mistakes.

```sql
-- ❌ WRONG — always returns 0 rows
SELECT * FROM users WHERE deleted_at = NULL;

-- ✅ CORRECT
SELECT * FROM users WHERE deleted_at IS NULL;
```

---

### <> and != Not Equal

- Returns TRUE when the column value is **NOT equal** to the given value.
- `<>` is the SQL standard. `!=` is MySQL-specific but works identically. Both are widely used.

```sql
-- Not equal on text
SELECT id, title, genre FROM books WHERE genre <> 'fiction';
-- Returns all books that are NOT fiction (horror, programming, science, history)

SELECT id, title, genre FROM books WHERE genre != 'programming';
-- Returns all books that are NOT programming books
-- <> and != are identical — choose one and be consistent

-- Not equal on numbers
SELECT id, title, stock FROM books WHERE stock <> 0;
-- Returns all books that have at least some stock (not out of stock)

SELECT id, title, rating FROM books WHERE rating != 5.0;
-- Returns all books that aren't rated perfectly

-- Not equal on status
SELECT id, name, status FROM users WHERE status <> 'active';
-- Returns inactive and banned users (everyone who isn't active)

SELECT id, name, role FROM users WHERE role != 'admin';
-- Returns everyone who is not an admin

-- Combining: not equal to multiple values (better to use NOT IN — covered later)
SELECT * FROM books WHERE genre <> 'fiction' AND genre <> 'horror';
-- Returns books that are neither fiction NOR horror
```

---

### > Greater Than

- Returns TRUE when the column value is **strictly greater than** the given value.
- The given value itself is NOT included.

```sql
-- Numbers greater than a value
SELECT id, title, price FROM books WHERE price > 20;
-- Returns books costing MORE than $20 (not $20 exactly — $20.01 and above)

SELECT id, title, stock FROM books WHERE stock > 100;
-- Returns books with more than 100 copies in stock

SELECT id, title, rating FROM books WHERE rating > 4.5;
-- Returns books rated higher than 4.5 (4.6, 4.7, 4.8, 4.9 — not 4.5 itself)

-- Dates greater than (after) a date
SELECT id, title, created_at FROM books WHERE created_at > '2026-01-01';
-- Returns books added AFTER January 1, 2026 (not on that date)

SELECT id, name, created_at FROM users WHERE created_at > '2025-06-01 00:00:00';
-- Returns users registered after June 1, 2025

-- Practical: find products running low on stock but not empty
SELECT id, title, stock FROM books WHERE stock > 0 AND stock < 50;
```

---

### >= Greater Than or Equal

- Returns TRUE when the column value is **greater than OR exactly equal** to the given value.
- The given value IS included.

```sql
-- Greater than or equal on numbers
SELECT id, title, price FROM books WHERE price >= 20;
-- Returns books costing $20 OR MORE (includes $20.00 exactly)

SELECT id, title, rating FROM books WHERE rating >= 4.5;
-- Returns books rated 4.5 OR HIGHER (includes 4.5 itself)

-- Very common pattern: minimum stock check
SELECT id, title, stock FROM books WHERE stock >= 100;
-- Returns books with 100 or more copies in stock

-- Dates: from a specific date onward (inclusive)
SELECT id, name, created_at FROM users WHERE created_at >= '2026-01-01';
-- Returns users who registered on OR after January 1, 2026

-- Age check: users who are 18 or older
SELECT id, name, age FROM users WHERE age >= 18;

-- Minimum rating filter (e.g., show only well-rated products)
SELECT id, title, rating FROM books WHERE rating >= 4.0;
```

---

### < Less Than

- Returns TRUE when the column value is **strictly less than** the given value.
- The given value itself is NOT included.

```sql
-- Numbers less than a value
SELECT id, title, price FROM books WHERE price < 15;
-- Returns books costing LESS THAN $15 (not $15 exactly — $14.99 and below)

SELECT id, title, stock FROM books WHERE stock < 50;
-- Returns books with fewer than 50 copies in stock (might need restocking)

-- Dates less than (before) a date
SELECT id, name, created_at FROM users WHERE created_at < '2025-01-01';
-- Returns users registered BEFORE January 1, 2025

-- Practical: find books running low (under 40 in stock)
SELECT id, title, stock FROM books WHERE stock < 40;
-- Use for restocking alerts
```

---

### <= Less Than or Equal

- Returns TRUE when the column value is **less than OR exactly equal** to the given value.
- The given value IS included.

```sql
-- Less than or equal on numbers
SELECT id, title, price FROM books WHERE price <= 15;
-- Returns books costing $15 OR LESS (includes $15.00 exactly)

SELECT id, title, stock FROM books WHERE stock <= 40;
-- Returns books with 40 or fewer copies in stock

-- Price range filter (budget books at $15 or less)
SELECT id, title, price FROM books WHERE price <= 15.00;

-- Dates: up to and including a specific date
SELECT id, name, created_at FROM users WHERE created_at <= '2025-12-31';
-- Returns users who registered on or before December 31, 2025

-- Practical: orders that should have been delivered by now
SELECT id, total, expected_delivery FROM orders
WHERE expected_delivery <= NOW() AND status != 'delivered';
-- Orders whose delivery date has passed but aren't delivered yet
```

### All Comparison Operators at a Glance

| Operator | Meaning | Includes the value? | Example |
|---|---|---|---|
| `=` | Equal | ✅ Yes | `price = 12.99` |
| `<>` | Not equal (standard) | — | `genre <> 'fiction'` |
| `!=` | Not equal (MySQL) | — | `genre != 'fiction'` |
| `>` | Greater than | ❌ No | `price > 20` |
| `>=` | Greater than or equal | ✅ Yes | `price >= 20` |
| `<` | Less than | ❌ No | `price < 15` |
| `<=` | Less than or equal | ✅ Yes | `price <= 15` |

---

## Logical Operators

Logical operators combine multiple conditions into one. They let you express complex questions like "fiction books AND priced under $15" or "programming OR science books".

---

### AND

- Returns TRUE only when **ALL conditions** are TRUE.
- If even ONE condition is FALSE → the entire AND expression is FALSE → row is skipped.
- Think of AND as "this AND that AND that — all must be true."

```sql
-- Two conditions: both must be true
SELECT id, title, genre, price
FROM books
WHERE genre = 'fiction' AND price < 15;
-- Must be fiction AND must cost less than $15
-- A fiction book at $16 → genre check PASSES, price check FAILS → SKIPPED
-- A $12 horror book → genre check FAILS → SKIPPED immediately

-- Three conditions: all must be true
SELECT id, title, genre, price, rating
FROM books
WHERE genre = 'fiction'
  AND price < 15
  AND rating >= 4.5;
-- Must be fiction, AND under $15, AND rated 4.5 or higher

-- Multiple AND conditions
SELECT id, name, role, status, created_at
FROM users
WHERE role    = 'admin'
  AND status  = 'active'
  AND deleted_at IS NULL
  AND created_at >= '2025-01-01';
-- Active admins, not deleted, registered from 2025 onward

-- Practical: find books low in stock that are also popular
SELECT id, title, stock, rating
FROM books
WHERE stock < 50 AND rating >= 4.5;
-- Low stock AND highly rated → priority restock candidates
```

```
AND truth table:
condition1 AND condition2 = result

TRUE  AND TRUE  = TRUE   ← row included
TRUE  AND FALSE = FALSE  ← row skipped
FALSE AND TRUE  = FALSE  ← row skipped
FALSE AND FALSE = FALSE  ← row skipped
```

---

### OR

- Returns TRUE when **AT LEAST ONE condition** is TRUE.
- A row is only skipped when ALL conditions are FALSE.
- Think of OR as "this OR that — at least one must be true."

```sql
-- Two conditions: at least one must be true
SELECT id, title, genre
FROM books
WHERE genre = 'fiction' OR genre = 'horror';
-- Returns fiction books AND horror books
-- (Better done with IN — covered later)

-- Different columns with OR
SELECT id, title, price, stock
FROM books
WHERE price > 40 OR stock < 35;
-- Returns books that are very expensive OR very low in stock (or both)

-- Three conditions: any one can trigger a match
SELECT id, title, author
FROM books
WHERE author = 'Stephen King'
   OR author = 'J.K. Rowling'
   OR author = 'George Orwell';
-- Returns books by any of these three authors

-- Practical: alert for any kind of problem
SELECT id, title, stock, price
FROM books
WHERE stock = 0             -- out of stock
   OR price < 0             -- invalid price
   OR rating = 0.0;         -- no rating yet
-- Returns books with ANY of these issues
```

```
OR truth table:
condition1 OR condition2 = result

TRUE  OR TRUE  = TRUE   ← row included
TRUE  OR FALSE = TRUE   ← row included
FALSE OR TRUE  = TRUE   ← row included
FALSE OR FALSE = FALSE  ← row skipped (only case where OR returns FALSE)
```

---

### NOT

- **Reverses** the result of a condition — TRUE becomes FALSE, FALSE becomes TRUE.
- Used as `NOT condition` or `NOT (condition)`.

```sql
-- NOT equal (same as <> or !=)
SELECT id, title, genre FROM books WHERE NOT genre = 'fiction';
-- Same as: WHERE genre <> 'fiction'

-- NOT with LIKE (covered more in LIKE section)
SELECT id, title FROM books WHERE NOT title LIKE 'Harry%';
-- Books whose titles do NOT start with "Harry"

-- NOT with BETWEEN
SELECT id, title, price FROM books WHERE NOT price BETWEEN 10 AND 20;
-- Books priced OUTSIDE the $10–$20 range
-- Same as: WHERE price < 10 OR price > 20

-- NOT with IN
SELECT id, title, genre FROM books WHERE NOT genre IN ('fiction', 'horror');
-- Same as: WHERE genre NOT IN ('fiction', 'horror')

-- NOT with NULL check
SELECT id, title, published_at FROM books WHERE NOT published_at IS NULL;
-- Same as: WHERE published_at IS NOT NULL
-- (but IS NOT NULL is clearer — prefer that form)

-- NOT with a complex condition in parentheses
SELECT id, title, genre, price
FROM books
WHERE NOT (genre = 'fiction' AND price > 20);
-- Returns all books EXCEPT fiction books that cost more than $20
-- Equivalent to: WHERE genre != 'fiction' OR price <= 20 (De Morgan's law)
```

---

### Combining AND, OR, NOT

When mixing AND and OR, **AND has higher precedence than OR** — it evaluates first, just like multiplication before addition in math. Use parentheses to control the order explicitly.

```sql
-- ❌ AMBIGUOUS — hard to read, might not do what you think
SELECT * FROM books WHERE genre = 'fiction' OR genre = 'horror' AND price < 15;

-- MySQL sees this as:
-- genre = 'fiction'  OR  (genre = 'horror' AND price < 15)
-- Because AND evaluates before OR!
-- Returns: ALL fiction books (any price) + horror books under $15

-- ✅ EXPLICIT with parentheses — clear intent
-- Option A: fiction or horror books, BOTH must be under $15
SELECT * FROM books WHERE (genre = 'fiction' OR genre = 'horror') AND price < 15;

-- Option B: fiction (any price) OR cheap horror books
SELECT * FROM books WHERE genre = 'fiction' OR (genre = 'horror' AND price < 15);

-- Real example — find users to notify:
-- (admin OR editor) AND active AND not deleted
SELECT id, name, role, status
FROM users
WHERE (role = 'admin' OR role = 'editor')
  AND status = 'active'
  AND deleted_at IS NULL;

-- Without parentheses this would be:
-- role = 'admin' OR (role = 'editor' AND status = 'active' AND deleted_at IS NULL)
-- Which would return ALL admins regardless of status!
```

> 💡 **Golden Rule:** Always use parentheses when mixing AND and OR. Even when you know the precedence rules, parentheses make your intention crystal clear to anyone reading the code later.

---

## BETWEEN

- Returns TRUE when a value falls within a **range** (inclusive on both ends).
- Works on numbers, dates, and text.
- `BETWEEN value1 AND value2` includes BOTH value1 and value2.
- Equivalent to `>= value1 AND <= value2`.

```sql
-- Syntax
WHERE column_name BETWEEN value1 AND value2;
-- IMPORTANT: value1 must be the SMALLER value, value2 the LARGER

-- Numbers: find books in a price range
SELECT id, title, price
FROM books
WHERE price BETWEEN 10 AND 20;
-- Returns books priced from $10.00 to $20.00 (both ends included)
-- Same as: WHERE price >= 10 AND price <= 20

-- Verify: is $10 included?  YES — BETWEEN is inclusive
-- Verify: is $20 included?  YES — BETWEEN is inclusive
-- Verify: is $9.99 included? NO  — below the lower bound
-- Verify: is $20.01 included? NO — above the upper bound

-- Different price ranges
SELECT id, title, price FROM books WHERE price BETWEEN 10 AND 15;   -- budget
SELECT id, title, price FROM books WHERE price BETWEEN 15 AND 25;   -- mid range
SELECT id, title, price FROM books WHERE price BETWEEN 30 AND 50;   -- premium

-- Stock range: books with 50 to 100 copies
SELECT id, title, stock FROM books WHERE stock BETWEEN 50 AND 100;

-- Rating range: books rated between 4.5 and 5.0
SELECT id, title, rating FROM books WHERE rating BETWEEN 4.5 AND 5.0;

-- Dates: find books added in a specific date range
SELECT id, title, created_at
FROM books
WHERE created_at BETWEEN '2026-01-01' AND '2026-06-30 23:59:59';
-- Books added in the first half of 2026
-- Include the time portion for TIMESTAMP columns!

-- Date range for orders this month
SELECT id, total, created_at
FROM orders
WHERE created_at BETWEEN '2026-06-01 00:00:00' AND '2026-06-30 23:59:59';

-- Text range (alphabetical): books whose titles fall between A and M
SELECT id, title FROM books WHERE title BETWEEN 'A' AND 'N';
-- Alphabetically from A to M (N is excluded from results as it's the boundary)
-- Less common — most developers use LIKE for text patterns
```

### NOT BETWEEN

```sql
-- NOT BETWEEN: outside the range
SELECT id, title, price
FROM books
WHERE price NOT BETWEEN 10 AND 20;
-- Returns books cheaper than $10 OR more expensive than $20
-- Same as: WHERE price < 10 OR price > 20

SELECT id, title, stock
FROM books
WHERE stock NOT BETWEEN 50 AND 150;
-- Books with very low or very high stock (outside the normal range)
```

> ⚠️ **Warning:** `BETWEEN value1 AND value2` requires value1 ≤ value2. If you write `BETWEEN 20 AND 10` (bigger first), MySQL returns zero rows — it won't swap them for you.

```sql
-- ❌ WRONG — value1 > value2 → no rows returned
SELECT * FROM books WHERE price BETWEEN 20 AND 10;  -- 0 rows!

-- ✅ CORRECT — smaller value first
SELECT * FROM books WHERE price BETWEEN 10 AND 20;
```

---

## LIKE — Pattern Matching

- **LIKE** searches for a **pattern** within text values — partial matches, starts with, ends with, contains.
- Uses two wildcard characters: `%` and `_`.
- Case-insensitive by default with MySQL's utf8mb4_unicode_ci collation.

```sql
-- Syntax
WHERE column_name LIKE 'pattern';
```

---

### % Wildcard

- `%` represents **zero or more characters** — any characters, any length.

```sql
-- 'a%' — starts with 'a' (anything after)
SELECT id, title FROM books WHERE title LIKE 'A%';
-- Matches: Animal Farm, A Brief History of Time
-- Does NOT match: The Alchemist (starts with T)

-- '%a' — ends with 'a' (anything before)
SELECT id, title FROM books WHERE title LIKE '%a';
-- Matches: Carrie (ends with e? No. Let me check actual data)
-- Actually matches titles ending in lowercase 'a'
-- Case-insensitive: matches ending in A or a

-- '%or%' — contains 'or' anywhere in the string
SELECT id, title FROM books WHERE title LIKE '%or%';
-- Matches: Harry Potter PS (Potter), Harry Potter CoS, Harry Potter PoA,
--          Harry Potter PoA, George Orwell... any title with "or" anywhere

-- Start pattern examples with our bookstore:
SELECT id, title FROM books WHERE title LIKE 'Harry%';
-- Matches: Harry Potter PS, Harry Potter CoS, Harry Potter PoA

SELECT id, title FROM books WHERE title LIKE 'The%';
-- Matches: The Shining, The Grand Design, The Alchemist, The Pragmatic Programmer

SELECT id, author FROM books WHERE author LIKE 'Stephen%';
-- Matches: Stephen King, Stephen Hawking

-- End pattern: all books ending in 'JS'
SELECT id, title FROM books WHERE title LIKE '%JS';
-- Matches: You Don't Know JS, Eloquent JavaScript? No — only ending in 'JS'
-- Matches: You Don't Know JS

-- Contains pattern: titles containing 'Potter'
SELECT id, title FROM books WHERE title LIKE '%Potter%';
-- Matches: Harry Potter PS, Harry Potter CoS, Harry Potter PoA

-- Contains: author names with 'Harari'
SELECT id, title, author FROM books WHERE author LIKE '%Harari%';
-- Matches: Sapiens, Homo Deus, 21 Lessons (all by Yuval Noah Harari)

-- Practical: search any word in title or author
SELECT id, title, author FROM books
WHERE title LIKE '%clean%' OR author LIKE '%clean%';
-- Matches: Clean Code (title match)
```

---

### _ Wildcard

- `_` represents **exactly ONE character** — any single character.
- Unlike `%` which matches zero or more, `_` matches precisely one.

```sql
-- '_r%' — second character is 'r' (any first char, 'r' second, anything after)
SELECT id, title FROM books WHERE title LIKE '_r%';
-- Matches titles where the 2nd character is 'r'
-- Example: "bravery" (b-r), "Greed" (G-r)
-- In our data: probably nothing matches (let's use a better example)

-- 'a_%' — starts with 'a', has at least 2 characters
SELECT id, title FROM books WHERE title LIKE 'a_%';
-- 'a' is 1st char, '_' ensures at least 1 more char, '%' allows anything after
-- Matches: Animal Farm, A Brief History of Time (if case-insensitive)

-- 'a_%_%' — starts with 'a', has at least 3 characters
SELECT id, title FROM books WHERE title LIKE 'a_%_%';
-- Must start with 'a' + at least 2 more chars
-- 'ab' would NOT match (only 2 chars total)
-- 'abc' would match (a + b + c = 3 chars minimum)

-- 'a%o' — starts with 'a', ends with 'o' (anything in between)
SELECT id, title FROM books WHERE title LIKE 'a%o';
-- Matches titles starting with 'a' AND ending with 'o'
-- Example: "Audio" matches, "And so" matches

-- Phone format validation example (10-digit phone)
SELECT id, name, phone FROM users WHERE phone LIKE '__________';
-- 10 underscores = exactly 10 characters
-- Matches: "0912345678" (10 digits)
-- Does NOT match: "091234567" (9 digits) or "09123456789" (11 digits)

-- Match a specific pattern position:
SELECT id, title FROM books WHERE title LIKE '_____ %';
-- Title where first 5 chars are anything, 6th is a space
-- Matches: "Clean Code" (Clean + space + Code)
```

---

### All LIKE Patterns

```sql
-- ─── THE ESSENTIAL PATTERNS ─────────────────────────────────────────────────

-- 'abc%'  → Starts with "abc"
SELECT * FROM books WHERE title LIKE 'Harry%';
-- Matches: "Harry Potter PS", "Harry Potter CoS", "Harry Houdini"
-- Does NOT match: "The Harry Potter", "harry potter" (case-insensitive: DOES match)

-- '%abc'  → Ends with "abc"
SELECT * FROM books WHERE title LIKE '%Code';
-- Matches: "Clean Code"
-- Does NOT match: "Code Complete"

-- '%abc%' → Contains "abc" anywhere
SELECT * FROM books WHERE title LIKE '%Potter%';
-- Matches: "Harry Potter PS", "Harry Potter CoS", "Harry Potter PoA"

-- 'a%o'   → Starts with 'a' AND ends with 'o'
SELECT * FROM books WHERE title LIKE 'A%o';
-- Matches: "A Brief History" → No (ends with y)
-- Matches: "Animal" → No (ends with l)
-- This would match something like "Auto" or "Allegro"

-- '_r%'   → Second character is 'r'
SELECT * FROM books WHERE author LIKE '_r%';
-- Matches: authors where 2nd letter is 'r'
-- Example: "Bram Stoker" (B-r), "Ernest" (E-r)
-- In our data: probably none

-- 'a_%'   → Starts with 'a', at least 2 characters total
SELECT * FROM books WHERE title LIKE 'A_%';
-- 'A' alone would NOT match (needs at least one more char)
-- 'A Brief History of Time' DOES match

-- 'a_%_%' → Starts with 'a', at least 3 characters total
SELECT * FROM books WHERE title LIKE 'A_%_%';
-- 'An' would NOT match (only 2 chars)
-- 'And' WOULD match (3 chars, starting with A)

-- '___'   → Exactly 3 characters
SELECT * FROM users WHERE country_code LIKE '___';
-- Matches 3-letter country codes: 'MMR', 'USA', 'GBR'
-- Does NOT match: 'MM' (2 chars), 'BURM' (4 chars)
```

### NOT LIKE

```sql
-- NOT LIKE: rows that do NOT match the pattern
SELECT id, title FROM books WHERE title NOT LIKE 'Harry%';
-- All books EXCEPT Harry Potter ones

SELECT id, title, author FROM books WHERE author NOT LIKE 'Stephen%';
-- Books by authors whose name doesn't start with Stephen

SELECT id, name, email FROM users WHERE email NOT LIKE '%@gmail.com';
-- Users NOT using Gmail addresses
```

### LIKE with Special Characters — Escaping

```sql
-- If the search term itself contains % or _, escape with backslash
-- Find products whose name literally contains "50%" 
SELECT * FROM products WHERE name LIKE '%50\%%';
-- \% is a literal percent sign, not a wildcard

-- Find a column that literally contains an underscore
SELECT * FROM codes WHERE code LIKE '%\_%';
-- \_ is a literal underscore, not the one-character wildcard
```

---

## IN — Match a List of Values

- **IN** checks if a value matches **any value in a list**.
- Much cleaner than writing multiple OR conditions for the same column.
- Works with numbers, text, dates — any data type.

```sql
-- Syntax
WHERE column_name IN (value1, value2, value3, ...);

-- Without IN (verbose — multiple OR):
SELECT * FROM books
WHERE genre = 'fiction' OR genre = 'horror' OR genre = 'science';

-- With IN (clean and readable):
SELECT * FROM books WHERE genre IN ('fiction', 'horror', 'science');
-- Same result, much cleaner!
```

```sql
-- IN with text
SELECT id, title, genre, price
FROM books
WHERE genre IN ('fiction', 'history', 'science');
-- Returns all books that belong to fiction OR history OR science genre

SELECT id, title, author
FROM books
WHERE author IN ('Stephen King', 'J.K. Rowling', 'George Orwell');
-- Returns books by any of these three authors

-- IN with numbers
SELECT id, title, price
FROM books
WHERE id IN (1, 5, 10, 15, 20);
-- Returns specific books by their IDs

SELECT id, name, role
FROM users
WHERE id IN (42, 101, 205, 308);
-- Fetch specific users by ID list

-- IN with status/role
SELECT id, name, role
FROM users
WHERE role IN ('admin', 'editor', 'moderator');
-- Users who are admin, editor, OR moderator

SELECT id, total, status
FROM orders
WHERE status IN ('pending', 'processing', 'confirmed');
-- All orders that are in an "active" state

-- IN with subquery (advanced — query inside a query)
SELECT id, title FROM books
WHERE author_id IN (
  SELECT id FROM authors WHERE country = 'United Kingdom'
);
-- Books by UK authors (the inner SELECT provides the list)
```

### NOT IN

```sql
-- NOT IN: exclude rows matching any value in the list
SELECT id, title, genre
FROM books
WHERE genre NOT IN ('programming', 'science');
-- Returns all books EXCEPT programming and science genres

SELECT id, name, status
FROM users
WHERE status NOT IN ('banned', 'inactive');
-- Returns only active users (excludes banned and inactive)

SELECT id, total, status
FROM orders
WHERE status NOT IN ('cancelled', 'refunded');
-- Returns all orders that are still valid (not cancelled or refunded)

-- NOT IN with numbers
SELECT id, title FROM books WHERE id NOT IN (1, 2, 3);
-- Returns all books EXCEPT books with id 1, 2, or 3
```

> ⚠️ **Warning: NOT IN and NULL don't mix well!**

```sql
-- If the IN list contains NULL, NOT IN returns NO rows at all!
-- This is a very common and confusing bug:

SELECT * FROM books WHERE id NOT IN (1, 2, NULL);
-- Returns 0 rows! (because NULL comparison is unknown)
-- MySQL can't determine if id is "not equal to NULL" — it's undefined

-- ✅ Safe fix: make sure your IN list contains no NULLs
-- Or use NOT EXISTS instead when working with subqueries that might return NULL
```

---

## IS NULL and IS NOT NULL

- `IS NULL` checks if a value is NULL (missing/unknown).
- `IS NOT NULL` checks if a value is NOT NULL (has a real value).
- You CANNOT use `= NULL` or `!= NULL` — they always return FALSE.

```sql
-- IS NULL — find rows where the column has no value
SELECT id, title, published_at
FROM books
WHERE published_at IS NULL;
-- Books that haven't been published yet (or published_at not filled in)

SELECT id, name, phone
FROM users
WHERE phone IS NULL;
-- Users who haven't provided a phone number

SELECT id, total, deleted_at
FROM orders
WHERE deleted_at IS NULL;
-- Orders that have NOT been soft-deleted (active orders)

-- IS NOT NULL — find rows where the column HAS a value
SELECT id, name, phone
FROM users
WHERE phone IS NOT NULL;
-- Users who HAVE provided a phone number

SELECT id, total, deleted_at
FROM orders
WHERE deleted_at IS NOT NULL;
-- Orders that have been soft-deleted

SELECT id, title, published_at
FROM books
WHERE published_at IS NOT NULL;
-- Books that have a published date set

-- Combining IS NULL with other conditions
SELECT id, name, email, phone, created_at
FROM users
WHERE status = 'active'
  AND phone IS NULL
  AND created_at >= '2026-01-01';
-- Active users registered this year who haven't added their phone yet
-- → Good candidate for a "complete your profile" email campaign
```

---

## Operator Precedence

When you mix different operators, MySQL evaluates them in this order (highest to lowest):

```
Precedence Order (highest first):
  1. ()         → Parentheses (always evaluated first)
  2. NOT        → Logical NOT
  3. AND        → Logical AND
  4. OR         → Logical OR
  5. BETWEEN, LIKE, IN, IS NULL  → Range/pattern operators
  6. =, <>, !=, >, >=, <, <=    → Comparison operators

Think of it like math:
  AND is like × (multiplication)
  OR  is like + (addition)
  Multiplication before addition — AND before OR
```

```sql
-- Example showing why precedence matters:
SELECT * FROM books WHERE genre = 'fiction' OR genre = 'horror' AND price < 15;

-- MySQL reads this as:
-- genre = 'fiction' OR (genre = 'horror' AND price < 15)
-- → Returns: ALL fiction (any price) + cheap horror books

-- If you meant: (fiction OR horror) that is cheap:
SELECT * FROM books WHERE (genre = 'fiction' OR genre = 'horror') AND price < 15;
-- Completely different result!

-- ALWAYS use parentheses to remove ambiguity:
-- ✅ Clear intent:
WHERE (role = 'admin' OR role = 'editor') AND status = 'active' AND deleted_at IS NULL
-- ❌ Ambiguous (AND evaluates before OR):
WHERE role = 'admin' OR role = 'editor' AND status = 'active' AND deleted_at IS NULL
```

---

## Combining Everything — Real World Queries

```sql
-- ─── E-COMMERCE ─────────────────────────────────────────────────────────────

-- Customer searches for "affordable fiction books with good reviews"
SELECT id, title, author, price, rating
FROM books
WHERE genre = 'fiction'
  AND price BETWEEN 10 AND 20
  AND rating >= 4.5
ORDER BY rating DESC, price ASC;

-- Admin dashboard: problematic products
SELECT id, title, stock, price, rating
FROM books
WHERE stock = 0                        -- out of stock
   OR price <= 0                       -- invalid price
   OR (rating < 3.0 AND rating > 0)   -- poorly rated
ORDER BY stock ASC, rating ASC;

-- Seasonal sale: discount certain genres
SELECT id, title, genre, price,
  ROUND(price * 0.8, 2) AS sale_price
FROM books
WHERE genre IN ('fiction', 'history')
  AND price BETWEEN 10 AND 30
  AND deleted_at IS NULL
ORDER BY price ASC;

-- Search: user types "harry potter" in the search box
SELECT id, title, author, price, genre
FROM books
WHERE (title  LIKE '%harry%' OR title  LIKE '%potter%'
   OR  author LIKE '%harry%' OR author LIKE '%potter%')
  AND deleted_at IS NULL
ORDER BY rating DESC;

-- ─── USER MANAGEMENT ────────────────────────────────────────────────────────

-- Find users to send "re-engagement" email
SELECT id, name, email, last_login_at
FROM users
WHERE status = 'active'
  AND deleted_at IS NULL
  AND last_login_at BETWEEN
    (NOW() - INTERVAL 90 DAY) AND (NOW() - INTERVAL 30 DAY)
  AND email NOT LIKE '%@test.com'
  AND email NOT LIKE '%@example.com';
-- Active users who haven't logged in for 30-90 days (not recent, not too old)
-- Exclude test/example email addresses

-- Admin search for a user
SELECT id, name, email, role, status, created_at
FROM users
WHERE (name  LIKE '%phyo%'
    OR email LIKE '%phyo%')
  AND deleted_at IS NULL
ORDER BY created_at DESC
LIMIT 20;

-- New users this week by role
SELECT role, COUNT(*) AS count
FROM users
WHERE created_at >= (NOW() - INTERVAL 7 DAY)
  AND status != 'banned'
  AND deleted_at IS NULL
GROUP BY role
ORDER BY count DESC;

-- ─── ORDERS ─────────────────────────────────────────────────────────────────

-- Orders that need attention (pending or processing, over 2 days old)
SELECT id, user_id, total, status, created_at
FROM orders
WHERE status IN ('pending', 'processing')
  AND created_at < (NOW() - INTERVAL 2 DAY)
  AND deleted_at IS NULL
ORDER BY created_at ASC;

-- High-value orders this month
SELECT id, user_id, total, status
FROM orders
WHERE total >= 100
  AND created_at BETWEEN '2026-06-01' AND '2026-06-30 23:59:59'
  AND status NOT IN ('cancelled', 'refunded')
ORDER BY total DESC;
```

---

## Conditions in PHP with PDO

Always use prepared statements — never concatenate user input into SQL.

```php
<?php
// ─── SINGLE CONDITION ───────────────────────────────────────────────────────
$stmt = $pdo->prepare("SELECT * FROM books WHERE id = :id");
$stmt->execute([':id' => 5]);
$book = $stmt->fetch();

// ─── COMPARISON OPERATORS ───────────────────────────────────────────────────
$stmt = $pdo->prepare(
    "SELECT id, title, price FROM books
     WHERE price >= :min AND price <= :max
     ORDER BY price ASC"
);
$stmt->execute([':min' => 10.00, ':max' => 30.00]);
$books = $stmt->fetchAll();

// ─── BETWEEN ────────────────────────────────────────────────────────────────
$stmt = $pdo->prepare(
    "SELECT id, title, price FROM books WHERE price BETWEEN :min AND :max"
);
$stmt->bindValue(':min', 10.00, PDO::PARAM_STR);  // DECIMAL → PARAM_STR
$stmt->bindValue(':max', 30.00, PDO::PARAM_STR);
$stmt->execute();
$books = $stmt->fetchAll();

// ─── LIKE (search) ──────────────────────────────────────────────────────────
$search = '%' . $_GET['q'] . '%';  // add wildcards around user input
$stmt   = $pdo->prepare(
    "SELECT id, title, author FROM books
     WHERE title LIKE :search OR author LIKE :search
     ORDER BY rating DESC
     LIMIT 20"
);
$stmt->bindValue(':search', $search, PDO::PARAM_STR);
$stmt->execute();
$results = $stmt->fetchAll();

// ─── IN (dynamic list) ──────────────────────────────────────────────────────
// Building IN clause dynamically (safe way with placeholders)
$genres    = ['fiction', 'history', 'science'];  // from filter checkboxes
$placeholders = implode(',', array_fill(0, count($genres), '?'));  // ?,?,?

$stmt = $pdo->prepare(
    "SELECT id, title, genre, price FROM books
     WHERE genre IN ($placeholders)
     ORDER BY title ASC"
);
$stmt->execute($genres);
$books = $stmt->fetchAll();

// ─── DYNAMIC WHERE BUILDER ───────────────────────────────────────────────────
// Build query dynamically based on which filters are set
function searchBooks(PDO $pdo, array $filters): array {
    $wheres = ['deleted_at IS NULL'];
    $params = [];

    // Genre filter
    if (!empty($filters['genre'])) {
        $wheres[]          = 'genre = :genre';
        $params[':genre']  = $filters['genre'];
    }

    // Price range
    if (!empty($filters['min_price'])) {
        $wheres[]             = 'price >= :min_price';
        $params[':min_price'] = (float) $filters['min_price'];
    }
    if (!empty($filters['max_price'])) {
        $wheres[]             = 'price <= :max_price';
        $params[':max_price'] = (float) $filters['max_price'];
    }

    // Text search
    if (!empty($filters['search'])) {
        $wheres[]           = '(title LIKE :search OR author LIKE :search)';
        $params[':search']  = '%' . $filters['search'] . '%';
    }

    // Minimum rating
    if (!empty($filters['min_rating'])) {
        $wheres[]              = 'rating >= :min_rating';
        $params[':min_rating'] = (float) $filters['min_rating'];
    }

    // In stock only
    if (!empty($filters['in_stock'])) {
        $wheres[] = 'stock > 0';
    }

    $sql  = "SELECT id, title, author, genre, price, rating, stock
             FROM books
             WHERE " . implode(' AND ', $wheres) . "
             ORDER BY rating DESC
             LIMIT 20";

    $stmt = $pdo->prepare($sql);
    $stmt->execute($params);
    return $stmt->fetchAll();
}

// Usage:
$books = searchBooks($pdo, [
    'genre'      => 'fiction',
    'min_price'  => 10,
    'max_price'  => 20,
    'min_rating' => 4.5,
    'in_stock'   => true,
]);
?>
```

---

## Common Mistakes

```sql
-- ❌ MISTAKE 1: Using = NULL instead of IS NULL
WHERE deleted_at = NULL     -- ALWAYS returns 0 rows (NULL ≠ anything)
WHERE phone = NULL          -- Same problem
-- ✅ Fix:
WHERE deleted_at IS NULL
WHERE phone IS NULL

────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 2: BETWEEN with wrong order (bigger first)
WHERE price BETWEEN 30 AND 10   -- 0 rows! Value1 must be ≤ value2
-- ✅ Fix:
WHERE price BETWEEN 10 AND 30

────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 3: Forgetting parentheses with OR and AND
WHERE status = 'active' OR role = 'admin' AND deleted_at IS NULL
-- MySQL reads: status='active' OR (role='admin' AND deleted_at IS NULL)
-- Returns ALL active users (even deleted ones!)
-- ✅ Fix:
WHERE (status = 'active' OR role = 'admin') AND deleted_at IS NULL

────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 4: NOT IN with a list containing NULL
WHERE id NOT IN (1, 2, NULL)  -- Returns 0 rows!
-- ✅ Fix: ensure no NULLs in your IN list
WHERE id NOT IN (1, 2)        -- Works correctly

────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 5: LIKE '%word%' on a huge table (slow)
WHERE title LIKE '%php%'   -- Starts with %, can't use index → full table scan!
-- ✅ Better: use FULLTEXT index for large tables
WHERE MATCH(title) AGAINST('php' IN NATURAL LANGUAGE MODE)

────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 6: Using OR when IN is cleaner
WHERE genre = 'fiction' OR genre = 'horror' OR genre = 'science' OR genre = 'history'
-- ✅ Much cleaner with IN:
WHERE genre IN ('fiction', 'horror', 'science', 'history')

────────────────────────────────────────────────────────────────

-- ❌ MISTAKE 7: LIKE pattern without % (becomes same as =)
WHERE name LIKE 'Phyo'   -- Only matches exact string "Phyo" (same as = 'Phyo')
-- ✅ Add wildcards if you want partial matching:
WHERE name LIKE 'Phyo%'  -- Matches "Phyo", "Phyo Min", "Phyo Jr."
WHERE name LIKE '%Phyo%' -- Matches "Phyo", "Mr Phyo", "Phyo Min Paing"
```

---

## Quick Revision

- A **condition** evaluates to TRUE or FALSE for each row. MySQL only returns rows where the condition is TRUE.
- **`=`** exact match. **`<>` / `!=`** not equal (same thing). **`>`** greater than (excludes value). **`>=`** greater than or equal (includes value). **`<`** less than. **`<=`** less than or equal.
- **`AND`** — ALL conditions must be TRUE. One FALSE → whole thing is FALSE.
- **`OR`** — AT LEAST ONE condition must be TRUE. All FALSE → row is skipped.
- **`NOT`** — reverses the condition. TRUE → FALSE, FALSE → TRUE.
- **Precedence:** AND evaluates before OR — always use parentheses when mixing them: `WHERE (a OR b) AND c`.
- **`BETWEEN val1 AND val2`** — range check, both ends INCLUSIVE. val1 must be ≤ val2. Use `NOT BETWEEN` for outside the range.
- **`LIKE 'pattern'`** — pattern matching. `%` = zero or more characters. `_` = exactly one character. Case-insensitive by default.
  - `'abc%'` starts with abc. `'%abc'` ends with abc. `'%abc%'` contains abc. `'a%o'` starts with a AND ends with o. `'a_%'` starts with a, at least 2 chars.
- **`IN (val1, val2, val3)`** — matches any value in the list. Cleaner than multiple OR conditions. Use `NOT IN` to exclude.
- **`IS NULL`** — checks for NULL. **`IS NOT NULL`** — checks value exists. NEVER use `= NULL` — it always returns FALSE.
- **LIKE with `%` at the start** (`'%word'` or `'%word%'`) cannot use a regular index — full table scan. For large tables, use FULLTEXT indexes.
- **NOT IN with NULL** — if the list contains NULL, NOT IN returns zero rows. Always ensure clean lists.
- **In PHP** — always use prepared statements with `:param` or `?` placeholders. For dynamic IN lists, build `?,?,?` placeholders from an array and pass the array to `execute()`.