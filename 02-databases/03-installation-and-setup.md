# Relational vs NoSQL & MySQL — Installation, Setup & First Steps

Understanding the difference between relational and NoSQL databases will help you make the right architectural decisions. Then we dive deep into MySQL — the most popular database for PHP development — covering installation, setup, and everything you need to know before writing a single query.

---

## Table of Contents

1. [Relational vs NoSQL — The Deep Dive](#relational-vs-nosql--the-deep-dive)
   - [How Relational Databases Think](#how-relational-databases-think)
   - [How NoSQL Databases Think](#how-nosql-databases-think)
   - [ACID vs BASE](#acid-vs-base)
   - [Scaling: Vertical vs Horizontal](#scaling-vertical-vs-horizontal)
   - [When to Use Which](#when-to-use-which)
   - [Full Comparison Table](#full-comparison-table)
2. [What is MySQL and Why Use It?](#what-is-mysql-and-why-use-it)
3. [MySQL Installation](#mysql-installation)
   - [Windows](#windows)
   - [macOS](#macos)
   - [Ubuntu / Debian](#ubuntu--debian)
4. [MySQL Service Management](#mysql-service-management)
5. [Securing MySQL After Installation](#securing-mysql-after-installation)
6. [Connecting to MySQL](#connecting-to-mysql)
7. [MySQL Server Architecture](#mysql-server-architecture)
8. [Databases and Users — The Admin Essentials](#databases-and-users--the-admin-essentials)
   - [Creating and Managing Databases](#creating-and-managing-databases)
   - [Creating and Managing Users](#creating-and-managing-users)
   - [Granting Permissions (GRANT)](#granting-permissions-grant)
   - [Revoking Permissions](#revoking-permissions)
9. [MySQL Configuration File (my.cnf)](#mysql-configuration-file-mycnf)
10. [Connecting MySQL to PHP](#connecting-mysql-to-php)
11. [MySQL GUI Tools](#mysql-gui-tools)
12. [Quick Revision](#quick-revision)

---

## Relational vs NoSQL — The Deep Dive

---

### How Relational Databases Think

- A relational database thinks of the world as **separate, related tables**.
- Every piece of information lives in exactly one place (normalization).
- Tables are linked through keys — relationships are declared upfront and enforced.
- The schema is a contract — decided before data goes in, every row follows it.

```
RELATIONAL MINDSET — Store each "thing" separately, link them together

users table:           orders table:           products table:
┌────┬──────────┐      ┌────┬─────────┬──────┐  ┌────┬──────────┬───────┐
│ id │ name     │      │ id │ user_id │ ...  │  │ id │ name     │ price │
├────┼──────────┤      ├────┼─────────┼──────┤  ├────┼──────────┼───────┤
│  1 │ Phyo     │◄─────│  1 │    1    │ ...  │  │  5 │ T-Shirt  │ 19.99 │
│  2 │ Alice    │      │  2 │    1    │ ...  │  └────┴──────────┴───────┘
└────┴──────────┘      └────┴─────────┴──────┘
                                  ↑
                        user_id=1 links to Phyo

Phyo's data is stored ONCE in users.
His orders reference him by user_id.
Change Phyo's name once → all queries see the update instantly.
```

**The data model is designed for accuracy.** Nothing is duplicated, everything is linked, and the database enforces the relationships.

---

### How NoSQL Databases Think

- A NoSQL database thinks of the world as **independent documents** (or key-value pairs, or graphs).
- Data that is queried together is **stored together** — even if that means repeating it.
- No schema enforcement — each document can have different fields.
- Relationships are either embedded (nested) or handled by the application, not the database.

```
NOSQL MINDSET (MongoDB) — Store everything you need for a query together

{
  "_id": "order_001",
  "user": {                        ← user data EMBEDDED inside the order
    "id": 1,
    "name": "Phyo",               ← name copied here (denormalization)
    "email": "phyo@example.com"   ← email copied here too
  },
  "items": [
    {
      "product_id": 5,
      "name": "T-Shirt",          ← product name copied here too
      "price": 19.99,
      "qty": 2
    }
  ],
  "total": 39.98,
  "status": "shipped"
}

No JOIN needed to get order details.
One query → all information in one document.
But if Phyo changes his email → you must update EVERY order document!
```

**The data model is designed for speed.** Reads are fast because everything is in one place, but updates are harder to keep consistent.

---

### ACID vs BASE

This is the most important fundamental difference in behavior between relational and many NoSQL databases.

#### ACID (Relational Databases)

```
A — Atomicity
  "All or nothing" — a transaction either fully completes or fully rolls back.
  Bank transfer: deduct from A AND credit B. If credit fails → deduction is reversed.
  You NEVER end up with money subtracted but not credited.

C — Consistency
  The database always moves from one valid state to another.
  Rules (constraints) are never violated during a transaction.
  A user_id in orders MUST exist in users — always. The DB enforces this.

I — Isolation
  Concurrent transactions don't see each other's partial results.
  User A transferring money doesn't see User B's half-completed transfer.
  Each transaction sees a consistent snapshot of the database.

D — Durability
  Once a transaction is COMMITTED, it's permanent.
  Even if the server loses power immediately after COMMIT → data is safe.
  MySQL writes to a transaction log BEFORE confirming success.
```

#### BASE (Many NoSQL Databases)

```
BA — Basically Available
  The system guarantees availability, even if some data is stale.
  "You'll always get a response, but it might not be the latest data."

S — Soft State
  The state of the system may change over time, even without input.
  Replicas catch up gradually — there's a window where data is inconsistent.

E — Eventually Consistent
  Given enough time (usually milliseconds to seconds), all copies of
  data across all nodes will converge to the same value.
  "It will be consistent... eventually."
```

```
Example of eventual consistency (Cassandra):

  Write: Update Phyo's email on Node 1 → immediately confirmed
  
  t+0ms:   Node 1: "phyo_new@example.com"  ✅
            Node 2: "phyo@example.com"       ← not yet updated
            Node 3: "phyo@example.com"       ← not yet updated
  
  t+50ms:  Node 1: "phyo_new@example.com"  ✅
            Node 2: "phyo_new@example.com"  ✅  ← caught up
            Node 3: "phyo@example.com"       ← still old
  
  t+150ms: All nodes: "phyo_new@example.com" ✅ ← eventually consistent

During that 50-150ms window, different reads might return different values!
This is acceptable for some use cases (social media likes, view counts).
This is NOT acceptable for banking or inventory (use relational for those).
```

---

### Scaling: Vertical vs Horizontal

This is where NoSQL has its strongest advantage over traditional relational databases.

#### Vertical Scaling (Scale Up) — Relational DBs

```
"Make the one machine bigger and more powerful"

  [MySQL Server]
  16GB RAM, 4 CPU, 500GB SSD
        ↓ (need more power)
  [Bigger MySQL Server]
  128GB RAM, 32 CPU, 10TB SSD

  ✅ Simple — same one machine, just bigger
  ✅ No application changes needed
  ❌ Hardware limits exist — you can't scale forever
  ❌ Downtime required for hardware upgrades
  ❌ Expensive — high-end servers cost tens of thousands of dollars
  ❌ Single point of failure — if this machine dies, everything dies
```

#### Horizontal Scaling (Scale Out) — NoSQL DBs

```
"Add more machines to share the load"

  [Cassandra Node 1] ←→ [Cassandra Node 2] ←→ [Cassandra Node 3]
  16GB RAM, 4 CPU        16GB RAM, 4 CPU        16GB RAM, 4 CPU

  Need more capacity? Just add Node 4, Node 5, Node 6...
  Cost: 6x cheap servers instead of 1x massive server

  ✅ No hardware limits — add nodes indefinitely
  ✅ No downtime to scale
  ✅ Commodity hardware (cheap servers)
  ✅ No single point of failure
  ❌ More complex to manage
  ❌ Harder to maintain consistency across nodes (BASE)
```

> 💡 **Modern reality:** PostgreSQL and MySQL can also scale horizontally now using read replicas, sharding (Vitess for MySQL, Citus for PostgreSQL), and distributed setups. The "NoSQL is the only way to scale" narrative from the 2010s is no longer fully accurate. Relational databases can handle enormous scale with the right architecture.

---

### When to Use Which

```
USE RELATIONAL (MySQL/PostgreSQL) when:

✅ Your data has clear relationships (users, orders, products)
✅ Data accuracy and consistency are critical (banking, healthcare, inventory)
✅ You need complex queries with JOINs across multiple tables
✅ You need transactions (money transfers, order processing)
✅ Your schema is relatively stable (doesn't change daily)
✅ You're building a typical web app (most apps!)
✅ Your team knows SQL (everyone does)
✅ You want ACID guarantees

Real examples: E-commerce, banking, hospital systems,
               CMS (WordPress uses MySQL), ERP systems

────────────────────────────────────────────────────────

USE NOSQL when:

✅ Your data structure varies wildly between records
✅ You need to store hierarchical/nested data naturally
✅ You need horizontal scaling to millions of nodes
✅ You need sub-millisecond response times (Redis)
✅ You need real-time data sync across clients (Firestore)
✅ Schema flexibility is more important than strict consistency
✅ You're building at petabyte scale (Cassandra)
✅ You need full-text search (Elasticsearch)

Real examples: Social media feeds, IoT sensor data,
               Real-time chat, Content management with varying fields,
               Product catalogs with diverse attributes
```

---

### Full Comparison Table

| Feature | Relational (MySQL/PostgreSQL) | NoSQL (MongoDB, Redis, etc.) |
|---|---|---|
| Data structure | Tables (rows + columns) | Documents, key-value, graph, wide-column |
| Schema | Fixed (defined upfront) | Flexible (schema-less or schema-optional) |
| Relationships | Built-in (foreign keys + JOIN) | Embedded or application-managed |
| Query language | SQL (standardized) | Varies (MQL, CQL, Cypher, etc.) |
| ACID transactions | ✅ Always | Varies (MongoDB: yes; Cassandra: limited) |
| Consistency | Strong (always consistent) | Eventual (usually) |
| Scaling | Vertical (primarily) | Horizontal (natively) |
| Joins | Native, optimized | Limited or application-level |
| Indexing | Mature, many types | Database-dependent |
| Data integrity | ✅ Enforced by DB | ❌ Application responsibility |
| Learning curve | SQL is universal, well-known | Each DB has its own API |
| Maturity | 50+ years old | 15–20 years old |
| Best for | Most web apps, finance, CRUD | Scale, flexibility, caching, search |
| Popular options | MySQL, PostgreSQL, SQLite | MongoDB, Redis, Cassandra, Elasticsearch |

---

## What is MySQL and Why Use It?

```
MySQL is:
  ✅ The world's most popular open-source relational database
  ✅ The "M" in the LAMP stack (Linux, Apache, MySQL, PHP)
  ✅ Used by 43% of websites worldwide (via WordPress)
  ✅ Supported natively by every PHP framework
  ✅ Available on every hosting provider on earth
  ✅ Free to use (Community Edition)
  ✅ In production at Facebook, YouTube, Twitter, Wikipedia, Airbnb
```

### Why MySQL is the Best Match for PHP

```
PHP + MySQL is the world's most common backend combination because:

1. LAMP Stack: Linux + Apache + MySQL + PHP
   → The original stack that powered the internet boom
   → Every PHP tutorial, framework, and hosting plan assumes MySQL

2. Native PHP support:
   → PDO_MySQL (recommended)
   → MySQLi extension
   → Both built into PHP by default

3. Laravel, Symfony, WordPress, Magento → all default to MySQL

4. Shared hosting: 100% of cPanel-based hosting includes MySQL

5. WAMP, XAMPP, Laragon, MAMP → all bundle PHP + MySQL together
```

### MySQL Key Facts

```
Full name:    MySQL (My = co-creator's daughter's name, SQL = Structured Query Language)
Created:      1995 by Michael Widenius and David Axmark (Sweden)
Owner:        Oracle Corporation (since 2010 via Sun acquisition)
License:      GPL (Community Edition — free) + Commercial (Enterprise)
Version:      8.0 / 8.4 LTS (latest)
Default port: 3306
Default user: root (superuser)
Config file:  /etc/mysql/my.cnf (Linux) / my.ini (Windows)
Data dir:     /var/lib/mysql/ (Linux)
```

---

## MySQL Installation

---

### Windows

#### Method 1 — MySQL Installer (Recommended for Windows)

```
1. Download MySQL Installer from:
   https://dev.mysql.com/downloads/installer/

2. Choose:
   "mysql-installer-community-8.0.x.msi" (includes everything)
   OR
   "mysql-installer-web-community-8.0.x.msi" (smaller, downloads components)

3. Run the installer → Choose Setup Type:
   ┌────────────────────────────────────────┐
   │  Setup Type                            │
   │  ○ Developer Default (recommended)     │ ← MySQL Server + Workbench + tools
   │  ○ Server only                         │ ← Just the database server
   │  ○ Client only                         │
   │  ○ Full                                │
   │  ○ Custom                              │
   └────────────────────────────────────────┘

4. During installation, set the root password when prompted.
   Choose a strong password and REMEMBER IT.

5. Leave the port as 3306 (default).

6. Choose "Use Strong Password Encryption" (recommended for MySQL 8+).

7. Start MySQL Server as Windows Service: YES ✅
   Service Name: MySQL80

8. After installation, verify:
```

```powershell
# In PowerShell or Command Prompt
mysql --version
# Output: mysql  Ver 8.0.xx Distrib 8.0.xx, for Win64 (x86_64)

# Connect to MySQL
mysql -u root -p
# Enter password: (type your root password)
```

#### Method 2 — Chocolatey (Package Manager)

```powershell
# Run PowerShell as Administrator
choco install mysql

# Start MySQL service
net start MySQL80

# Connect
mysql -u root -p
```

#### Method 3 — XAMPP (PHP + MySQL + Apache all-in-one)

```
Perfect for beginners on Windows — one installer, everything included.

1. Download XAMPP from: https://www.apachefriends.org/
2. Run installer → install to C:\xampp
3. Open XAMPP Control Panel
4. Click "Start" next to MySQL (and Apache if needed)
5. MySQL now running on port 3306

Connect via browser: http://localhost/phpmyadmin
(phpMyAdmin comes bundled with XAMPP)

Connect via command line:
C:\xampp\mysql\bin\mysql.exe -u root -p
(No password by default in XAMPP — just press Enter)
```

> ⚠️ **Warning:** XAMPP's default MySQL setup has no root password and is for LOCAL DEVELOPMENT ONLY. Never use XAMPP in production.

---

### macOS

#### Method 1 — Homebrew (Recommended)

```bash
# Install Homebrew first if not installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install MySQL
brew install mysql

# Start MySQL service (auto-starts on login)
brew services start mysql

# OR start manually (without auto-start)
mysql.server start

# Secure the installation (sets root password)
mysql_secure_installation

# Verify installation
mysql --version
# mysql  Ver 8.0.xx Distrib 8.0.xx, for macos14.x (arm64)

# Connect to MySQL
mysql -u root -p
```

#### Method 2 — MySQL Official DMG Installer

```
1. Go to: https://dev.mysql.com/downloads/mysql/
2. Select "macOS" → Download .dmg file
3. Double-click .dmg → follow installer
4. During installation, a temporary root password is shown on screen
   → COPY AND SAVE IT IMMEDIATELY (it's only shown once)
5. After install, go to: System Preferences → MySQL → Start MySQL Server

Connect:
/usr/local/mysql/bin/mysql -u root -p
# Or add to PATH: export PATH=$PATH:/usr/local/mysql/bin
```

#### Method 3 — MAMP (Mac, Apache, MySQL, PHP)

```
1. Download: https://www.mamp.info/
2. Install and open MAMP
3. Click "Start Servers"
4. MySQL runs on port 8889 (MAMP default — different from 3306!)
5. Default credentials: root / root

Access phpMyAdmin: http://localhost:8888/phpmyadmin
```

---

### Ubuntu / Debian

#### Method 1 — APT Package Manager (Recommended for Ubuntu)

```bash
# Step 1: Update package lists
sudo apt update

# Step 2: Install MySQL Server
sudo apt install mysql-server

# Step 3: Check the service is running
sudo systemctl status mysql
# Should show: Active: active (running)

# Step 4: Secure the installation
sudo mysql_secure_installation
```

**What `mysql_secure_installation` asks:**

```
Securing the MySQL server deployment.

VALIDATE PASSWORD COMPONENT:
Press y|Y for Yes, any other key for No: Y
  → Y = enforce password strength rules (recommended for production)
  → N = skip (fine for development)

Please enter 0 = LOW, 1 = MEDIUM, 2 = STRONG: 1
  → 0 = any password
  → 1 = 8+ chars with numbers + uppercase
  → 2 = 8+ chars, numbers, uppercase, symbols, dictionary check

New password: (enter a strong password)
Re-enter new password: (confirm)

Remove anonymous users? Y                    ← YES (always)
Disallow root login remotely? Y              ← YES (security!)
Remove test database and access to it? Y     ← YES (clean up)
Reload privilege tables now? Y               ← YES (apply changes)

All done!
```

```bash
# Step 5: Connect to MySQL
# On Ubuntu, root initially uses auth_socket (no password needed with sudo)
sudo mysql
# OR with password authentication:
mysql -u root -p

# Step 6: Verify
SELECT VERSION();
# +----------+
# | VERSION()|
# +----------+
# | 8.0.36   |
# +----------+
```

#### Install a Specific Version

```bash
# Add MySQL APT repository for specific versions
wget https://dev.mysql.com/get/mysql-apt-config_0.8.29-1_all.deb
sudo dpkg -i mysql-apt-config_0.8.29-1_all.deb
# Select MySQL 8.0 or 8.4 in the dialog
sudo apt update
sudo apt install mysql-server
```

#### Method 2 — Snap Package

```bash
sudo snap install mysql
sudo snap start mysql
sudo mysql -u root
```

---

## MySQL Service Management

Once installed, you need to know how to control the MySQL service.

```bash
# ─── LINUX (systemctl) ───────────────────────────────────────

# Check if MySQL is running
sudo systemctl status mysql

# Start MySQL
sudo systemctl start mysql

# Stop MySQL
sudo systemctl stop mysql

# Restart MySQL (after config changes)
sudo systemctl restart mysql

# Enable auto-start on boot
sudo systemctl enable mysql

# Disable auto-start on boot
sudo systemctl disable mysql

# ─── macOS (Homebrew) ────────────────────────────────────────

brew services start mysql    # Start + auto-start on login
brew services stop mysql     # Stop
brew services restart mysql  # Restart
mysql.server start           # Start manually (no auto-start)
mysql.server stop            # Stop manually

# ─── WINDOWS ─────────────────────────────────────────────────

# In PowerShell as Administrator:
net start MySQL80            # Start
net stop MySQL80             # Stop

# Or via Services: Windows Key → "Services" → MySQL80 → Right-click
```

---

## Securing MySQL After Installation

These are the steps every professional does after installing MySQL — especially on a production server.

```bash
# Run the security wizard
sudo mysql_secure_installation

# After that, verify with:
sudo mysql

# Check current users
SELECT user, host, plugin FROM mysql.user;
# +------------------+-----------+-----------------------+
# | user             | host      | plugin                |
# +------------------+-----------+-----------------------+
# | mysql.infoschema | localhost | caching_sha2_password |
# | mysql.session    | localhost | caching_sha2_password |
# | mysql.sys        | localhost | caching_sha2_password |
# | root             | localhost | auth_socket           |
# +------------------+-----------+-----------------------+
```

### Change Root Authentication from auth_socket to Password

```sql
-- Ubuntu installs root with auth_socket (login via sudo, no password)
-- Change to password-based authentication for remote tools

ALTER USER 'root'@'localhost'
  IDENTIFIED WITH caching_sha2_password
  BY 'YourStrongPassword123!';

FLUSH PRIVILEGES;

-- Now you can connect with:
-- mysql -u root -p
-- (enter your password)
```

> ⚠️ **Production Security Rules:**
> - Never use `root` for your application's database connection.
> - Create a dedicated user for each application with minimal permissions.
> - Disable remote root login (`Disallow root login remotely` = YES).
> - Never expose port 3306 to the public internet.
> - Use strong passwords (12+ characters, mixed case, numbers, symbols).

---

## Connecting to MySQL

### From the Command Line

```bash
# Basic connection (local)
mysql -u root -p
# -u = username
# -p = prompt for password (never put password directly in command!)

# Connect and select a database immediately
mysql -u root -p my_database

# Connect to a remote server
mysql -h 192.168.1.100 -P 3306 -u root -p

# Connect and run a single command (non-interactive)
mysql -u root -p -e "SHOW DATABASES;"

# Run a SQL file
mysql -u root -p my_database < /path/to/script.sql

# Common options:
# -h = host (default: localhost)
# -P = port (default: 3306)
# -u = username
# -p = password (prompted)
# -D = database to use
# -e = execute this SQL and exit
```

### The MySQL Shell (Interactive Mode)

```sql
-- After connecting with: mysql -u root -p

-- See all databases on this server
SHOW DATABASES;
-- +--------------------+
-- | Database           |
-- +--------------------+
-- | information_schema |
-- | mysql              |
-- | performance_schema |
-- | sys                |
-- +--------------------+

-- Switch to a database
USE mysql;

-- See all tables in current database
SHOW TABLES;

-- See current user
SELECT USER();
-- root@localhost

-- See current database
SELECT DATABASE();

-- See MySQL version
SELECT VERSION();

-- Exit MySQL shell
EXIT;
-- OR:
QUIT;
-- OR: Ctrl+D
```

---

## MySQL Server Architecture

Understanding the internal structure of MySQL helps you become a better developer.

```
MySQL Server Internal Architecture:

┌─────────────────────────────────────────────────────────────┐
│                    MySQL Server Process                     │
│                                                             │
│  Connection Layer:                                          │
│  Client connects → Thread created per connection           │
│  Authentication check → Authorization check                │
│                                                             │
│  SQL Layer:                                                 │
│  Query Cache (deprecated in 8.0) → Parser                  │
│  → Preprocessor → Optimizer → Execution Engine             │
│                                                             │
│  Storage Engine Layer (pluggable):                          │
│  ┌──────────┐  ┌─────────┐  ┌────────┐  ┌──────────┐     │
│  │  InnoDB  │  │ MyISAM  │  │ Memory │  │  NDB     │     │
│  │ (default)│  │(legacy) │  │(temp)  │  │(cluster) │     │
│  └──────────┘  └─────────┘  └────────┘  └──────────┘     │
│                                                             │
│  InnoDB Architecture:                                       │
│  Buffer Pool (RAM cache) → Redo Log → Undo Log             │
│  → .ibd files (data + indexes on disk)                     │
└─────────────────────────────────────────────────────────────┘

Key file locations on Linux:
  /var/lib/mysql/          → All database data files
  /etc/mysql/my.cnf        → Main config file
  /var/log/mysql/error.log → Error log
  /var/run/mysqld/         → PID file and socket
```

---

## Databases and Users — The Admin Essentials

This is the core of MySQL administration — what you'll do every time you set up a new project.

---

### Creating and Managing Databases

```sql
-- Connect first:
-- mysql -u root -p

-- VIEW all databases
SHOW DATABASES;

-- CREATE a new database
CREATE DATABASE myapp;

-- CREATE with explicit character set (always do this!)
CREATE DATABASE myapp
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
-- utf8mb4 = full Unicode including emoji and Burmese text
-- utf8mb4_unicode_ci = case-insensitive comparison

-- CREATE only if it doesn't exist (safe — no error if already there)
CREATE DATABASE IF NOT EXISTS myapp
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

-- SELECT (switch to) a database to work in
USE myapp;

-- See which database is currently selected
SELECT DATABASE();

-- VIEW the CREATE statement of a database
SHOW CREATE DATABASE myapp;
-- +--------+-------------------------------------------------------------------+
-- | Database | Create Database                                               |
-- +--------+-------------------------------------------------------------------+
-- | myapp  | CREATE DATABASE `myapp` /*!40100 DEFAULT CHARACTER SET utf8mb4   |
-- |        | COLLATE utf8mb4_unicode_ci */                                     |
-- +--------+-------------------------------------------------------------------+

-- RENAME a database (MySQL has no direct RENAME DATABASE)
-- Approach: create new → migrate data → drop old
-- Most people just use mysqldump for this

-- DROP (delete) a database — PERMANENT! ALL DATA GONE!
DROP DATABASE myapp;
DROP DATABASE IF EXISTS myapp;  -- No error if it doesn't exist
```

> ⚠️ **Critical Warning:** `DROP DATABASE` is instant and **permanent**. There is no undo, no recycle bin, no warning prompt. Always verify you're connected to the right server and right database before running DROP commands. Take a backup first.

---

### Creating and Managing Users

```sql
-- VIEW all existing users
SELECT user, host, plugin, account_locked
FROM mysql.user;

-- ─── CREATE USER ────────────────────────────────────────────

-- Create user for localhost only (cannot connect from other machines)
CREATE USER 'phyo'@'localhost' IDENTIFIED BY 'SecurePassword123!';

-- Create user that can connect from ANY host (use carefully!)
CREATE USER 'phyo'@'%' IDENTIFIED BY 'SecurePassword123!';
-- '%' is a wildcard meaning "any host"

-- Create user for a specific IP only (best for remote access)
CREATE USER 'phyo'@'192.168.1.100' IDENTIFIED BY 'SecurePassword123!';

-- Create user IF NOT EXISTS (safe — no error if already exists)
CREATE USER IF NOT EXISTS 'phyo'@'localhost' IDENTIFIED BY 'SecurePassword123!';

-- Create user that must connect with SSL
CREATE USER 'secure_user'@'%'
  IDENTIFIED BY 'Password123!'
  REQUIRE SSL;

-- ─── VIEW USER INFO ──────────────────────────────────────────

-- Check what a user exists
SELECT user, host FROM mysql.user WHERE user = 'phyo';

-- Check what permissions a user has
SHOW GRANTS FOR 'phyo'@'localhost';

-- ─── CHANGE PASSWORD ─────────────────────────────────────────

-- MySQL 8.0+ (recommended way)
ALTER USER 'phyo'@'localhost' IDENTIFIED BY 'NewPassword456!';

-- Apply privilege changes immediately
FLUSH PRIVILEGES;

-- ─── RENAME A USER ───────────────────────────────────────────

RENAME USER 'phyo'@'localhost' TO 'phyo_min'@'localhost';

-- ─── LOCK / UNLOCK A USER ────────────────────────────────────
-- Lock: prevent user from logging in (without deleting)
ALTER USER 'phyo'@'localhost' ACCOUNT LOCK;

-- Unlock: re-enable login
ALTER USER 'phyo'@'localhost' ACCOUNT UNLOCK;

-- ─── DELETE A USER ───────────────────────────────────────────

DROP USER 'phyo'@'localhost';
DROP USER IF EXISTS 'phyo'@'localhost';  -- No error if not found
```

### Understanding `user@host`

```
MySQL users are identified by TWO parts: username AND host

'phyo'@'localhost'    → Phyo connecting from the same machine
'phyo'@'%'           → Phyo connecting from anywhere
'phyo'@'192.168.1.%' → Phyo connecting from 192.168.1.x subnet
'phyo'@'10.0.0.5'    → Phyo connecting only from IP 10.0.0.5

IMPORTANT: 'phyo'@'localhost' and 'phyo'@'127.0.0.1' are DIFFERENT users!
Even though localhost resolves to 127.0.0.1, MySQL treats them separately.
```

---

### Granting Permissions (GRANT)

This is one of the most important commands — it controls what each user can do.

```sql
-- ─── GRANT SYNTAX ────────────────────────────────────────────
GRANT privilege_list ON scope TO 'user'@'host';

-- ─── GRANT SPECIFIC PRIVILEGES ON ONE DATABASE ───────────────

-- Full access to one database (typical for a web app user)
GRANT ALL PRIVILEGES ON myapp.* TO 'phyo'@'localhost';
--         ↑                 ↑
--   All privileges    myapp database, all tables (*.*)

-- Read-only access (SELECT only)
GRANT SELECT ON myapp.* TO 'readonly_user'@'localhost';

-- Specific privileges
GRANT SELECT, INSERT, UPDATE, DELETE ON myapp.* TO 'app_user'@'localhost';

-- Only on a specific table
GRANT SELECT, INSERT ON myapp.orders TO 'phyo'@'localhost';

-- Only specific columns
GRANT SELECT (id, name, email) ON myapp.users TO 'phyo'@'localhost';

-- ─── GRANT SUPERUSER PRIVILEGES (be careful!) ────────────────

-- Grant ALL privileges on ALL databases (superuser level)
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' WITH GRANT OPTION;
-- WITH GRANT OPTION = this user can grant privileges to others

-- ─── LIST OF ALL AVAILABLE PRIVILEGES ────────────────────────
-- SELECT          → Read rows
-- INSERT          → Add rows
-- UPDATE          → Modify rows
-- DELETE          → Remove rows
-- CREATE          → Create tables/databases
-- DROP            → Delete tables/databases
-- INDEX           → Manage indexes
-- ALTER           → Modify table structure
-- CREATE VIEW     → Create views
-- SHOW VIEW       → See view definitions
-- TRIGGER         → Create triggers
-- CREATE ROUTINE  → Create stored procedures/functions
-- EXECUTE         → Run stored procedures
-- LOCK TABLES     → Use LOCK TABLES
-- REFERENCES      → Create foreign key constraints
-- PROCESS         → See all running queries (SHOW PROCESSLIST)
-- RELOAD          → FLUSH privileges/tables/logs
-- SUPER           → Kill connections, set global variables
-- ALL PRIVILEGES  → Everything above

-- ─── APPLY CHANGES ───────────────────────────────────────────
-- Always run FLUSH PRIVILEGES after changes to grant tables
-- (not always required after GRANT/REVOKE, but good practice)
FLUSH PRIVILEGES;

-- ─── VERIFY GRANTS ───────────────────────────────────────────
SHOW GRANTS FOR 'phyo'@'localhost';
-- +-----------------------------------------------------------------------+
-- | Grants for phyo@localhost                                              |
-- +-----------------------------------------------------------------------+
-- | GRANT USAGE ON *.* TO `phyo`@`localhost`                              |
-- | GRANT ALL PRIVILEGES ON `myapp`.* TO `phyo`@`localhost`               |
-- +-----------------------------------------------------------------------+
```

---

### Revoking Permissions

```sql
-- REVOKE specific privileges
REVOKE DELETE ON myapp.* FROM 'phyo'@'localhost';

-- REVOKE all privileges on a database
REVOKE ALL PRIVILEGES ON myapp.* FROM 'phyo'@'localhost';

-- REVOKE all privileges everywhere
REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'phyo'@'localhost';

-- Verify revocation
SHOW GRANTS FOR 'phyo'@'localhost';
```

### Complete New Project Setup Workflow

```sql
-- This is what you do every time you start a new PHP project

-- 1. Create the database
CREATE DATABASE myapp_db
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

-- 2. Create a dedicated application user (never use root for your app!)
CREATE USER 'myapp_user'@'localhost' IDENTIFIED BY 'App_Password_2026!';

-- 3. Grant the user access to ONLY the app's database
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER
  ON myapp_db.*
  TO 'myapp_user'@'localhost';

-- 4. Apply privileges
FLUSH PRIVILEGES;

-- 5. Verify
SHOW GRANTS FOR 'myapp_user'@'localhost';

-- 6. Test the new user connection (in another terminal)
-- mysql -u myapp_user -p myapp_db

-- 7. Create your tables (covered in SQL notes)
USE myapp_db;
CREATE TABLE users (...);
```

---

## MySQL Configuration File (my.cnf)

- The configuration file controls MySQL's behavior — memory limits, character sets, networking, performance.
- **Linux:** `/etc/mysql/my.cnf` or `/etc/mysql/mysql.conf.d/mysqld.cnf`
- **macOS (Homebrew):** `/usr/local/etc/my.cnf` or `/opt/homebrew/etc/my.cnf`
- **Windows:** `C:\ProgramData\MySQL\MySQL Server 8.0\my.ini`

```ini
# /etc/mysql/mysql.conf.d/mysqld.cnf  (Ubuntu)

[mysqld]
# ── Networking ──────────────────────────────────────────────
bind-address            = 127.0.0.1   # Only listen on localhost (security!)
                                        # Change to 0.0.0.0 for remote access
port                    = 3306

# ── Character Set ────────────────────────────────────────────
character-set-server    = utf8mb4     # Support emoji, Burmese, all Unicode
collation-server        = utf8mb4_unicode_ci

# ── Memory & Performance ─────────────────────────────────────
innodb_buffer_pool_size = 256M        # RAM for caching data+indexes (set to 50-70% of RAM)
max_connections         = 150         # Max simultaneous connections
query_cache_size        = 0           # Disable query cache (deprecated in 8.0)
innodb_flush_log_at_trx_commit = 1   # 1=full ACID, 2=faster but less safe

# ── Logging ──────────────────────────────────────────────────
log_error               = /var/log/mysql/error.log
slow_query_log          = ON          # Log slow queries for optimization
slow_query_log_file     = /var/log/mysql/slow.log
long_query_time         = 2           # Log queries taking > 2 seconds
general_log             = OFF         # Log ALL queries (ONLY for debugging — huge file!)

# ── Security ─────────────────────────────────────────────────
local-infile            = 0           # Disable LOAD DATA LOCAL INFILE (security)

[mysql]
# Default settings for the mysql command-line client
default-character-set   = utf8mb4

[client]
default-character-set   = utf8mb4
```

```bash
# After editing my.cnf, restart MySQL to apply changes
sudo systemctl restart mysql

# Verify character set
mysql -u root -p -e "SHOW VARIABLES LIKE 'character_set%';"
# +--------------------------+----------------------------+
# | Variable_name            | Value                      |
# +--------------------------+----------------------------+
# | character_set_client     | utf8mb4                    |
# | character_set_connection | utf8mb4                    |
# | character_set_database   | utf8mb4                    |
# | character_set_server     | utf8mb4                    |
# +--------------------------+----------------------------+
```

---

## Connecting MySQL to PHP

The two ways PHP connects to MySQL — **PDO** (recommended) and **MySQLi**.

```php
<?php
// ─── METHOD 1: PDO (PHP Data Objects) — RECOMMENDED ────────
// Works with MySQL, PostgreSQL, SQLite — portable
$dsn = "mysql:host=localhost;port=3306;dbname=myapp_db;charset=utf8mb4";

try {
    $pdo = new PDO(
        $dsn,
        "myapp_user",    // username
        "App_Password_2026!",  // password
        [
            PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,  // Throw exceptions on error
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,        // Fetch as associative array
            PDO::ATTR_EMULATE_PREPARES   => false,                    // Use real prepared statements
        ]
    );
    echo "Connected successfully via PDO";
} catch (PDOException $e) {
    // Log the error — never show to users!
    error_log("DB Connection failed: " . $e->getMessage());
    die("Database connection failed. Please try again later.");
}

// ─── METHOD 2: MySQLi (MySQL-specific) ──────────────────────
$mysqli = new mysqli("localhost", "myapp_user", "App_Password_2026!", "myapp_db");

if ($mysqli->connect_error) {
    error_log("MySQLi connection failed: " . $mysqli->connect_error);
    die("Database connection failed.");
}

$mysqli->set_charset("utf8mb4");
echo "Connected via MySQLi";

// ─── PHP + MySQL Config (use environment variables!) ─────────
// .env file:
// DB_HOST=localhost
// DB_PORT=3306
// DB_NAME=myapp_db
// DB_USER=myapp_user
// DB_PASS=App_Password_2026!

// PHP code:
$pdo = new PDO(
    sprintf(
        "mysql:host=%s;port=%s;dbname=%s;charset=utf8mb4",
        getenv("DB_HOST"),
        getenv("DB_PORT") ?: "3306",
        getenv("DB_NAME")
    ),
    getenv("DB_USER"),
    getenv("DB_PASS"),
    [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
);
?>
```

---

## MySQL GUI Tools

Command-line is powerful, but GUI tools make daily database work faster and more visual.

| Tool | Platform | Cost | Best For |
|---|---|---|---|
| **MySQL Workbench** | Win/Mac/Linux | Free | Official tool, EER diagrams, admin |
| **phpMyAdmin** | Web browser | Free | Web-based, shared hosting standard |
| **TablePlus** | Win/Mac/Linux | Free + Paid | Beautiful UI, multiple DB support |
| **DBeaver** | Win/Mac/Linux | Free + Paid | Multi-DB, great for all databases |
| **HeidiSQL** | Windows | Free | Simple, fast, Windows-focused |
| **Sequel Pro** | macOS only | Free | Mac-native, clean interface |
| **DataGrip** | Win/Mac/Linux | Paid (JetBrains) | Best IDE-level DB tool, AI assistance |

### phpMyAdmin — Installation on Ubuntu

```bash
# Install phpMyAdmin (web-based MySQL admin)
sudo apt install phpmyadmin

# During install:
# Select web server: apache2 (or nginx)
# Configure database? YES
# Set phpMyAdmin password

# Access at: http://localhost/phpmyadmin
# Login with: root + your root password
```

---

## Quick Revision

- **Relational databases** model the world as **separate linked tables** — data lives in one place, linked by keys. **NoSQL** stores data **together** in one document/record — fast to read, harder to keep consistent.
- **ACID** (relational) = all-or-nothing transactions, always consistent. **BASE** (NoSQL) = basically available, eventually consistent. Use relational for money/inventory/accuracy. Use NoSQL for scale/flexibility/speed.
- **Vertical scaling** = bigger machine (relational default). **Horizontal scaling** = more machines (NoSQL native). PostgreSQL and MySQL can also scale horizontally with the right tools.
- **MySQL** is the most popular open-source web database — the M in LAMP — used by WordPress, YouTube, Airbnb, and 43% of the web. Perfect match for PHP.
- **Install MySQL:** Windows → MySQL Installer or XAMPP. macOS → `brew install mysql`. Ubuntu → `sudo apt install mysql-server`.
- **Always run** `mysql_secure_installation` after installing MySQL — sets root password, removes test db, disables remote root login.
- **Connect:** `mysql -u root -p` — never put the password in the command itself.
- **Every user is `'username'@'host'`** — `'phyo'@'localhost'` and `'phyo'@'%'` are completely different users.
- **New project workflow:** `CREATE DATABASE myapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;` → `CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'password';` → `GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON myapp.* TO 'app_user'@'localhost';` → `FLUSH PRIVILEGES;`.
- **Never use `root` in your application connection** — always create a dedicated user with only the permissions the app needs (principle of least privilege).
- **Always use `utf8mb4`** (not `utf8`) as the character set — it supports full Unicode including emoji and Burmese characters. MySQL's `utf8` is broken (only 3 bytes, not full Unicode).
- **PDO is preferred** over MySQLi for PHP connections — it supports multiple database types and has a cleaner API.
- **Set `bind-address = 127.0.0.1`** in `my.cnf` to prevent external connections. Never expose port 3306 to the public internet.
- **`FLUSH PRIVILEGES`** — run after manually editing grant tables. Not strictly needed after `GRANT`/`REVOKE` but is good practice.