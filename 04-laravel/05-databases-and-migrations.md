# Laravel 05 — Database, MySQL, phpMyAdmin & Migrations

A migration is a version-controlled PHP file that defines changes to your database structure, so your whole team can build and update the same database schema without ever touching phpMyAdmin by hand.

## Table of Contents

- [Connecting Laravel to MySQL](#connecting-laravel-to-mysql)
- [What phpMyAdmin Is For](#what-phpmyadmin-is-for)
- [What a Migration Actually Is](#what-a-migration-actually-is)
- [Creating a Migration](#creating-a-migration)
- [Anatomy of a Migration File](#anatomy-of-a-migration-file)
- [Running Migrations](#running-migrations)
- [Rolling Back Migrations](#rolling-back-migrations)
- [Every Migration Command Explained](#every-migration-command-explained)
- [Column Types Reference](#column-types-reference)
- [Column Modifiers](#column-modifiers)
- [Foreign Keys & Relationships in Migrations](#foreign-keys--relationships-in-migrations)
- [Indexes](#indexes)
- [Seeders](#seeders)
- [Laravel Faker](#laravel-faker)
- [Factories](#factories)
- [Quick Revision](#quick-revision)

---

## Connecting Laravel to MySQL

Open your `.env` file (from note 01) and set these values to match your local MySQL setup:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=my_app_db
DB_USERNAME=root
DB_PASSWORD=
```

- **`DB_DATABASE`**: the name of the actual database you want Laravel to use — you must create this database first (via phpMyAdmin or the MySQL CLI) before running migrations.
- **`DB_USERNAME` / `DB_PASSWORD`**: your MySQL login credentials. On a fresh local install (e.g., via XAMPP/Laragon), the default is often `root` with a blank password.

> ⚠️ **Warning:** After editing `.env`, if Laravel doesn't seem to pick up the change, run `php artisan config:clear` — Laravel caches config in some situations, and a stale cache can silently keep old values active.

---

## What phpMyAdmin Is For

- **phpMyAdmin**: a web-based graphical interface for managing MySQL databases — letting you view tables, run SQL queries, and inspect data visually instead of using the command line.
- You'll use phpMyAdmin mainly to: create the initial empty database, visually inspect what your migrations actually produced, and manually poke at data while debugging.

> 💡 **Tip:** You will **not** use phpMyAdmin to build your actual table structure once you're using Laravel properly — that's what migrations are for. Treat phpMyAdmin as a read/inspect tool, not your primary schema-design tool.

| Task | Tool to Use |
|---|---|
| Create the empty database itself | phpMyAdmin (once) |
| Define/change table structure | Migrations (Laravel) |
| Insert real application data | Your app itself (forms, seeders) |
| Quickly inspect/debug data | phpMyAdmin |

---

## What a Migration Actually Is

- **Migration**: a PHP file describing one specific database change (e.g., "create the products table," "add a `discount` column to products") — stored in `database/migrations/`.
- Migrations solve a very real team problem: instead of telling a teammate "hey, manually add a `phone` column to your local `users` table," you commit a migration file to Git, and they run one command to get the exact same change.

> 💡 **Tip:** Think of migrations as **Git for your database structure**. Each migration is a "commit" that moves the schema from one state to the next.

---

## Creating a Migration

```bash
php artisan make:migration create_products_table
```

- Laravel automatically prefixes the generated file with a timestamp (e.g., `2026_01_15_120000_create_products_table.php`) so migrations always run in the correct chronological order.
- Because the name starts with `create_..._table`, Laravel is smart enough to pre-fill the file with a `Schema::create(...)` block already targeting a `products` table.

**Adding a column to an existing table:**

```bash
php artisan make:migration add_discount_to_products_table --table=products
```

- **`--table=products`**: tells Laravel this migration modifies an existing `products` table, rather than creating a brand-new one — it pre-fills a `Schema::table(...)` block instead.

---

## Anatomy of a Migration File

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('description')->nullable();
            $table->decimal('price', 8, 2);
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('products');
    }
};
```

- **`return new class extends Migration`**: modern Laravel migrations are anonymous classes (no class name to accidentally duplicate across files).
- **`up()`**: what happens when you *run* this migration — here, creating the table.
- **`down()`**: what happens when you *undo* this migration — should always be the exact reverse of `up()`.
- **`Schema::create('products', function (Blueprint $table) {...})`**: `Schema` is Laravel's database-agnostic tool for building tables; `$table` (a `Blueprint` instance) is what you call column-defining methods on.

> ⚠️ **Warning:** Always keep `down()` accurate. If `up()` creates a table but `down()` doesn't drop it (or drops the wrong one), rolling back will leave your database in a broken, inconsistent state.

---

## Running Migrations

```bash
php artisan migrate
```

- Runs every migration file that **hasn't been run yet**, in timestamp order.
- Laravel tracks which migrations have already run in a special `migrations` table inside your database itself — so re-running `php artisan migrate` is always safe; it skips anything already applied.

---

## Rolling Back Migrations

```bash
# Undo the LAST batch of migrations
php artisan migrate:rollback

# Undo the last 3 batches
php artisan migrate:rollback --step=3

# Undo EVERY migration
php artisan migrate:reset

# Undo everything, then re-run everything from scratch
php artisan migrate:fresh

# Rollback everything, then re-migrate (uses down() then up())
php artisan migrate:refresh
```

| Command | What It Does | When to Use |
|---|---|---|
| `migrate:rollback` | Undoes the most recent batch | Undo a mistake you just made |
| `migrate:reset` | Undoes ALL migrations | Rarely — full teardown |
| `migrate:fresh` | Drops all tables, then re-runs everything from zero | Local dev, want a clean slate fast |
| `migrate:refresh` | Rolls back then re-migrates (runs `down()` then `up()`) | Local dev, testing rollback logic itself |
| `migrate:fresh --seed` | Same as `fresh`, then runs seeders automatically | Rebuilding local dev data |

> ⚠️ **Warning:** `migrate:fresh` and `migrate:reset` **permanently delete all data** in the affected tables. Never run these against a production database — they're strictly local/dev tools.

---

## Every Migration Command Explained

| Command | Purpose |
|---|---|
| `php artisan make:migration name` | Create a new blank migration file |
| `php artisan make:migration name --create=table` | Create a migration pre-filled for a new table |
| `php artisan make:migration name --table=table` | Create a migration pre-filled to modify an existing table |
| `php artisan migrate` | Run all pending migrations |
| `php artisan migrate:status` | Show which migrations have run and which haven't |
| `php artisan migrate:rollback` | Undo the last batch of migrations |
| `php artisan migrate:rollback --step=N` | Undo the last N batches |
| `php artisan migrate:reset` | Undo every migration |
| `php artisan migrate:refresh` | Roll back everything, then re-run it all |
| `php artisan migrate:fresh` | Drop everything, then re-run all migrations clean |

> 💡 **Tip:** Run `php artisan migrate:status` any time you're confused about "did this already run?" — it's your source of truth.

---

## Column Types Reference

These are called inside the `Blueprint $table` closure. This is the vocabulary you'll use constantly.

| Method | MySQL Result | Use For |
|---|---|---|
| `$table->id()` | `BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY` | Standard primary key (named `id`) |
| `$table->string('name')` | `VARCHAR(255)` | Short text (names, titles, emails) |
| `$table->string('name', 100)` | `VARCHAR(100)` | Short text with a custom max length |
| `$table->text('body')` | `TEXT` | Longer text (article bodies, descriptions) |
| `$table->longText('content')` | `LONGTEXT` | Very large text content |
| `$table->integer('quantity')` | `INT` | Whole numbers |
| `$table->unsignedInteger('quantity')` | `INT UNSIGNED` | Whole numbers, no negatives allowed |
| `$table->bigInteger('views')` | `BIGINT` | Very large whole numbers |
| `$table->decimal('price', 8, 2)` | `DECIMAL(8,2)` | Exact-precision numbers (**always use for money**) |
| `$table->float('rating')` | `FLOAT` | Approximate decimal numbers |
| `$table->boolean('is_active')` | `TINYINT(1)` | True/false flags |
| `$table->date('birth_date')` | `DATE` | Date only, no time |
| `$table->dateTime('published_at')` | `DATETIME` | Date and time together |
| `$table->timestamp('verified_at')` | `TIMESTAMP` | Date/time, often used for event timestamps |
| `$table->timestamps()` | Adds both `created_at` and `updated_at` | Standard on almost every table |
| `$table->softDeletes()` | Adds a nullable `deleted_at` column | "Soft delete" (hide, don't truly delete) rows |
| `$table->enum('status', ['draft','published'])` | `ENUM(...)` | A fixed, limited set of allowed string values |
| `$table->json('metadata')` | `JSON` | Storing structured JSON data in one column |
| `$table->uuid('id')` | `CHAR(36)` | UUID-based identifiers instead of auto-increment |
| `$table->foreignId('user_id')` | `BIGINT UNSIGNED` | A column meant to reference another table's `id` |

> ⚠️ **Warning:** Never store money using `float` or `double` — floating-point rounding errors can silently corrupt financial data over time. Always use `decimal('price', 8, 2)` for currency.

---

## Column Modifiers

Chained onto a column definition to fine-tune its behavior:

```php
$table->string('phone')->nullable();
$table->string('email')->unique();
$table->integer('stock')->default(0);
$table->string('slug')->index();
```

| Modifier | Effect |
|---|---|
| `->nullable()` | Allows the column to be empty/`NULL` |
| `->unique()` | No two rows can share the same value in this column |
| `->default($value)` | Sets a default value if none is provided |
| `->index()` | Speeds up lookups/searches on this column |
| `->comment('text')` | Adds a database-level comment (visible in phpMyAdmin) |
| `->after('column')` | Positions the column after a specific existing column (MySQL only) |

> 💡 **Tip:** Every column is `NOT NULL` by default in Laravel migrations — forgetting `->nullable()` on an optional field is one of the most common beginner errors, causing "field doesn't have a default value" errors when saving.

---

## Foreign Keys & Relationships in Migrations

To connect two tables (e.g., every `product` belongs to a `category`):

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->foreignId('category_id')->constrained()->onDelete('cascade');
    $table->timestamps();
});
```

- **`$table->foreignId('category_id')`**: creates an unsigned big integer column named `category_id`.
- **`->constrained()`**: automatically assumes this references the `id` column on the `categories` table (guessed from the column name `category_id` → `categories`).
- **`->onDelete('cascade')`**: if the referenced category is deleted, automatically delete every product pointing to it too (prevents "orphaned" rows).

Other `onDelete` options: `'restrict'` (block the delete if related rows exist — the safer default for many real apps) and `'set null'` (requires the column to be `nullable()`).

> ⚠️ **Warning:** Migrations that add foreign keys must run **after** the migration that creates the referenced table — Laravel's timestamp-based ordering handles this automatically as long as you create tables in a logical order (e.g., `categories` before `products`).

---

## Indexes

- **Index**: a database-level structure that dramatically speeds up searching/filtering on a column, at the small cost of slightly slower writes and extra storage.

```php
$table->string('slug')->index();

// Composite index across multiple columns
$table->index(['user_id', 'created_at']);
```

> 💡 **Tip:** As a rule of thumb, index any column you frequently filter (`WHERE`), sort (`ORDER BY`), or join on. Don't index everything blindly — indexes have a real cost on write performance, which matters at senior level when designing tables for scale.

---

## Seeders

**Seeder**: a PHP class that inserts data into your database — useful for both essential starter data (like default roles) and realistic dummy data during development.

```bash
php artisan make:seeder ProductSeeder
```

```php
// database/seeders/ProductSeeder.php
public function run(): void
{
    DB::table('products')->insert([
        'name' => 'Sample Product',
        'price' => 19.99,
        'created_at' => now(),
        'updated_at' => now(),
    ]);
}
```

Run it:

```bash
php artisan db:seed --class=ProductSeeder
```

Or register it in `database/seeders/DatabaseSeeder.php` so it runs automatically with `php artisan migrate:fresh --seed`.

---

## Laravel Faker

**Faker**: a PHP library bundled with Laravel that generates realistic-looking fake data (names, emails, addresses, paragraphs) — so you don't have to manually type 50 fake products to test your app.

```php
use Faker\Factory as Faker;

$faker = Faker::create();

echo $faker->name;          // "Judy Kertzmann"
echo $faker->email;         // "judy@example.org"
echo $faker->sentence;      // A random fake sentence
echo $faker->numberBetween(10, 500); // A random number in range
```

| Faker Method | Produces |
|---|---|
| `$faker->name` | A random full name |
| `$faker->email` | A random fake email address |
| `$faker->address` | A random fake street address |
| `$faker->sentence` / `$faker->paragraph` | Random fake text |
| `$faker->word` | A single random word |
| `$faker->boolean` | Random `true`/`false` |
| `$faker->numberBetween($min, $max)` | Random integer in range |
| `$faker->dateTimeBetween($start, $end)` | Random date/time in range |

> 💡 **Tip:** Faker is almost never used directly like above in real projects — it's meant to be used **inside Factories** (next section), which is the standard, structured way Laravel expects you to generate fake data.

---

## Factories

**Factory**: a blueprint class that defines *how* to generate one fake instance of a Model, using Faker internally — letting you generate 1 or 1,000 realistic fake rows with one line of code.

```bash
php artisan make:model Product -f
```

- `-f` (or `--factory`) generates a matching factory alongside the model.

```php
// database/factories/ProductFactory.php
public function definition(): array
{
    return [
        'name' => fake()->words(3, true),
        'description' => fake()->paragraph(),
        'price' => fake()->randomFloat(2, 5, 500),
    ];
}
```

**Using the factory** (commonly inside a Seeder):

```php
Product::factory()->count(50)->create();
```

- This single line inserts 50 realistic, randomly-varied fake products into your database — instantly.

> 💡 **Tip:** `fake()` is a global helper function available anywhere in Laravel (introduced to save you from importing the Faker class manually) — it returns the same Faker instance under the hood.

---

## Quick Revision

- Configure MySQL connection details in `.env` (`DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`, etc.); create the actual empty database via **phpMyAdmin** first.
- phpMyAdmin is for creating the database and inspecting data — **migrations**, not phpMyAdmin, are how you actually define and change table structure in a real Laravel project.
- A **migration** is a version-controlled PHP file with `up()` (apply the change) and `down()` (undo it) — create one with `php artisan make:migration`, using `--create=table` or `--table=table` to pre-fill it correctly.
- `php artisan migrate` applies pending migrations; `migrate:rollback` undoes the last batch; `migrate:fresh` wipes and rebuilds everything (local dev only — **never run destructive commands against production**).
- Learn the core column types (`string`, `text`, `decimal` for money, `boolean`, `timestamps()`, `foreignId()`) and modifiers (`nullable()`, `unique()`, `default()`, `index()`) — this vocabulary is used constantly.
- `foreignId('x_id')->constrained()->onDelete(...)` is the standard way to link tables and control what happens when a referenced row is deleted.
- **Seeders** insert data into your database (`php artisan make:seeder`); **Faker** generates realistic fake data; **Factories** combine both to bulk-generate fake Model records (`Product::factory()->count(50)->create()`).
- Next up (note 06): **Eloquent ORM Basics** — using Models to query and manipulate this database data in PHP, plus your first look at Tinker.