# Laravel 07 — Eloquent Relationships Deep-Dive

A relationship in Eloquent is a defined connection between two Models that lets you query related data using natural, chainable PHP instead of writing manual JOIN statements.

## Table of Contents

- [Quick Recap From Note 06](#quick-recap-from-note-06)
- [Many-to-Many Relationships](#many-to-many-relationships)
- [Working With Pivot Data](#working-with-pivot-data)
- [Has One Through](#has-one-through)
- [Has Many Through](#has-many-through)
- [Polymorphic Relationships](#polymorphic-relationships)
- [Many-to-Many Polymorphic Relationships](#many-to-many-polymorphic-relationships)
- [The N+1 Query Problem](#the-n1-query-problem)
- [Eager Loading With `with()`](#eager-loading-with-with)
- [Relationship Types — Full Comparison](#relationship-types--full-comparison)
- [Quick Revision](#quick-revision)

---

## Quick Recap From Note 06

You already know:
- **`hasOne` / `belongsTo`** → One-to-One (e.g., `User` ↔ `Profile`)
- **`hasMany` / `belongsTo`** → One-to-Many (e.g., `Category` → `Products`)

This note covers everything else you'll need for real-world, senior-level schema design.

---

## Many-to-Many Relationships

**Many-to-Many**: many rows in one table relate to many rows in another — e.g., a `Product` can have many `Tags`, and a `Tag` can belong to many `Products`.

Unlike the relationships in note 06, this requires a **pivot table** — an intermediate table that only stores pairs of IDs connecting the two.

**Migration for the pivot table** (named alphabetically, singular, joined with an underscore — Laravel's convention: `product_tag`):

```php
Schema::create('product_tag', function (Blueprint $table) {
    $table->foreignId('product_id')->constrained()->onDelete('cascade');
    $table->foreignId('tag_id')->constrained()->onDelete('cascade');
});
```

**On both Models:**

```php
// In Product.php
public function tags()
{
    return $this->belongsToMany(Tag::class);
}

// In Tag.php
public function products()
{
    return $this->belongsToMany(Product::class);
}
```

- **`belongsToMany`**: used on **both** sides of the relationship — neither side is the "owner"; they're peers connected through the pivot table.
- Laravel automatically guesses the pivot table name (`product_tag`) from the two model names, alphabetically ordered and singular.

**Using it:**

```php
$product = Product::find(1);

$product->tags;                       // Get all tags for this product
$product->tags()->attach($tagId);     // Link a tag to this product
$product->tags()->detach($tagId);     // Unlink a tag
$product->tags()->sync([1, 2, 3]);    // Replace ALL tags with exactly these IDs
```

| Method | Effect |
|---|---|
| `attach($id)` | Adds a new link, keeps existing ones |
| `detach($id)` | Removes a specific link |
| `detach()` | Removes ALL links for this record |
| `sync([...])` | Replaces all links with exactly the given set |
| `toggle([...])` | Attaches IDs not currently linked, detaches ones that are |

> 💡 **Tip:** `sync()` is what you'll use for something like a tag-picker form — it handles adding new tags and removing unchecked ones in a single call, instead of manually diffing old vs. new selections yourself.

---

## Working With Pivot Data

Sometimes the pivot table itself needs extra data — e.g., *when* a tag was attached, or a `quantity` for a product inside an order.

```php
Schema::create('order_product', function (Blueprint $table) {
    $table->foreignId('order_id')->constrained();
    $table->foreignId('product_id')->constrained();
    $table->integer('quantity');
    $table->timestamps();
});
```

```php
public function products()
{
    return $this->belongsToMany(Product::class)
        ->withPivot('quantity')
        ->withTimestamps();
}
```

- **`->withPivot('quantity')`**: tells Eloquent to also pull this extra column when loading the relationship.
- **`->withTimestamps()`**: tells Eloquent the pivot table has its own `created_at`/`updated_at` columns to manage.

**Accessing pivot data:**

```php
foreach ($order->products as $product) {
    echo $product->pivot->quantity;
}
```

- **`->pivot`**: a special property Eloquent attaches to each related model, holding the extra pivot-table columns for that specific pairing.

> 💡 **Tip:** If your "pivot" table starts accumulating a lot of its own columns and business logic (like an `order_product` table with quantity, discount, notes, etc.), consider making it a full Model of its own — Laravel supports this via a custom pivot model class. That's a senior-level refactor worth knowing exists, even if you don't need it on day one.

---

## Has One Through

**Has One Through**: lets a model access a single related record on a **distant** table, by passing through one intermediate table — without needing to manually chain two relationship calls.

Example: a `Mechanic` has one `Car`, and that `Car` has one `Owner`. You want the `Mechanic` to directly access the `Owner`.

```
Mechanic → has one → Car → has one → Owner
```

```php
// In Mechanic.php
public function carOwner()
{
    return $this->hasOneThrough(Owner::class, Car::class);
}
```

- First argument: the **final** model you actually want (`Owner`).
- Second argument: the **intermediate** model connecting them (`Car`).

```php
$owner = $mechanic->carOwner;
```

> 💡 **Tip:** This relationship type is genuinely less common than the others — you'll use it occasionally, mostly for reporting-style lookups that skip a middle layer.

---

## Has Many Through

**Has Many Through**: the same idea as above, but for retrieving **many** distant records instead of one.

The official Laravel example: a `Country` has many `Users`, and each `User` has many `Posts`. You want to fetch every post written by users in a given country — directly from the `Country` model.

```
Country → has many → User → has many → Post
```

```php
// In Country.php
public function posts()
{
    return $this->hasManyThrough(Post::class, User::class);
}
```

```php
$country = Country::find(1);
$posts = $country->posts; // Every post written by any user from this country
```

- Laravel automatically figures out the foreign keys based on convention (`user_id` on `posts`, `country_id` on `users`) — you can override them with additional arguments if your columns don't follow convention.

> ⚠️ **Warning:** `hasManyThrough` only supports **one** level of intermediate table. If you need to go through two or more intermediate tables, you'd need a third-party package or restructure your queries manually — this is an edge case, but good to know it exists before you spend an hour confused about why it "isn't working" for a 3-level chain.

---

## Polymorphic Relationships

**Polymorphic Relationship**: lets a single Model (like `Comment` or `Image`) belong to **more than one type of other Model**, using one shared table instead of duplicating the table per relationship.

Example: both `Post` and `Video` need to support comments.

**Without polymorphism**, you'd need two separate tables: `post_comments` and `video_comments` — duplicated structure, duplicated logic.

**With polymorphism**, one `comments` table handles both:

```php
Schema::create('comments', function (Blueprint $table) {
    $table->id();
    $table->text('body');
    $table->morphs('commentable'); // adds commentable_id + commentable_type columns
    $table->timestamps();
});
```

- **`->morphs('commentable')`**: a shortcut that adds two columns — `commentable_id` (which row) and `commentable_type` (which Model class, e.g. `App\Models\Post`).

```php
// In Comment.php
public function commentable()
{
    return $this->morphTo();
}

// In Post.php
public function comments()
{
    return $this->morphMany(Comment::class, 'commentable');
}

// In Video.php
public function comments()
{
    return $this->morphMany(Comment::class, 'commentable');
}
```

**Using it:**

```php
$post->comments()->create(['body' => 'Great article!']);
$video->comments()->create(['body' => 'Nice video!']);

$comment->commentable; // Returns either the Post or the Video, automatically
```

| Method | Used On | Purpose |
|---|---|---|
| `morphTo()` | The shared/child model (`Comment`) | Points back to whichever parent owns it |
| `morphMany()` | Each parent model (`Post`, `Video`) | Declares "I can have many of these" |
| `morphOne()` | Each parent model | Same idea, but for a single related record |

> 💡 **Tip:** Polymorphic relationships are a genuine senior-level schema design pattern — recognizing "these two features need the exact same child structure" and consolidating them into one polymorphic table (comments, likes, images, tags) instead of duplicating tables is a hallmark of clean database design.

---

## Many-to-Many Polymorphic Relationships

Combines both ideas: many-to-many, but the "owning" side can be more than one type of Model.

Example: both `Post` and `Video` need `Tags`, and each `Tag` can apply to many of both.

```php
Schema::create('taggables', function (Blueprint $table) {
    $table->foreignId('tag_id')->constrained();
    $table->morphs('taggable'); // taggable_id + taggable_type
});
```

```php
// In Tag.php
public function posts()
{
    return $this->morphedByMany(Post::class, 'taggable');
}

public function videos()
{
    return $this->morphedByMany(Video::class, 'taggable');
}

// In Post.php
public function tags()
{
    return $this->morphToMany(Tag::class, 'taggable');
}

// In Video.php
public function tags()
{
    return $this->morphToMany(Tag::class, 'taggable');
}
```

| Direction | Method |
|---|---|
| From the shared side (`Tag` → `Post`/`Video`) | `morphedByMany()` |
| From the owning side (`Post`/`Video` → `Tag`) | `morphToMany()` |

---

## The N+1 Query Problem

Flagged back in note 06 — now let's actually fix it.

```php
$categories = Category::all();

foreach ($categories as $category) {
    echo $category->products->count(); // Runs a NEW query on every single iteration!
}
```

- **N+1 Query Problem**: 1 initial query to get all categories, **plus N more queries** (one per category) to get each one's products — instead of 2 total queries. With 100 categories, that's 101 database queries for something that should take 2.

> ⚠️ **Warning:** N+1 is one of the most common real-world Laravel performance bugs — it often works "fine" in local development with tiny test data, then silently cripples performance in production once real data volume hits. Senior developers actively watch for this pattern.

---

## Eager Loading With `with()`

**Eager Loading**: telling Eloquent up front "I'm also going to need this relationship," so it fetches everything in a couple of efficient combined queries instead of one query per row.

```php
$categories = Category::with('products')->get();

foreach ($categories as $category) {
    echo $category->products->count(); // No extra queries — already loaded!
}
```

- `with('products')` runs just **2** queries total (one for categories, one for all their related products combined) no matter how many categories exist.

**Eager loading multiple relationships, and nested ones:**

```php
$categories = Category::with(['products', 'products.tags'])->get();
```

- `'products.tags'` — dot notation eager-loads a relationship *of* a relationship (categories → products → tags), all still in a small, fixed number of queries.

**Detecting N+1 problems during development:**

```php
use Illuminate\Database\Eloquent\Model;

Model::preventLazyLoading(! app()->isProduction());
```

- Placed in a Service Provider's `boot()` method — this makes Laravel **throw an exception** the moment your code tries to lazy-load a relationship, forcing you to add proper eager loading during development, without ever risking a crash in production (since it's disabled there).

> 💡 **Tip:** Turning on `preventLazyLoading` early in a project is a genuinely senior-level habit — it catches N+1 bugs the moment you introduce them, rather than discovering them later via a slow production dashboard.

---

## Relationship Types — Full Comparison

| Relationship | Method(s) | Real-World Example |
|---|---|---|
| One-to-One | `hasOne` / `belongsTo` | `User` ↔ `Profile` |
| One-to-Many | `hasMany` / `belongsTo` | `Category` → `Products` |
| Many-to-Many | `belongsToMany` (both sides) | `Product` ↔ `Tag` |
| Has One Through | `hasOneThrough` | `Mechanic` → `Car` → `Owner` |
| Has Many Through | `hasManyThrough` | `Country` → `User` → `Post` |
| Polymorphic (one) | `morphOne` / `morphTo` | `Post`/`Video` → `Image` |
| Polymorphic (many) | `morphMany` / `morphTo` | `Post`/`Video` → `Comments` |
| Many-to-Many Polymorphic | `morphToMany` / `morphedByMany` | `Post`/`Video` ↔ `Tags` |

---

## Quick Revision

- **Many-to-Many** (`belongsToMany`) requires a pivot table; use `attach()`, `detach()`, and especially `sync()` to manage the links, and `withPivot()`/`withTimestamps()` when the pivot table holds its own data.
- **Has One/Many Through** (`hasOneThrough`, `hasManyThrough`) let a model reach a distant related record by passing through exactly one intermediate table (e.g., `Country` → `User` → `Post`).
- **Polymorphic relationships** (`morphTo`, `morphMany`, `morphOne`) let one child table (like `comments`) serve multiple different parent Models, avoiding duplicated table structures.
- **Many-to-Many Polymorphic** (`morphToMany`, `morphedByMany`) combines both ideas for things like a shared `Tag` system across multiple content types.
- The **N+1 query problem** happens when you loop over a relationship without eager loading — fix it with `with('relationName')`, or nested with `with(['relation.nestedRelation'])`.
- `Model::preventLazyLoading()` (dev-only) is a senior-level habit that forces you to catch N+1 bugs immediately, instead of discovering them in production.
- You now know every relationship type Eloquent supports — this is genuinely one of the most senior-relevant topics in the entire framework, since schema design mistakes here are expensive to fix later.
- Next up (note 08): **Validation & Form Requests** — making sure data is safe and correct before it ever reaches your Models.