# Types of Databases & Popular DBMS Software

Not all databases work the same way. Different types of databases were invented to solve different problems — one type is perfect for banking, another for social networks, another for real-time caching. Understanding each type makes you a better architect who picks the right tool for the job.

---

## Table of Contents

1. [Why So Many Types?](#why-so-many-types)
2. [Relational Databases (RDBMS)](#relational-databases-rdbms)
3. [NoSQL Databases](#nosql-databases)
   - [Document Databases](#document-databases)
   - [Key-Value Databases](#key-value-databases)
   - [Wide-Column Databases](#wide-column-databases)
   - [Graph Databases](#graph-databases)
4. [Columnar Databases](#columnar-databases)
5. [Object-Oriented Databases](#object-oriented-databases)
6. [Hierarchical Databases](#hierarchical-databases)
7. [Cloud Databases](#cloud-databases)
8. [NewSQL Databases](#newsql-databases)
9. [Time-Series Databases](#time-series-databases)
10. [Search Engines as Databases](#search-engines-as-databases)
11. [All Types at a Glance](#all-types-at-a-glance)
12. [Popular DBMS Software — Deep Dive](#popular-dbms-software--deep-dive)
    - [MySQL](#mysql)
    - [PostgreSQL](#postgresql)
    - [SQLite](#sqlite)
    - [MariaDB](#mariadb)
    - [MongoDB](#mongodb)
    - [Redis](#redis)
    - [Cassandra](#cassandra)
    - [Neo4j](#neo4j)
    - [Elasticsearch](#elasticsearch)
    - [ClickHouse](#clickhouse)
    - [Firebase Firestore](#firebase-firestore)
    - [Amazon DynamoDB](#amazon-dynamodb)
13. [How to Choose the Right Database](#how-to-choose-the-right-database)
14. [Quick Revision](#quick-revision)

---

## Why So Many Types?

- In the early days (1970s–1990s), **relational databases** were used for everything — and they worked well.
- But as the internet grew, new problems appeared:
  - Websites needed to handle **millions of users simultaneously** — relational DBs struggled to scale.
  - Data became **less structured** — social media posts, sensor readings, user activity logs don't fit neatly into rows and columns.
  - Some problems needed completely different structures — a social network's "friend of a friend" relationship is very different from a bank's account table.
- So engineers built **specialized databases** — each optimized for one type of problem — sacrificing generality for performance.

```
One type of database CANNOT be best at everything.

Best analogy:
  A hammer is great for nails.
  A screwdriver is great for screws.
  You wouldn't use a hammer for screws, even though they're both tools.

Same with databases:
  MySQL is great for user accounts and orders.
  Redis is great for caching and sessions.
  Elasticsearch is great for full-text search.
  Neo4j is great for social networks and fraud detection.
```

---

## Relational Databases (RDBMS)

- The **oldest, most common, and most important** type of database for backend developers.
- Data is organized into **tables** (rows and columns), and tables are **related** to each other through keys.
- You interact with them using **SQL** (Structured Query Language).
- The foundation of virtually every traditional web application.

### How Data is Stored

```
customers table:
┌────┬──────────┬───────────────────────┬─────────────┐
│ id │ name     │ email                 │ city        │
├────┼──────────┼───────────────────────┼─────────────┤
│  1 │ Phyo     │ phyo@example.com      │ Yangon      │
│  2 │ Alice    │ alice@example.com     │ Mandalay    │
│  3 │ Bob      │ bob@example.com       │ Yangon      │
└────┴──────────┴───────────────────────┴─────────────┘

orders table:
┌────┬─────────────┬────────┬───────────┐
│ id │ customer_id │ total  │ status    │
├────┼─────────────┼────────┼───────────┤
│  1 │     1       │  99.99 │ shipped   │
│  2 │     1       │  45.00 │ pending   │
│  3 │     2       │ 150.00 │ delivered │
└────┴─────────────┴────────┴───────────┘

customer_id in orders REFERENCES id in customers.
Phyo (id=1) has 2 orders. Alice (id=2) has 1 order.
```

### Core Characteristics

| Feature | Description |
|---|---|
| **Schema** | Fixed — defined upfront, every row follows the same structure |
| **Relationships** | Enforced via foreign keys and JOINs |
| **ACID** | Full support — safe for financial data |
| **Query language** | SQL — standardized across all RDBMS |
| **Scaling** | Primarily vertical (more RAM/CPU on one machine) |
| **Consistency** | Very strong — no partial or inconsistent data |

### Real Strengths of Relational Databases

```sql
-- Ask complex questions across multiple tables in one query
SELECT
    c.name,
    COUNT(o.id)  AS total_orders,
    SUM(o.total) AS lifetime_value
FROM customers c
JOIN orders o ON c.id = o.customer_id
WHERE c.city = 'Yangon'
GROUP BY c.id
ORDER BY lifetime_value DESC;

-- Result: Yangon customers ranked by how much they've spent
-- This is incredibly powerful and natural in a relational DB
```

### When to Use

- ✅ User management (accounts, profiles, permissions)
- ✅ Financial data (transactions, accounts, invoices)
- ✅ E-commerce (products, orders, inventory)
- ✅ Content management (articles, comments, categories)
- ✅ Any app where relationships between data matter
- ✅ When data accuracy and consistency are critical

### Popular RDBMS: MySQL, PostgreSQL, SQLite, MariaDB, Oracle, SQL Server

---

## NoSQL Databases

- **NoSQL** stands for "Not Only SQL" — a family of databases that break away from the rigid table/row/column structure.
- They were designed for **scale, speed, and flexibility** — not to replace relational DBs, but to solve problems they struggle with.
- There is no single "NoSQL" — it's an umbrella term covering several very different types.

---

### Document Databases

- Stores data as **documents** — usually JSON or BSON format.
- Each document is a self-contained unit — it can have nested objects, arrays, and different fields from other documents.
- No fixed schema — documents in the same collection don't need the same structure.
- Think of a document as a "row that can hold anything, including other rows."

```json
// Product document — rich, nested structure
{
  "_id": "prod_abc123",
  "name": "iPhone 15 Pro",
  "brand": "Apple",
  "price": 999.99,
  "specs": {
    "storage": "256GB",
    "camera": "48MP",
    "chip": "A17 Pro"
  },
  "tags": ["smartphone", "apple", "premium"],
  "reviews": [
    { "user": "Phyo", "rating": 5, "comment": "Amazing phone" },
    { "user": "Alice", "rating": 4, "comment": "Great but expensive" }
  ],
  "available_colors": ["Black", "White", "Gold"],
  "in_stock": true
}
```

```json
// A different product — completely different structure in the SAME collection
{
  "_id": "prod_xyz789",
  "name": "PHP Handbook",
  "author": "John Doe",
  "isbn": "978-3-16-148410-0",
  "pages": 450,
  "publisher": "Tech Press"
  // No specs, no reviews, no colors — totally fine in NoSQL!
}
```

### Document Database Strengths

| Situation | Why Document DB Wins |
|---|---|
| Data structure varies per record | No migration needed when you add new fields |
| Nested/hierarchical data | Store naturally — no complex JOINs needed |
| Rapid prototyping | Change the "schema" instantly |
| Content management | Blog posts, products, user profiles |

> ⚠️ **Weakness:** When you need relationships between documents (orders belonging to users), you have to manage them yourself in application code — there's no built-in JOIN.

**Popular:** MongoDB, CouchDB, Amazon DocumentDB, Firestore

---

### Key-Value Databases

- The **simplest possible database** — stores data as a dictionary of `key → value` pairs.
- You look up data by its exact key — no searching, no filtering, no complex queries.
- **Extremely fast** because of its simplicity — reads and writes happen in microseconds.
- Think of it like a massive, super-fast PHP `array` or `hashmap` that persists to disk and is shared across servers.

```
key                     value
──────────────────────────────────────────────────────────
"user:1001"          →  '{"name":"Phyo","role":"admin"}'
"session:abc123"     →  '{"user_id":1001,"expires":1720000000}'
"cache:products:p1"  →  '[{"id":1,"name":"Shirt",...}]'
"rate_limit:1.2.3.4" →  "47"
"lock:order:99"      →  "1"
"counter:views:home" →  "128492"
```

```php
<?php
// Using Redis (the most popular key-value store)
$redis = new Redis();
$redis->connect("127.0.0.1", 6379);

// Store a value with 1-hour expiry
$redis->setex("user:1001", 3600, json_encode($user));

// Get it back — INSTANT (from RAM)
$user = json_decode($redis->get("user:1001"), true);

// Increment a counter
$redis->incr("counter:views:home");  // Atomic — thread safe
?>
```

### Key-Value Strengths

| Strength | Example |
|---|---|
| Blazing speed (microseconds) | Caching DB query results in Redis |
| Session storage | Store `$_SESSION` data shared across servers |
| Rate limiting | Count requests per IP per minute |
| Pub/Sub messaging | Real-time notifications |
| Atomic counters | View counts, like counts |
| Leaderboards | Sorted sets in Redis for game scores |

> ⚠️ **Weakness:** You can ONLY look up by exact key. There's no "find all users in Yangon" — you'd have to know the exact key. Complex queries are impossible without additional data structures.

**Popular:** Redis, Memcached, Amazon DynamoDB, Riak

---

### Wide-Column Databases

- Stores data in **rows and columns** — but unlike relational DBs, different rows can have **completely different columns**.
- Designed for **massive scale** — built to run across thousands of servers simultaneously.
- Columns are grouped into **column families** — groups of related columns stored together on disk.
- No ACID transactions in the traditional sense — trades consistency for availability and partition tolerance.

```
Traditional Relational Table (same columns for every row):
┌──────┬──────────┬────────┬─────────┐
│  id  │ name     │ email  │ phone   │
├──────┼──────────┼────────┼─────────┤
│  1   │ Phyo     │ ...    │ NULL    │ ← has to store NULL for phone
│  2   │ Alice    │ ...    │ ...     │
└──────┴──────────┴────────┴─────────┘

Wide-Column (columns are per-row — no NULLs needed):
Row Key: "user:1"
  → name: "Phyo"
  → email: "phyo@..."
  (no phone column at all — it just doesn't exist for this row)

Row Key: "user:2"
  → name: "Alice"
  → email: "alice@..."
  → phone: "+1-555-9999"
  → twitter: "@alice"   ← extra column only Alice has
```

### Wide-Column Strengths

| Strength | Example |
|---|---|
| Petabyte-scale storage | Netflix stores viewing history for 250M users |
| High write throughput | IoT sensors writing millions of data points/second |
| Distributed globally | Data automatically spread across 10+ data centers |
| Time-series data | Activity logs, metric data |

**Popular:** Apache Cassandra, HBase, Google Bigtable, Amazon Keyspaces

> 💡 **Real-world users:** Netflix (Cassandra), Instagram (Cassandra), Apple (Cassandra — stores over 75 petabytes), Uber, Discord.

---

### Graph Databases

- Stores data as **nodes** (entities) and **edges** (relationships between entities).
- Purpose-built for data where **relationships themselves are as important as the data**.
- Querying relationships in a graph DB is extremely fast — while the same query in a relational DB requires expensive self-joins.

```
Relational DB — "friends of Phyo's friends who also like PHP":
SELECT DISTINCT u3.name
FROM users u1
JOIN friendships f1 ON u1.id = f1.user_id
JOIN users u2 ON f1.friend_id = u2.id
JOIN friendships f2 ON u2.id = f2.user_id
JOIN users u3 ON f2.friend_id = u3.id
JOIN interests i ON u3.id = i.user_id
WHERE u1.name = 'Phyo'
AND i.topic = 'PHP'
AND u3.id != u1.id;
-- Complex SQL, slow on large data

Graph DB (Neo4j) — same query:
MATCH (phyo:Person {name: "Phyo"})-[:FRIENDS_WITH*2]->(person:Person)
WHERE (person)-[:LIKES]->(:Topic {name: "PHP"})
RETURN DISTINCT person.name
-- Simple, expressive, and FAST
```

```
Visual structure of a graph database:

        [Phyo] ──FRIENDS_WITH──► [Alice] ──FRIENDS_WITH──► [Bob]
           │                        │                         │
        LIKES                    WORKS_AT                  LIKES
           │                        │                         │
         [PHP]                   [Google]                  [PHP]

Nodes:  Phyo, Alice, Bob, PHP, Google
Edges:  FRIENDS_WITH, LIKES, WORKS_AT
Properties on nodes: {name: "Phyo", age: 25}
Properties on edges: {since: "2020-01-01"}
```

### Graph Database Strengths

| Strength | Use Case |
|---|---|
| Relationship traversal | Social networks ("people you may know") |
| Path finding | GPS navigation, supply chain |
| Fraud detection | Detect suspicious connection patterns |
| Recommendation engines | "Customers who bought X also bought Y" |
| Knowledge graphs | AI and semantic search |

**Popular:** Neo4j, Amazon Neptune, ArangoDB, JanusGraph

---

## Columnar Databases

- Traditional row-oriented DBs store **entire rows together** on disk.
- Columnar databases store **entire columns together** on disk.
- This sounds like a small difference but has massive performance implications for analytics.

```
ROW-ORIENTED storage (traditional — MySQL, PostgreSQL):
Physical disk layout:
  [1, "Phyo", 99.99, "Yangon"] [2, "Alice", 149.99, "Mandalay"] [3, "Bob", 25.00, "Yangon"]
  ← Each row stored as one unit

  Query: "SELECT SUM(total) FROM orders"
  → Must read ALL data for ALL rows, even columns you don't need (id, name, city)
  → Very slow for analytics on wide tables

COLUMN-ORIENTED storage (ClickHouse, Redshift):
Physical disk layout:
  id column:    [1] [2] [3]
  name column:  ["Phyo"] ["Alice"] ["Bob"]
  total column: [99.99] [149.99] [25.00]
  city column:  ["Yangon"] ["Mandalay"] ["Yangon"]

  Query: "SELECT SUM(total) FROM orders"
  → Reads ONLY the total column — skips id, name, city entirely
  → Columns also compress extremely well (similar values together)
  → 10x–100x faster for analytical queries!
```

### Row-Oriented vs Column-Oriented

| Feature | Row-Oriented (MySQL) | Column-Oriented (ClickHouse) |
|---|---|---|
| Insert a new row | ✅ Fast (write all columns at once) | ❌ Slower (update all column files) |
| Read a full row | ✅ Fast (one disk read) | ❌ Slower (read from many column files) |
| Aggregate a column | ❌ Slow (reads all columns) | ✅ Very fast (reads one column) |
| Analytics/reporting | ❌ Poor | ✅ Excellent |
| OLTP (transactions) | ✅ Excellent | ❌ Not designed for it |
| OLAP (analytics) | ❌ Poor | ✅ Excellent |

> 💡 **OLTP vs OLAP:**
> - **OLTP** (Online Transaction Processing) — lots of small, fast reads/writes. "Add this order", "Update this user." → Use MySQL/PostgreSQL.
> - **OLAP** (Online Analytical Processing) — large, complex queries over millions of rows for reporting. "What was our revenue by product category last quarter?" → Use columnar databases.

**Popular:** ClickHouse, Amazon Redshift, Google BigQuery, Apache Parquet, Snowflake, Apache Druid

---

## Object-Oriented Databases

- Stores data as **objects** — exactly like objects in object-oriented programming languages.
- Objects have attributes (data) AND methods (behavior) stored together.
- Designed to eliminate the **object-relational impedance mismatch** — the friction that occurs when mapping OOP objects to relational tables.

```
In OOP you have:
class Product {
    public int    $id;
    public string $name;
    public array  $images;       ← array of Image objects
    public Category $category;   ← reference to another object
}

In a relational DB, this becomes THREE tables:
  products, images, categories — with complex JOINs

In an Object-Oriented DB:
  $product is stored EXACTLY as it is in your code
  No mapping, no JOINs, no translation
```

> 💡 **In practice**, object-oriented databases never became mainstream. Relational databases with ORMs (Object-Relational Mappers like Eloquent in Laravel) solve the mapping problem "good enough" for most applications. You'll likely never use a pure object-oriented database in the real world — but understanding why they were invented helps you understand ORMs.

**Examples:** db4o (now abandoned), ObjectDB, Caché (InterSystems)

---

## Hierarchical Databases

- The **oldest type** of database — invented in the 1960s by IBM (called IMS).
- Data is stored in a **tree structure** — parent nodes and child nodes, like a family tree or a file system.
- Every record has exactly one parent (except the root).
- Navigation happens by following parent-child pointers — not by querying.

```
Hierarchical structure:
            [Company]
           /          \
    [Department A]  [Department B]
    /      \              |
[Team 1] [Team 2]    [Team 3]
    |         |
[Emp 1]   [Emp 2]

Each employee belongs to exactly one team.
Each team belongs to exactly one department.
← STRICT parent-child, one parent only
```

```
Real example still in use: XML / JSON documents ARE hierarchical
<company>
  <department name="Engineering">
    <team name="Backend">
      <employee name="Phyo" />
      <employee name="Alice" />
    </team>
  </department>
</company>
```

> ⚠️ **Weakness:** If an employee works on TWO teams, you can't represent that — hierarchical databases only allow one parent. This limitation is why relational databases replaced them.

> 💡 **Still relevant today:** File systems (folders and files), XML/JSON/HTML structures, DNS (domain names), organizational charts, and directory services like **LDAP** (used for corporate user authentication) are all hierarchical structures. You encounter hierarchical data constantly — it's just no longer stored in hierarchical DBMS software.

**Examples:** IBM IMS (still used in banking mainframes), Windows Registry, LDAP/Active Directory

---

## Cloud Databases

- A **cloud database** is a database that runs in the cloud — hosted and managed by a cloud provider rather than on a server you own and maintain yourself.
- You don't install software, configure servers, apply patches, or manage backups — the cloud provider handles all of that.
- You pay for what you use (pay-per-query, pay-per-storage, pay-per-hour).

### Types of Cloud Database Services

#### DBaaS (Database as a Service)

- A fully managed version of a traditional database — same SQL, same features, zero infrastructure management.

```
Traditional (Self-Managed):
  → Buy a server
  → Install MySQL
  → Configure replication
  → Set up backups
  → Monitor disk space
  → Apply security patches
  → Handle crashes

Cloud DBaaS (e.g., Amazon RDS):
  → Click "Create database"
  → Choose MySQL 8.0, size, region
  → Connect from your app
  → Done. AWS handles everything else.
```

#### Serverless Databases

- Auto-scale to zero when not in use — you pay only when queries run.
- Excellent for development, low-traffic apps, and unpredictable workloads.

#### Popular Cloud Database Services

| Service | Provider | Type | Best For |
|---|---|---|---|
| **Amazon RDS** | AWS | MySQL/PostgreSQL/MariaDB/Oracle | Managed relational DB |
| **Amazon Aurora** | AWS | MySQL/PostgreSQL compatible | High-performance managed |
| **Amazon DynamoDB** | AWS | Key-value / Document | Serverless, massive scale |
| **Amazon Redshift** | AWS | Columnar | Data warehousing |
| **Google Cloud SQL** | GCP | MySQL/PostgreSQL | Managed relational |
| **Google BigQuery** | GCP | Columnar/Analytics | Data warehousing, analytics |
| **Google Firestore** | GCP | Document | Mobile/web apps, Firebase |
| **Azure SQL Database** | Azure | SQL Server | Enterprise Microsoft stack |
| **Azure Cosmos DB** | Azure | Multi-model (any type) | Global-scale any model |
| **PlanetScale** | Independent | MySQL compatible | Serverless MySQL |
| **Supabase** | Independent | PostgreSQL | Firebase alternative, open-source |
| **Neon** | Independent | PostgreSQL | Serverless PostgreSQL |
| **MongoDB Atlas** | MongoDB | Document | Managed MongoDB |
| **Redis Cloud** | Redis | Key-value | Managed Redis |

### Self-Managed vs Cloud Database

| Feature | Self-Managed | Cloud (DBaaS) |
|---|---|---|
| Setup | You install & configure | Click and go |
| Maintenance | You patch, update, monitor | Provider handles it |
| Backups | You set it up | Automatic, built-in |
| Scaling | You add hardware | Click to scale |
| Cost | Server cost (fixed) | Pay per use (variable) |
| Control | Full | Limited (can't SSH to the DB server) |
| Security | Your responsibility | Shared with provider |
| Best for | Full control, cost efficiency at scale | Small teams, startups, rapid dev |

---

## NewSQL Databases

- **NewSQL** tries to combine the best of both worlds — the scalability of NoSQL with the ACID guarantees and SQL of relational databases.
- Built to scale horizontally (across many machines) while still supporting full SQL and transactions.

```
Old problem:
  Need scale → Use NoSQL → Lose ACID and SQL
  Need ACID  → Use SQL   → Can't scale

NewSQL solution:
  Get SQL + ACID + horizontal scale all at once
```

**Examples:** CockroachDB, Google Spanner, TiDB, YugabyteDB, VoltDB

> 💡 **When relevant:** You'll see NewSQL databases in large distributed systems — global applications that need to be deployed across multiple geographic regions while maintaining strong consistency. For most projects, PostgreSQL or MySQL is sufficient.

---

## Time-Series Databases

- Optimized for storing data that **changes over time** — measurements at regular intervals.
- Every record has a **timestamp** as its primary dimension.
- Extremely efficient at storing and querying time-based data that would be slow in a traditional DB.

```
Sensor readings stored every second:
  2026-06-28 14:30:00 | CPU: 45%  | RAM: 2.1GB | Disk: 120MB/s
  2026-06-28 14:30:01 | CPU: 47%  | RAM: 2.1GB | Disk: 118MB/s
  2026-06-28 14:30:02 | CPU: 52%  | RAM: 2.2GB | Disk: 125MB/s
  ...
  (86,400 rows per day per server — millions per day across a fleet)

Queries: "What was average CPU usage between 2pm and 3pm?"
         "Show me traffic spikes in the last 7 days"
         "Alert me when temperature exceeds 80°C"
```

**Popular:** InfluxDB, TimescaleDB (PostgreSQL extension), Prometheus, Victoria Metrics

> 💡 **When relevant:** Monitoring dashboards (Grafana + Prometheus/InfluxDB), IoT device data, stock price history, application performance monitoring (APM), server metrics.

---

## Search Engines as Databases

- Technically a specialized type of document store, but optimized specifically for **full-text search** — finding documents that match words or phrases.
- Traditional databases are terrible at "find me all products that match the word 'wireless' even if spelled differently or as a related term."
- Search engines use inverted indexes — a mapping from every word to all documents containing that word.

```
User searches: "wireless headphones noise cancelling"

MySQL LIKE query (bad):
SELECT * FROM products
WHERE name LIKE '%wireless%'
AND name LIKE '%noise%'
→ Scans every row, only exact substring matches, no ranking

Elasticsearch (good):
{
  "query": {
    "multi_match": {
      "query": "wireless headphones noise cancelling",
      "fields": ["name", "description", "tags"]
    }
  }
}
→ Instant results, relevance-ranked, handles typos, synonyms,
  searches across multiple fields, highlights matching text
```

**Popular:** Elasticsearch, Apache Solr, Meilisearch, Typesense, Algolia (cloud)

---

## All Types at a Glance

| Type | How Data is Stored | Best For | Examples |
|---|---|---|---|
| **Relational (SQL)** | Tables + rows + columns | Most web apps, finance, e-commerce | MySQL, PostgreSQL, SQLite |
| **Document** | JSON/BSON documents | Flexible schemas, content, catalogs | MongoDB, Firestore, CouchDB |
| **Key-Value** | key → value pairs | Caching, sessions, counters | Redis, Memcached, DynamoDB |
| **Wide-Column** | Rows with variable columns, column families | Massive scale, IoT, time-series | Cassandra, HBase, Bigtable |
| **Graph** | Nodes + edges + properties | Social networks, fraud, recommendations | Neo4j, Amazon Neptune |
| **Columnar** | Columns stored together | Analytics, reporting, data warehousing | ClickHouse, BigQuery, Redshift |
| **Object-Oriented** | Objects with methods | OOP-native persistence (rare) | db4o, ObjectDB |
| **Hierarchical** | Tree (parent-child) | Legacy systems, LDAP, file systems | IBM IMS, LDAP |
| **Cloud / DBaaS** | Any of the above, managed | Managed infrastructure | RDS, Aurora, Firestore, Atlas |
| **NewSQL** | Distributed SQL | Scale + ACID | CockroachDB, Spanner, TiDB |
| **Time-Series** | Timestamp + measurements | Metrics, monitoring, IoT | InfluxDB, Prometheus, Timescale |
| **Search Engine** | Inverted index | Full-text search, ranking | Elasticsearch, Meilisearch |

---

## Popular DBMS Software — Deep Dive

---

### MySQL

```
Type:           Relational (SQL)
Current Version: 8.0 / 8.4 (LTS)
Owner:          Oracle Corporation
License:        GPL (Community) + Commercial (Enterprise)
Default Port:   3306
Storage Format: InnoDB (default engine)

Architecture:
  Client (PHP) → MySQL Protocol → Connection Handler
  → Query Cache → Parser → Optimizer → InnoDB Storage Engine
  → .ibd files on disk (one per table)
```

**Strengths:**
- Most widely deployed database in the world — especially in PHP/web applications.
- Excellent read performance — handles millions of reads per second.
- Supported by every PHP framework (Laravel, Symfony, WordPress, etc.).
- Huge ecosystem — tools, libraries, tutorials, hosting support everywhere.
- Master-replica replication built-in for read scaling.

**Weaknesses:**
- Historically permissive — accepted invalid data silently (improving in MySQL 8+).
- Owned by Oracle — some community distrust.
- Less feature-rich than PostgreSQL (fewer data types, less advanced SQL).

**Real-world users:** WordPress (powers 43% of the web), Facebook (historically), Twitter (historically), YouTube, Airbnb, Uber.

```bash
# Install on Ubuntu
sudo apt install mysql-server

# Connect
mysql -u root -p

# Basic operations
CREATE DATABASE myapp;
USE myapp;
CREATE TABLE users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(100));
INSERT INTO users (name) VALUES ("Phyo");
SELECT * FROM users;
```

---

### PostgreSQL

```
Type:           Object-Relational (SQL)
Current Version: 16 / 17
License:        PostgreSQL License (completely free, forever)
Default Port:   5432
Storage:        MVCC (Multi-Version Concurrency Control) — no write locks for reads

Architecture:
  Each connection → dedicated backend process (not thread)
  WAL (Write-Ahead Log) → ensures durability
  VACUUM process → reclaims storage from deleted rows
```

**Strengths:**
- The most feature-complete open-source database in existence.
- True SQL standard compliance — no silent type coercions or surprises.
- Advanced data types: JSON/JSONB, arrays, ranges, geometric types, UUID, enum.
- Built-in full-text search (surprisingly capable).
- Excellent for complex analytics — window functions, CTEs, lateral joins.
- Extensions: PostGIS (geographic data), pgvector (AI embeddings), TimescaleDB (time-series).
- 100% community-driven — no corporate control.

**Weaknesses:**
- Historically slower than MySQL for simple read-heavy workloads (gap has narrowed significantly).
- More complex configuration for beginners.
- Memory usage can be higher.

**Real-world users:** Apple, Instagram (handled scaling from 0 to 1 billion users), Reddit, Spotify, GitLab, GitHub, Shopify, Heroku.

```bash
# Install on Ubuntu
sudo apt install postgresql

# Connect
sudo -u postgres psql

# Basic operations
CREATE DATABASE myapp;
\c myapp
CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(100));
INSERT INTO users (name) VALUES ('Phyo');
SELECT * FROM users;

# PostgreSQL-specific advanced features
CREATE TABLE products (
    id      SERIAL PRIMARY KEY,
    name    VARCHAR(255),
    tags    TEXT[],                    -- Array of strings!
    specs   JSONB,                     -- Binary JSON with indexing!
    price   NUMERIC(10,2)
);

INSERT INTO products (name, tags, specs, price) VALUES
('Laptop', ARRAY['electronics', 'portable'], '{"ram":"16GB","cpu":"M3"}', 999.99);

-- Query JSON directly
SELECT name, specs->>'ram' AS ram FROM products WHERE specs->>'cpu' = 'M3';

-- Query arrays
SELECT * FROM products WHERE 'electronics' = ANY(tags);
```

---

### SQLite

```
Type:           Embedded Relational (SQL)
License:        Public Domain (completely free, no restrictions whatsoever)
Storage:        Single .db file
Port:           None — no server, runs inside your application process

Architecture: (completely different from MySQL/PostgreSQL)
  Your App Process
      │
      └── SQLite library (embedded directly into your app)
              │
              └── single .db file on disk
  No server, no network, no authentication
```

**Strengths:**
- Zero setup — the database is just a file.
- Unbelievably widely deployed — in every Android device, iPhone, macOS, Windows 10+, Firefox, Chrome.
- Perfect for testing, local development, prototyping.
- No concurrent writers needed → single-user or low-concurrency apps.
- Supports most SQL standard features.
- PHP supports SQLite natively — no extensions needed.

**Weaknesses:**
- Single writer at a time — can't handle multiple simultaneous writes (web apps with concurrent users).
- No user authentication — anyone who can access the file can read it.
- Limited data types compared to PostgreSQL.
- Not suitable for production web apps with multiple users.

```php
<?php
// PHP + SQLite — no server needed, just a file
$pdo = new PDO("sqlite:/var/www/myapp/database.db");

$pdo->exec("CREATE TABLE IF NOT EXISTS users (
    id   INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT UNIQUE
)");

$stmt = $pdo->prepare("INSERT INTO users (name, email) VALUES (?, ?)");
$stmt->execute(["Phyo", "phyo@example.com"]);

$users = $pdo->query("SELECT * FROM users")->fetchAll(PDO::FETCH_ASSOC);
?>
```

---

### MariaDB

```
Type:           Relational (SQL) — MySQL-compatible fork
Founder:        Michael "Monty" Widenius (original MySQL creator)
License:        GPL (fully open source — no Oracle involvement)
Port:           3306 (same as MySQL — drop-in replacement)
Version:        10.x, 11.x
```

**Why it exists:** When Oracle acquired MySQL in 2010 via the Sun Microsystems purchase, MySQL's original creator was worried Oracle would commercialize or kill it. He forked MySQL and created MariaDB as a guaranteed-open-source alternative.

**Strengths:**
- Near 100% MySQL-compatible — switch from MySQL to MariaDB with virtually zero code changes.
- Fully open source — no commercial version, no features locked behind a paywall.
- Often slightly faster than MySQL for specific workloads.
- Additional storage engines (Aria, ColumnStore).
- Better community responsiveness to feature requests.

**Used by:** Wikipedia (switched from MySQL), Google, Red Hat, Arch Linux.

---

### MongoDB

```
Type:           Document Database (NoSQL)
License:        SSPL (Server Side Public License) — controversial
Current Version: 7.0
Port:           27017
Data Format:    BSON (Binary JSON)
Query Language: MQL (MongoDB Query Language) — not SQL
```

**Core concepts:**

```
SQL Term     → MongoDB Term
──────────────────────────────
Database     → Database
Table        → Collection
Row/Record   → Document
Column       → Field
Primary Key  → _id (auto-generated ObjectID)
JOIN         → $lookup (aggregation) or embedded docs
```

```javascript
// MongoDB queries look like JavaScript/JSON
// Find all users in Yangon
db.users.find({ city: "Yangon" })

// Find users with orders over 100
db.users.find({ "orders.total": { $gt: 100 } })

// Aggregate: total revenue per city
db.orders.aggregate([
  { $group: { _id: "$city", revenue: { $sum: "$total" } } },
  { $sort: { revenue: -1 } }
])
```

**Strengths:**
- Flexible schema — add new fields without migrations.
- Natural fit for JSON-heavy applications.
- Built-in horizontal scaling (sharding).
- Great for hierarchical data where embedding is natural.

**Weaknesses:**
- No multi-document ACID until version 4.0 (now supported but slower).
- SSPL license is controversial — some consider it non-open-source.
- JOINs ($lookup) are slow compared to relational JOINs.
- Easy to accidentally create inconsistent data.

**Used by:** eBay, Forbes, Cisco, Adobe, Foursquare.

---

### Redis

```
Type:           In-Memory Key-Value Store (+ more data structures)
License:        BSD (OSS Redis), RSAL (Redis 7.4+ some restrictions)
Port:           6379
Data location:  RAM (optionally persisted to disk)
Speed:          100,000+ operations per second, sub-millisecond latency
```

**Data structures Redis supports:**

```
String:  SET user:1 "Phyo"
List:    LPUSH notifications:1 "New order" "New message"
Set:     SADD online_users 1001 1002 1003
Hash:    HSET user:1 name "Phyo" email "phyo@example.com"
ZSet:    ZADD leaderboard 9500 "Phyo" 8200 "Alice"  ← sorted by score
Bitmap:  SETBIT daily_active:2026-06-28 1001 1  ← which users were active today
Stream:  XADD events * type "purchase" user_id 1001  ← real-time event log
```

```php
<?php
$redis = new Redis();
$redis->connect("127.0.0.1", 6379);

// Cache a database query result
$cacheKey = "products:category:shoes";
$cached   = $redis->get($cacheKey);

if (!$cached) {
    $products = $pdo->query("SELECT * FROM products WHERE category='shoes'")->fetchAll();
    $redis->setex($cacheKey, 3600, json_encode($products));  // Cache 1 hour
    echo "From database";
} else {
    $products = json_decode($cached, true);
    echo "From cache (instant!)";
}

// Rate limiting
$key     = "rate_limit:" . $_SERVER["REMOTE_ADDR"];
$current = $redis->incr($key);
if ($current === 1) $redis->expire($key, 60);  // Expire in 1 minute
if ($current > 100) die("Rate limit exceeded");

// Session storage
session_save_path("tcp://127.0.0.1:6379");
session_start();
?>
```

**Used by:** Twitter, GitHub, Snapchat, Stack Overflow, Pinterest.

---

### Cassandra

```
Type:           Wide-Column Database (NoSQL)
License:        Apache License 2.0 (completely open source)
Port:           9042 (CQL), 7000 (inter-node)
Created by:     Facebook (donated to Apache in 2008)
Architecture:   Masterless ring — every node is equal, no single point of failure
```

**What makes Cassandra special:**

```
Cassandra's architecture:
  Node 1 ←→ Node 2 ←→ Node 3
     ↑                    ↓
  Node 6              Node 4
     ↑                    ↓
  Node 5 ←→ ← ← ← ←Node 5

Every node is identical — no master/slave.
Data is automatically replicated to multiple nodes.
If 3 nodes die → system still works with remaining nodes.
Linear scaling: double the nodes → double the throughput.
```

**Cassandra Query Language (CQL) — looks like SQL:**

```sql
CREATE TABLE user_activity (
    user_id   UUID,
    timestamp TIMESTAMP,
    action    TEXT,
    page      TEXT,
    PRIMARY KEY (user_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);

INSERT INTO user_activity (user_id, timestamp, action, page)
VALUES (uuid(), toTimestamp(now()), 'click', '/products');

SELECT * FROM user_activity
WHERE user_id = some-uuid
AND timestamp > '2026-06-01'
LIMIT 100;
```

**Used by:** Netflix (all viewing history), Apple (10+ petabytes), Instagram, Spotify, Uber, Discord.

---

### Neo4j

```
Type:           Graph Database
License:        Community (open source) + Enterprise (commercial)
Port:           7687 (Bolt protocol), 7474 (HTTP browser)
Query Language: Cypher (visual, pattern-matching language)
Storage:        Native graph storage — nodes and edges stored with direct pointers
```

**Cypher query language — reads like a picture:**

```cypher
// Create nodes and relationships
CREATE (phyo:Person {name: "Phyo", age: 25})
CREATE (alice:Person {name: "Alice", age: 30})
CREATE (php:Technology {name: "PHP"})
CREATE (phyo)-[:FRIENDS_WITH {since: "2020"}]->(alice)
CREATE (phyo)-[:KNOWS]->(php)
CREATE (alice)-[:KNOWS]->(php)

// Find friends of friends who also know PHP
MATCH (phyo:Person {name: "Phyo"})-[:FRIENDS_WITH*1..2]->(friend:Person)
WHERE (friend)-[:KNOWS]->(:Technology {name: "PHP"})
AND friend.name <> "Phyo"
RETURN friend.name, friend.age

// Find shortest path between two people
MATCH path = shortestPath(
  (phyo:Person {name: "Phyo"})-[*]-(target:Person {name: "Bob"})
)
RETURN path
```

**Strengths:** Relationship-heavy queries that would require 10+ JOINs in SQL run in milliseconds.

**Used by:** eBay (product knowledge graph), Walmart (supply chain), NASA, Airbus, LinkedIn (relationship graph).

---

### Elasticsearch

```
Type:           Distributed Search Engine + Document Store
License:        Elastic License 2.0 (partially proprietary since 7.11)
Port:           9200 (HTTP REST API), 9300 (transport)
Based on:       Apache Lucene (full-text search library)
Data model:     JSON documents with inverted indexes
```

**How Elasticsearch finds text so fast — Inverted Index:**

```
Documents stored:
  Doc 1: "PHP is a powerful language for web development"
  Doc 2: "MySQL is a popular database for web applications"
  Doc 3: "PHP and MySQL work great together for web apps"

Inverted Index built automatically:
  "php"         → [Doc 1, Doc 3]
  "mysql"       → [Doc 2, Doc 3]
  "web"         → [Doc 1, Doc 2, Doc 3]
  "language"    → [Doc 1]
  "database"    → [Doc 2]
  "powerful"    → [Doc 1]

Search: "PHP web"
→ Find docs containing "php": [1, 3]
→ Find docs containing "web": [1, 2, 3]
→ Intersect + rank by relevance: Doc 1 (both), Doc 3 (both), Doc 2 (web only)
→ Instant results with relevance score!
```

**Used by:** Wikipedia (article search), GitHub (code search), Netflix, Airbnb, eBay, Stack Overflow.

---

### ClickHouse

```
Type:           Columnar OLAP Database
License:        Apache License 2.0
Port:           8123 (HTTP), 9000 (native)
Created by:     Yandex (Russian search engine) — now independent company
Speed:          Can process BILLIONS of rows per second
```

**What makes ClickHouse extraordinary:**

```sql
-- Count 1 billion rows with GROUP BY — takes ~1 second in ClickHouse!
SELECT
    toStartOfMonth(event_time) AS month,
    count()                     AS events,
    uniq(user_id)              AS unique_users
FROM page_views
WHERE event_time BETWEEN '2026-01-01' AND '2026-12-31'
GROUP BY month
ORDER BY month;

-- Same query in MySQL on 1 billion rows: would take HOURS
-- ClickHouse on 1 billion rows: 0.8 seconds
```

**Used by:** Cloudflare (processes 6 million events/second), Uber, eBay, Spotify, Yandex.

---

### Firebase Firestore

```
Type:           Document Database (NoSQL) — Cloud-native
Provider:       Google (Firebase platform)
License:        Proprietary (Google service)
SDK:            JavaScript, Swift, Kotlin, Flutter, PHP (unofficial)
Real-time:      Yes — clients receive updates automatically via WebSocket
```

**What makes Firestore unique:**

```javascript
// Real-time listener — auto-updates the UI when data changes
db.collection("messages")
  .where("room", "==", "general")
  .orderBy("timestamp", "desc")
  .limit(50)
  .onSnapshot((snapshot) => {
    snapshot.docChanges().forEach((change) => {
      if (change.type === "added") {
        displayMessage(change.doc.data());  // Automatically called when new messages arrive!
      }
    });
  });
// No polling needed — updates arrive in real time
```

**Used by:** Mobile apps (iOS/Android), real-time chat apps, collaborative tools, startups that want zero infrastructure.

---

### Amazon DynamoDB

```
Type:           Key-Value + Document (NoSQL) — Serverless
Provider:       Amazon Web Services
License:        Proprietary (AWS service)
Scaling:        Automatic — scales to any traffic level instantly
Pricing:        Per read/write unit consumed
```

**DynamoDB's superpower:**

```
Traditional database: you pick a size upfront
  "I'll take 4 CPUs, 16GB RAM" — pay whether you use it or not
  Traffic spike? → Database crashes → Users see errors

DynamoDB: serverless, auto-scaling
  0 requests/sec  → pay almost nothing
  1M requests/sec → handles it automatically, pay for what you use
  Zero capacity planning needed
```

**Used by:** Amazon itself, Lyft, Samsung, Toyota, Netflix, Capital One.

---

## How to Choose the Right Database

```
Decision flowchart for backend developers:

START
  ↓
Is your data relational? (users → orders → products)
  YES → Use PostgreSQL or MySQL (start here for most apps)
  NO  ↓
      ↓
Is it simple key→value with speed as top priority?
  YES → Use Redis (caching, sessions, rate limiting)
  NO  ↓
      ↓
Is it flexible/hierarchical JSON documents?
  YES → Use MongoDB or Firestore
  NO  ↓
      ↓
Is it relationship-heavy? (social graphs, fraud detection)
  YES → Use Neo4j
  NO  ↓
      ↓
Is it analytical? (billions of rows, reports, metrics)
  YES → Use ClickHouse, BigQuery, or Redshift
  NO  ↓
      ↓
Is it massive-scale time-series or IoT data?
  YES → Use Cassandra, InfluxDB, or TimescaleDB
  NO  ↓
      ↓
Is it full-text search?
  YES → Use Elasticsearch or Meilisearch
```

### The Real-World Combination (Polyglot Persistence)

Most serious backend systems use **multiple databases** — each doing what it does best:

```
E-Commerce Platform:
  MySQL/PostgreSQL  → Users, products, orders (the source of truth)
  Redis             → Session data, shopping carts, rate limiting, caching
  Elasticsearch     → Product search ("wireless headphones under $100")
  MongoDB           → Product reviews (flexible structure, vary per product)
  ClickHouse        → Analytics (sales reports, conversion funnels)

This is called "Polyglot Persistence" — using the right DB for each job.
```

---

## Quick Revision

- **Relational (SQL)** — tables + rows + columns + JOINs + ACID. Use for most web apps. MySQL and PostgreSQL are your primary tools.
- **Document** — JSON documents, flexible schema, no JOINs. Use for content, catalogs, apps with varying structure. MongoDB, Firestore.
- **Key-Value** — simple key → value, blazing fast (RAM). Use for caching, sessions, rate limiting. Redis is king here.
- **Wide-Column** — variable columns per row, masterless, petabyte scale. Use for IoT, time-series, global scale. Cassandra, HBase.
- **Graph** — nodes + edges, relationship-native queries. Use for social networks, fraud detection, recommendations. Neo4j.
- **Columnar** — columns stored together, 10-100x faster for analytics. Use for reporting, dashboards, data warehousing. ClickHouse, BigQuery, Redshift.
- **Object-Oriented** — stores objects directly. Mostly obsolete — ORMs solve this in relational DBs.
- **Hierarchical** — tree structure, one parent per node. Legacy systems and LDAP — still relevant in concept (file systems, XML, JSON are hierarchical).
- **Cloud databases** — same types but fully managed. No installation, auto-backup, auto-scale. AWS RDS, Firebase, Supabase, MongoDB Atlas, PlanetScale.
- **NewSQL** — distributed SQL with horizontal scale + ACID. CockroachDB, Spanner, TiDB.
- **Time-Series** — timestamp-centric, metric data. InfluxDB, Prometheus, TimescaleDB.
- **Search Engines** — inverted indexes for full-text search. Elasticsearch, Meilisearch, Typesense.
- **For your learning path:** Master **MySQL or PostgreSQL** first (relational foundation). Add **Redis** for caching. Learn **MongoDB** for NoSQL exposure. Everything else is specialization.
- **Polyglot persistence** — production systems use multiple databases together. Each database does what it's best at — relational for core data, Redis for speed, Elasticsearch for search, etc.
- **MySQL** = most popular in web/PHP. **PostgreSQL** = most feature-complete and correct. **SQLite** = zero setup, single file, for local/embedded. **MongoDB** = flexible JSON documents. **Redis** = in-memory speed. **Cassandra** = horizontal scale + no single point of failure. **Elasticsearch** = full-text search. **ClickHouse** = analytical queries on billions of rows.