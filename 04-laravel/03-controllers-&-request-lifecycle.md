# Laravel 03 — Controllers & Request Lifecycle

A Controller is a PHP class that groups together the logic for handling related requests, so that logic doesn't have to live directly inside your routes file.

## Table of Contents

- [The Request Lifecycle (Big Picture First)](#the-request-lifecycle-big-picture-first)
- [What a Controller Is](#what-a-controller-is)
- [Creating a Basic Controller](#creating-a-basic-controller)
- [Connecting a Controller to a Route](#connecting-a-controller-to-a-route)
- [Resource Controllers (CRUD in One Command)](#resource-controllers-crud-in-one-command)
- [API Controllers](#api-controllers)
- [Invokable (Single Action) Controllers](#invokable-single-action-controllers)
- [Dependency Injection in Controllers](#dependency-injection-in-controllers)
- [Returning Different Response Types](#returning-different-response-types)
- [Quick Revision](#quick-revision)

---

## The Request Lifecycle (Big Picture First)

Before diving into Controllers, it helps to see where they fit in the full journey of a request — from browser to response.

```
Browser sends request
        ↓
public/index.php (the single entry point for EVERY request)
        ↓
HTTP Kernel boots the app + runs global Middleware
        ↓
Service Providers register the app's core services (DB, auth, etc.)
        ↓
Router matches the URL to a Route
        ↓
Route-specific Middleware runs (e.g., "is user logged in?")
        ↓
Controller method (or route closure) runs your actual logic
        ↓
Controller returns a Response (HTML view, JSON, redirect, etc.)
        ↓
Response sent back to the browser
```

- **Entry point**: every single request to a Laravel app — no matter the URL — physically starts at `public/index.php`. Your web server (Nginx/Apache) is configured to funnel all traffic there.
- **HTTP Kernel**: the class responsible for taking the raw request, running it through the middleware stack, and eventually handing it to the router.
- **Service Providers**: classes (in `app/Providers/`) that "bootstrap" — i.e., register and configure — every major feature of the framework (database, routing, validation, etc.) before your app can handle any request.

> 💡 **Tip:** You don't need to memorize this diagram. Just remember the order: **Request → Middleware → Router → Controller → Response**. Everything else is internal plumbing.

---

## What a Controller Is

- **Controller**: a PHP class living in `app/Http/Controllers/` whose methods each handle one specific action (show a list, save a new record, delete a record, etc.).
- Without controllers, all your logic would sit inside `routes/web.php` as closures — fine for a 2-route toy app, unreadable for anything real.

| Without a Controller | With a Controller |
|---|---|
| Logic written inline in `routes/web.php` | Logic organized into named methods in a dedicated class |
| Hard to reuse or test | Easy to reuse, test, and navigate |
| Fine for tiny demos only | Standard for any real Laravel app |

---

## Creating a Basic Controller

```bash
php artisan make:controller ProductController
```

This generates `app/Http/Controllers/ProductController.php`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ProductController extends Controller
{
    //
}
```

- `namespace App\Http\Controllers;` — tells PHP where this class "lives" so it can be found and imported elsewhere with `use`.
- `extends Controller` — inherits shared helper functionality from Laravel's base Controller class.
- The class starts empty — you now add your own methods to it.

**Adding a method:**

```php
class ProductController extends Controller
{
    public function index()
    {
        return 'This will show all products';
    }
}
```

> ⚠️ **Warning:** Controller class names should be **StudlyCase / PascalCase** and singular-plus-"Controller" (e.g., `ProductController`, not `productcontroller` or `ProductsControllers`). Laravel's generators expect this convention, and breaking it causes confusing autoload errors.

---

## Connecting a Controller to a Route

Recall from note 02:

```php
use App\Http\Controllers\ProductController;

Route::get('/products', [ProductController::class, 'index']);
```

- `ProductController::class` — resolves to the fully qualified class name as a string, so Laravel knows which file/class to load.
- `'index'` — the exact method name inside that class to run when this route is hit.

---

## Resource Controllers (CRUD in One Command)

Almost every real feature (products, posts, users, orders...) needs the same five operations: list, view one, create, update, delete. Laravel calls this pattern **CRUD** (Create, Read, Update, Delete), and gives you a shortcut to scaffold it instantly.

```bash
php artisan make:controller ProductController --resource
```

This generates **seven pre-named empty methods** instead of zero:

| Method | HTTP Verb | URL (example) | Purpose |
|---|---|---|---|
| `index` | GET | `/products` | Show a list of all products |
| `create` | GET | `/products/create` | Show the "add new product" form |
| `store` | POST | `/products` | Save a newly submitted product |
| `show` | GET | `/products/{id}` | Show one specific product |
| `edit` | GET | `/products/{id}/edit` | Show the "edit product" form |
| `update` | PUT/PATCH | `/products/{id}` | Save changes to a product |
| `destroy` | DELETE | `/products/{id}` | Delete a product |

**Register all seven routes with a single line** (instead of writing seven `Route::` lines by hand):

```php
use App\Http\Controllers\ProductController;

Route::resource('products', ProductController::class);
```

**Generate a controller already wired to a specific Model:**

```bash
php artisan make:controller ProductController --resource --model=Product
```

- `--model=Product` — pre-fills each generated method with the `Product` model already type-hinted where relevant (via Route Model Binding, from note 02), saving you from writing those imports/type-hints by hand.

> 💡 **Tip:** `create` and `edit` only make sense for a traditional server-rendered app (they return HTML forms). If you're building a pure API, skip them — see the next section.

---

## API Controllers

Since a JSON API has no HTML "create form" or "edit form" pages to show, Laravel gives you a leaner version of the resource controller for APIs — 5 methods instead of 7 (no `create`, no `edit`):

```bash
php artisan make:controller Api/ProductController --api --model=Product
```

- `--api` — skips `create` and `edit` since they're not needed when you're only returning JSON.
- `Api/ProductController` — placing it inside an `Api/` subfolder (creating `app/Http/Controllers/Api/ProductController.php`) is a common convention to separate API controllers from web ones.

Register it with `apiResource` instead of `resource`:

```php
Route::apiResource('products', ProductController::class);
```

| Controller Type | Command Flag | Methods Generated | Used For |
|---|---|---|---|
| Plain | *(no flag)* | 0 (empty) | Fully custom logic |
| Resource | `--resource` | 7 (incl. `create`, `edit`) | Server-rendered HTML CRUD |
| API | `--api` | 5 (no `create`/`edit`) | JSON APIs |
| Invokable | `--invokable` | 1 (`__invoke`) | A controller that does exactly one thing |

> 💡 **Tip:** For your end-of-series project (a React/Next.js frontend talking to a Laravel API), `--api` resource controllers plus `apiResource` routes will be your main building block — full deep-dive comes in note 12.

---

## Invokable (Single Action) Controllers

Sometimes a controller only ever needs to do **one thing** — no need for 7 methods.

```bash
php artisan make:controller GenerateReportController --invokable
```

```php
class GenerateReportController extends Controller
{
    public function __invoke(Request $request)
    {
        // does the one thing this controller exists for
    }
}
```

- **`__invoke`**: a special PHP "magic method" — when a class has it, you can call an *instance* of that class as if it were a function itself. Laravel takes advantage of this so the route doesn't need to specify a method name at all.

```php
Route::get('/report', GenerateReportController::class);
```

---

## Dependency Injection in Controllers

You can type-hint classes in a controller method's parameters, and Laravel automatically supplies (**"injects"**) an instance for you — no manual instantiation needed.

```php
use Illuminate\Http\Request;

public function store(Request $request)
{
    $name = $request->input('name');
    // ...
}
```

- **`Request $request`**: Laravel automatically injects an object representing the current incoming HTTP request — containing form data, query parameters, headers, uploaded files, etc.
- **Dependency Injection**: a design pattern where a class receives the objects it needs (its "dependencies") from an outside source, rather than creating them itself.

This also works combined with Route Model Binding from note 02:

```php
public function show(Product $product)
{
    return $product; // Laravel already fetched this row from the DB for you
}
```

> ⚠️ **Warning:** Never trust raw request data blindly — always validate incoming input before saving it. Validation is covered fully in note 08, but keep this in mind from day one.

---

## Returning Different Response Types

A controller method's **return value** is what actually gets sent back to the browser/client.

```php
// Return a Blade view (HTML)
return view('products.index', ['products' => $products]);

// Return JSON (common for APIs)
return response()->json(['message' => 'Product created successfully']);

// Redirect the user elsewhere
return redirect()->route('products.index');
```

| Return Type | When to Use |
|---|---|
| `view(...)` | Server-rendered HTML pages (Blade — note 04) |
| `response()->json(...)` | API endpoints returning data to a JS frontend |
| `redirect()->...` | After a form submission, send the user somewhere else |

---

## Quick Revision

- Every request starts at `public/index.php`, passes through the **HTTP Kernel** and **Middleware**, gets matched to a **Route**, and finally reaches a **Controller** method, which returns a **Response**.
- A **Controller** is just a class that groups related request-handling methods, keeping logic out of your routes file.
- Create one with `php artisan make:controller Name`; add `--resource` to instantly scaffold the 7 standard CRUD methods (`index`, `create`, `store`, `show`, `edit`, `update`, `destroy`).
- Use `--api` instead of `--resource` for JSON-only controllers (skips `create`/`edit`), paired with `Route::apiResource(...)`.
- `--model=ModelName` pre-wires a generated controller to a specific Eloquent Model.
- `--invokable` generates a controller with a single `__invoke()` method for one-off actions.
- Laravel automatically injects dependencies (like the current `Request`, or a Model via Route Model Binding) into your controller method parameters — you rarely instantiate these yourself.
- A controller method can return a Blade `view()`, a `response()->json()`, or a `redirect()` — the shape of your return value determines what the client receives.
- Next up (note 04): **Blade Templating** — how views actually render dynamic HTML.