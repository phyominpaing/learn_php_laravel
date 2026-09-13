# Laravel 06 — Eloquent ORM Basics

Eloquent is Laravel's built-in ORM (Object-Relational Mapper) that lets you query and manipulate database tables using PHP objects and methods instead of writing raw SQL.

## Table of Contents

- [What an ORM Actually Solves](#what-an-orm-actually-solves)
- [Creating a Model](#creating-a-model)
- [Eloquent's Naming Conventions](#eloquents-naming-conventions)
- [Retrieving Data](#retrieving-data)
- [Query Builder Methods](#query-builder-methods)
- [Creating Records](#creating-records)
- [Mass Assignment & `$fillable`](#mass-assignment--fillable)
- [Updating Records](#updating-records)
- [Deleting Records](#deleting-records)
- [Soft Deletes](#soft-deletes)
- [Accessors & Mutators](#accessors--mutators)
- [Your First Relationships: One-to-One & One-to-Many](#your-first-relationships-one-to-one--one-to-many)
- [Tinker — Laravel's Interactive Console](#tinker--laravels-interactive-console)
- [Quick Revision](#quick-revision)

---

## What an ORM Actually Solves

Without an ORM, working with a database in plain PHP looks like this:

```php
$result = $pdo->query("SELECT * FROM products WHERE stock > 0");
```

- You write raw SQL as strings, manually handle escaping (to avoid SQL injection), and manually convert database rows back into usable PHP data.

With Eloquent, the same thing looks like this:

```php
$products = Product::where('stock', '>', 0)->get();
```

- **ORM (Object-Relational Mapper)**: a layer that maps database tables to PHP classes ("Models") and rows to PHP objects, so you interact with your database using normal PHP syntax instead of SQL strings.
- Eloquent also automatically protects you from SQL injection as long as you use its query methods (rather than manually concatenating raw SQL).

> 💡 **Tip:** You can absolutely still drop down to raw SQL when needed (via the `DB` facade) — but for 90%+ of everyday work, Eloquent is faster to write, easier to read, and safer by default.

---

## Creating a Model

```bash
php artisan make:model Product
```

This creates `app/Models/Product.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    //
}
```

**Generate a model together with related files in one command:**

```bash
php artisan make:model Product -m      # + migration
php artisan make:model Product -f      # + factory
php artisan make:model Product -s      # + seeder
php artisan make:model Product -c      # + controller
php artisan make:model Product --all   # + migration, factory, seeder, controller, policy, form requests
```

| Flag | Also Generates |
|---|---|
| `-m` / `--migration` | A migration file for this model's table |
| `-f` / `--factory` | A factory (for fake data — note 05) |
| `-s` / `--seed` | A seeder |
| `-c` / `--controller` | A controller |
| `-R` / `--resource` | A controller pre-filled with resource methods |
| `--api` | An API-style controller (no `create`/`edit`) |
| `--all` | Everything above, in one shot |

> 💡 **Tip:** `php artisan make:model Product --all` is a huge time-saver once you know what you're doing — it scaffolds an entire feature's skeleton in one command. As a beginner, generate pieces individually first so you understand what each file actually does.

---

## Eloquent's Naming Conventions

Eloquent guesses a lot of things automatically based on naming, following predictable rules:

| Convention | Rule | Example |
|---|---|---|
| Model name | Singular, StudlyCase | `Product` |
| Table name | Plural, snake_case (auto-guessed) | `products` |
| Primary key | Assumed to be `id` | `id` |
| Foreign key (in relationships) | Singular model name + `_id` | `product_id` |

- If your table name doesn't follow convention, override it explicitly:

```php
class Product extends Model
{
    protected $table = 'store_products';
}
```

> ⚠️ **Warning:** Fighting Eloquent's naming conventions constantly (custom table names, custom primary keys, etc.) makes your code harder for other Laravel developers to read and maintain. Stick to convention unless you have a real reason not to — this is a genuine senior-level judgment call: consistency beats cleverness.

---

## Retrieving Data

```php
// Get every row
$products = Product::all();

// Get every row matching a condition
$products = Product::where('stock', '>', 0)->get();

// Get the first matching row (or null)
$product = Product::where('name', 'Laptop')->first();

// Get one row by primary key, or null
$product = Product::find(1);

// Get one row by primary key, or throw a 404 automatically
$product = Product::findOrFail(1);
```

| Method | Returns |
|---|---|
| `all()` | Every row, as a Collection |
| `get()` | Every row matching the query, as a Collection |
| `first()` | The first matching row, or `null` |
| `find($id)` | One row by primary key, or `null` |
| `findOrFail($id)` | One row by primary key, or a 404 exception |
| `firstOrFail()` | First matching row, or a 404 exception |

- **Collection**: a powerful array-like object Eloquent returns for multiple results, with built-in helper methods (`->map()`, `->filter()`, `->pluck()`, etc.) beyond what plain PHP arrays offer.

> 💡 **Tip:** Prefer `findOrFail()` / `firstOrFail()` in controllers over `find()` / `first()` — letting Laravel auto-generate a proper 404 response is cleaner than manually checking `if ($product === null)` everywhere.

---

## Query Builder Methods

Eloquent methods chain together fluently to build more specific queries:

```php
$products = Product::where('stock', '>', 0)
    ->where('price', '<', 100)
    ->orderBy('created_at', 'desc')
    ->limit(10)
    ->get();
```

| Method | Purpose |
|---|---|
| `->where('col', 'value')` | Filter rows where column equals value |
| `->where('col', '>', 10)` | Filter with a comparison operator |
| `->orWhere(...)` | Add an "OR" condition |
| `->whereIn('col', [1,2,3])` | Filter where column matches any value in the array |
| `->whereNull('col')` | Filter where column is `NULL` |
| `->orderBy('col', 'asc\|desc')` | Sort results |
| `->limit(10)` | Cap the number of results |
| `->count()` | Count matching rows instead of fetching them |
| `->pluck('name')` | Get just one column's values as a flat list |
| `->paginate(15)` | Get results split into pages of 15, with pagination links |

> 💡 **Tip:** `->paginate()` is what you'll use constantly for real-world lists (product catalogs, admin tables) instead of dumping thousands of rows onto one page.

---

## Creating Records

```php
$product = new Product();
$product->name = 'Wireless Mouse';
$product->price = 25.99;
$product->save();
```

Or, more commonly, in one step:

```php
$product = Product::create([
    'name' => 'Wireless Mouse',
    'price' => 25.99,
]);
```

- **`create()`**: instantiates a new Model, fills its attributes from the array, and saves it to the database — all in one call.

---

## Mass Assignment & `$fillable`

> ⚠️ **Warning:** Running `Product::create([...])` with an array straight from user input, on a fresh Model with no protection set up, will throw a `MassAssignmentException` — and that's Laravel protecting you on purpose.

- **Mass Assignment**: passing an entire array of data into `create()` or `fill()` at once, instead of setting each attribute individually.
- **Mass Assignment Vulnerability**: if every column were mass-assignable by default, a malicious user could inject an unexpected field (like `is_admin: true`) into a form submission and silently grant themselves admin rights.

Laravel's fix: you must explicitly declare which fields are safe to mass-assign.

```php
class Product extends Model
{
    protected $fillable = ['name', 'description', 'price'];
}
```

- **`$fillable`**: a whitelist — only these listed fields can be set via mass assignment.

The inverse approach:

```php
class Product extends Model
{
    protected $guarded = ['id', 'is_admin'];
}
```

- **`$guarded`**: a blacklist — every field is mass-assignable **except** the ones listed.

| Approach | Behavior | Recommended For |
|---|---|---|
| `$fillable` | Only listed fields allowed | Most models — safer, explicit |
| `$guarded = []` | Everything allowed | Rare — only when you fully trust the input source |

> 💡 **Tip:** Default to `$fillable` and explicitly list every field. It forces you to consciously decide what user input is allowed to touch — a habit senior developers rely on to avoid entire classes of security bugs.

---

## Updating Records

```php
$product = Product::find(1);
$product->price = 19.99;
$product->save();
```

Or in one step:

```php
Product::where('id', 1)->update(['price' => 19.99]);
```

- The second form updates directly in the database without first loading the model into PHP — slightly more efficient for simple updates.

---

## Deleting Records

```php
$product = Product::find(1);
$product->delete();

// Or directly:
Product::destroy(1);
Product::destroy([1, 2, 3]); // multiple at once

// Delete by query condition, no need to fetch first
Product::where('stock', 0)->delete();
```

---

## Soft Deletes

Sometimes you don't want to truly erase data — just hide it while keeping it recoverable.

**Migration:**

```php
$table->softDeletes(); // adds a nullable deleted_at column
```

**Model:**

```php
use Illuminate\Database\Eloquent\SoftDeletes;

class Product extends Model
{
    use SoftDeletes;
}
```

- **Soft Delete**: instead of removing the row, Eloquent sets its `deleted_at` timestamp. From then on, normal queries automatically exclude it — as if it were gone — while the data still physically exists.

```php
$product->delete();          // sets deleted_at, doesn't remove the row
$product->restore();         // un-deletes it
$product->forceDelete();     // actually removes it permanently
Product::withTrashed()->get(); // include soft-deleted rows in results
```

> 💡 **Tip:** Use soft deletes for anything you might need to recover or audit later (orders, user accounts, posts). Use real `delete()` for genuinely disposable data.

---

## Accessors & Mutators

- **Accessor**: code that transforms an attribute's value automatically whenever you **read** it.
- **Mutator**: code that transforms a value automatically whenever you **set/save** it.

```php
use Illuminate\Database\Eloquent\Casts\Attribute;

class Product extends Model
{
    protected function name(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
            set: fn (string $value) => strtolower($value),
        );
    }
}
```

- Now, `$product->name = 'WIRELESS MOUSE'` automatically stores `"wireless mouse"`, and reading `$product->name` automatically returns `"Wireless mouse"` — the transformation happens transparently, everywhere, with zero extra effort at the call site.

> 💡 **Tip:** Accessors/mutators are ideal for consistent formatting (capitalization, currency formatting, hashing a password automatically on set) without repeating that logic every time you touch the field.

---

## Your First Relationships: One-to-One & One-to-Many

A **relationship** in Eloquent is a method on a Model that tells Laravel how it connects to another Model's table.

### One-to-One

Example: a `User` has exactly one `Profile`.

**Migration** (on the `profiles` table): `$table->foreignId('user_id')->constrained();`

```php
// In User.php
public function profile()
{
    return $this->hasOne(Profile::class);
}

// In Profile.php
public function user()
{
    return $this->belongsTo(User::class);
}
```

- **`hasOne`**: declared on the "parent" side — "this User has one Profile."
- **`belongsTo`**: declared on the "child" side, the table holding the foreign key — "this Profile belongs to a User."

**Using it:**

```php
$user = User::find(1);
echo $user->profile->bio; // Access the related model like a property
```

### One-to-Many

Example: a `Category` has many `Products`.

**Migration** (on the `products` table): `$table->foreignId('category_id')->constrained();`

```php
// In Category.php
public function products()
{
    return $this->hasMany(Product::class);
}

// In Product.php
public function category()
{
    return $this->belongsTo(Category::class);
}
```

**Using it:**

```php
$category = Category::find(1);
foreach ($category->products as $product) {
    echo $product->name;
}

$product = Product::find(1);
echo $product->category->name;
```

| Relationship | Declared On | Method | Foreign Key Lives On |
|---|---|---|---|
| One-to-One | The "owning" side | `hasOne()` | The other table |
| One-to-One (inverse) | The "owned" side | `belongsTo()` | This table |
| One-to-Many | The "one" side | `hasMany()` | The other table |
| One-to-Many (inverse) | The "many" side | `belongsTo()` | This table |

> 💡 **Tip:** Notice `$user->profile` and `$category->products` are accessed **without parentheses** even though they're defined as methods — Eloquent uses PHP "magic properties" to let you treat relationship methods like plain attributes. Call them with parentheses (`$user->profile()`) only when you want to chain more query methods onto them first.

> ⚠️ **Warning:** Looping through relationships without eager loading (e.g., `$category->products` inside a loop over many categories) causes the notorious **N+1 query problem** — one query per category, instead of one combined query. This gets fixed with `with()` in note 07.

---

## Tinker — Laravel's Interactive Console

**Tinker**: an interactive PHP REPL (Read-Eval-Print Loop) bundled with Laravel, letting you run real Eloquent code against your actual database directly from the command line — no browser, no route, no controller needed.

```bash
php artisan tinker
```

Once inside, you get a live PHP prompt with your entire Laravel app already loaded:

```php
>>> Product::count()
=> 12

>>> Product::first()
=> App\Models\Product {#...}

>>> Product::create(['name' => 'Test Item', 'price' => 9.99])

>>> Product::where('price', '>', 50)->get()
```

> 💡 **Tip:** Tinker is one of the most useful daily-driver tools in Laravel. Whenever you're unsure "will this Eloquent query actually work?", test it in Tinker first instead of building a whole route/controller just to find out.

> ⚠️ **Warning:** Tinker operates on your **real** configured database (whatever `.env` currently points to). Be careful running `delete()` or `truncate()` commands in Tinker if you're not 100% sure which database you're connected to.

Exit Tinker any time with:

```bash
exit
```

---

## Quick Revision

- Eloquent is Laravel's ORM — it maps each database table to a Model class, letting you query and manipulate data with PHP methods instead of raw SQL.
- `php artisan make:model Name` creates a Model; flags like `-m`, `-f`, `--all` generate related files (migration, factory, controller) at the same time.
- Eloquent guesses table names, primary keys, and foreign key names by convention (`Product` model → `products` table, `id` primary key) — override only when necessary.
- Retrieve data with `all()`, `get()`, `first()`, `find()`/`findOrFail()`, and chain `where()`, `orderBy()`, `limit()`, `paginate()` to refine queries.
- **Mass assignment** requires declaring `$fillable` (whitelist, recommended) or `$guarded` (blacklist) on a Model — this exists specifically to prevent malicious/unexpected fields from being saved.
- **Soft deletes** (`use SoftDeletes` + `softDeletes()` migration column) hide rows instead of erasing them, recoverable via `restore()`.
- **Accessors/mutators** let you transform a field's value automatically every time it's read or written.
- **`hasOne`/`belongsTo`** define One-to-One relationships; **`hasMany`/`belongsTo`** define One-to-Many — the foreign key always lives on the "belongs to" side.
- Looping over relationships without eager loading causes the **N+1 query problem** — the fix (`with()`) is covered in note 07, along with Many-to-Many and more advanced relationship types.
- **Tinker** (`php artisan tinker`) is your interactive console for testing real Eloquent queries against your actual database without building a route first.
- Next up (note 07): **Eloquent Relationships (Deep-Dive)** — Many-to-Many, Has One/Many Through, polymorphic relationships, and fixing N+1 with eager loading.