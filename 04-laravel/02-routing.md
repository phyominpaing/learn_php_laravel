# Laravel 02 — Routing

Routing is the layer in Laravel that decides which piece of code runs when a specific URL is visited.

## Table of Contents

- [What a Route Actually Is](#what-a-route-actually-is)
- [Basic Route Syntax](#basic-route-syntax)
- [`web.php` vs `api.php`](#webphp-vs-apiphp)
- [HTTP Verbs Laravel Understands](#http-verbs-laravel-understands)
- [Route Parameters](#route-parameters)
- [Named Routes](#named-routes)
- [Route Groups](#route-groups)
- [Route Model Binding](#route-model-binding)
- [Fallback Routes & 404s](#fallback-routes--404s)
- [Rate Limiting Routes](#rate-limiting-routes)
- [Viewing & Caching Routes (Artisan)](#viewing--caching-routes-artisan)
- [Quick Revision](#quick-revision)

---

## What a Route Actually Is

- **Route**: a rule that maps a URL pattern + HTTP method to a piece of code (a closure or a Controller method).
- **Closure**: an anonymous inline function — fine for tiny routes, but for real apps you'll point routes to Controllers (note 03) instead of writing logic directly here.

```php
use Illuminate\Support\Facades\Route;

Route::get('/greeting', function () {
    return 'Hello World';
});
```

- Visiting `http://yourapp.test/greeting` in a browser triggers this closure and prints `Hello World`.

> 💡 **Tip:** Think of `routes/web.php` as the master "switchboard" — every incoming browser request checks this file (in order) to find a matching rule.

---

## Basic Route Syntax

Pointing a route to a **Controller** instead of a closure (the pattern you'll actually use in real projects):

```php
use App\Http\Controllers\UserController;

Route::get('/user', [UserController::class, 'index']);
```

- `UserController::class` — the Controller class to use.
- `'index'` — the specific method inside that Controller to call.
- This behaves identically to the closure example above, just organized properly — the actual logic lives in `app/Http/Controllers/UserController.php` instead of cluttering the routes file.

---

## `web.php` vs `api.php`

Laravel automatically loads every file inside the `routes/` directory. Routes defined in `routes/web.php` are treated as your web interface — they're wrapped with the `web` middleware group, which gives you session handling and CSRF protection for free.

If your app also needs to expose an API, you turn it on with an Artisan command:

```bash
php artisan install:api
```

This installs **Laravel Sanctum** (a lightweight token-based API authentication system) and creates a fresh `routes/api.php` file for you. Routes in that file are stateless (no session/cookies involved) and automatically get an `/api` prefix added to their URLs.

| File | Purpose | Middleware Group | Typical Response |
|---|---|---|---|
| `routes/web.php` | Browser-facing pages | `web` (sessions, CSRF) | HTML (Blade views) |
| `routes/api.php` | API endpoints | `api` (stateless) | JSON |

> 💡 **Tip:** As a frontend dev, if you're building a React/Next.js frontend that talks to Laravel purely as a backend, you'll live almost entirely in `routes/api.php` — covered in depth in note 12.

---

## HTTP Verbs Laravel Understands

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

The router can register a route for any HTTP verb individually, or you can make one route respond to several verbs at once with `match(['get', 'post'], ...)`, or to literally every verb using `any(...)`.

| Verb | Typical Use |
|---|---|
| `GET` | Retrieve/display data (viewing a page, fetching a list) |
| `POST` | Create new data (submitting a form) |
| `PUT` / `PATCH` | Update existing data |
| `DELETE` | Remove data |

> ⚠️ **Warning:** Plain HTML forms can only natively send `GET` or `POST`. To submit a `PUT`, `PATCH`, or `DELETE` request from an HTML form, you need a hidden `_method` input field — Laravel gives you the `@method('PUT')` Blade directive to generate it automatically.

---

## Route Parameters

**Required parameter:**

```php
Route::get('/user/{id}', function (string $id) {
    return 'User '.$id;
});
```

- `{id}` — a placeholder segment of the URL. Visiting `/user/5` makes `$id` equal `"5"`.
- Parameter names may only contain letters and underscores, and multiple parameters are matched into your function by their **order**, not their variable name.

**Optional parameter** (needs a `?` and a default value):

```php
Route::get('/user/{name?}', function (?string $name = 'John') {
    return $name;
});
```

**Constraining the format with `where`:**

```php
Route::get('/user/{id}', function (string $id) {
    // ...
})->where('id', '[0-9]+');
```

- This forces `{id}` to only match digits. If the incoming URL doesn't satisfy the constraint, Laravel automatically returns a 404 response instead of running the route.

Laravel also ships convenience shortcuts so you don't have to write raw regex every time:

```php
Route::get('/user/{id}', function (string $id) {
    // ...
})->whereNumber('id');

Route::get('/user/{name}', function (string $name) {
    // ...
})->whereAlpha('name');
```

> 💡 **Tip:** Use `whereNumber()`, `whereAlpha()`, `whereUuid()` etc. instead of memorizing regex — they read better and are less error-prone.

---

## Named Routes

Naming a route lets you refer to it anywhere in your app without hardcoding the URL string.

```php
Route::get('/user/profile', function () {
    // ...
})->name('profile');
```

Then, generate a link to it from anywhere:

```php
$url = route('profile');
return redirect()->route('profile');
```

- **Named route**: a route with a unique string identifier (`->name('profile')`) used to generate its URL dynamically.
- If the route has parameters, pass them as an array: `route('profile', ['id' => 1])`.

> 💡 **Tip:** Always prefer named routes over hardcoded URL strings in your Blade views and redirects. If you ever change the URL path itself, every `route('name')` call still works — nothing breaks.

---

## Route Groups

**Route Group**: a way to apply shared settings (middleware, a URL prefix, a name prefix) to many routes at once instead of repeating yourself.

**Grouping by middleware:**

```php
Route::middleware(['auth'])->group(function () {
    Route::get('/dashboard', function () {
        // Only accessible to logged-in users
    });
});
```

**Grouping by shared controller:**

```php
use App\Http\Controllers\OrderController;

Route::controller(OrderController::class)->group(function () {
    Route::get('/orders/{id}', 'show');
    Route::post('/orders', 'store');
});
```

**Grouping by URL prefix + name prefix** (very common for admin sections):

```php
Route::prefix('admin')->name('admin.')->group(function () {
    Route::get('/users', function () {
        // Matches "/admin/users", route name becomes "admin.users"
    })->name('users');
});
```

| Group Feature | Method | Effect |
|---|---|---|
| Shared middleware | `->middleware([...])` | Applies auth/logic checks to all routes in the group |
| Shared controller | `->controller(X::class)` | Avoids repeating the controller class name per route |
| URL prefix | `->prefix('admin')` | Adds `/admin` before every route path in the group |
| Name prefix | `->name('admin.')` | Adds `admin.` before every route name in the group |

---

## Route Model Binding

**Route Model Binding**: Laravel automatically fetches the matching database row (as a Model instance) based on an ID in the URL — so you don't manually write `User::find($id)` in every controller method.

```php
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
```

- Because `$user` is type-hinted as the `User` Eloquent model, and its variable name matches the `{user}` URL segment, Laravel automatically looks up and injects the matching row from the database. If nothing matches, it automatically responds with a 404 — you don't write that check yourself.

This also works with a column other than `id`:

```php
use App\Models\Post;

Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});
```

> 💡 **Tip:** This is one of Laravel's biggest time-savers. Instead of manually looking up records by ID in every controller, just type-hint the Model and Laravel does the lookup (and the 404-if-missing) automatically.

---

## Fallback Routes & 404s

```php
Route::fallback(function () {
    // Runs when no other route matches
});
```

- **Fallback Route**: a catch-all route that only runs if nothing else matched — useful for a custom "page not found" response.

---

## Rate Limiting Routes

**Rate Limiting**: restricting how many times a client can hit a route in a given time window, to prevent abuse (e.g., brute-force login attempts).

```php
Route::middleware(['throttle:uploads'])->group(function () {
    Route::post('/audio', function () {
        // ...
    });
});
```

If a client exceeds the configured limit, Laravel automatically responds with an HTTP `429 Too Many Requests` status — you don't have to write that logic yourself.

> ⚠️ **Warning:** Always rate-limit sensitive endpoints like login and password reset forms — without it, your app is wide open to brute-force attacks.

---

## Viewing & Caching Routes (Artisan)

This is the artisan command you asked about — here's the full breakdown.

**See every route registered in your app:**

```bash
php artisan route:list
```

- Prints a table of every route: its HTTP method, URL pattern, name, and the controller/action it points to.
- Extremely useful for debugging "why isn't this URL working?"

**Useful flags:**

```bash
# Show middleware assigned to each route
php artisan route:list -v

# Also expand middleware GROUPS into individual middleware
php artisan route:list -vv

# Only show routes starting with a given path
php artisan route:list --path=api

# Hide routes defined by third-party packages
php artisan route:list --except-vendor

# Show ONLY routes defined by third-party packages
php artisan route:list --only-vendor
```

**Route caching (for production, not local dev):**

```bash
php artisan route:cache
```

- Compiles all your routes into a single optimized file so Laravel doesn't have to re-process `web.php`/`api.php` on every request — this drastically cuts down the time it takes to register all your app's routes on each request.

```bash
php artisan route:clear
```

- Clears that cache. **You must run this any time you add or change a route while the cache is active**, or your new route won't be recognized.

> ⚠️ **Warning:** Route caching should only be run during deployment, and you must regenerate the cache every time your routes change — this trips up a lot of beginners who cache routes locally, then can't figure out why their brand-new route "doesn't exist."

| Command | Purpose |
|---|---|
| `php artisan route:list` | View all registered routes |
| `php artisan route:list -v` | Include middleware info |
| `php artisan route:list --path=api` | Filter routes by path |
| `php artisan route:cache` | Cache routes for production performance |
| `php artisan route:clear` | Clear the route cache |

---

## Quick Revision

- A **route** connects a URL pattern + HTTP verb to code — either a closure (quick tests only) or a Controller method (real apps).
- `routes/web.php` is for browser pages (sessions, CSRF protection); `routes/api.php` is for stateless JSON APIs, enabled via `php artisan install:api`.
- **Route parameters** (`{id}`) capture URL segments; use `where()` or shortcuts like `whereNumber()` to constrain their format.
- **Named routes** (`->name('profile')`) let you generate URLs without hardcoding paths — always prefer these.
- **Route groups** let you share middleware, controllers, and prefixes across many routes at once.
- **Route Model Binding** automatically fetches the matching database record and injects it into your route/controller — a huge time-saver that auto-404s if the record isn't found.
- `php artisan route:list` (with `-v`, `--path=`, etc.) is your go-to debugging tool to see what routes actually exist.
- `php artisan route:cache` speeds up production — but you must run `route:clear` (or re-cache) after every route change, or updates won't take effect.
- Next up (note 03): **Controllers** — where your actual route logic should live, including `php artisan make:controller`.