# Laravel 12 — API Development

An API (Application Programming Interface) in this context is a set of URLs your Laravel backend exposes that return JSON data instead of HTML, so a separate frontend (like your React/Next.js app) can talk to it.

## Table of Contents

- [Why This Note Ties the Whole Series Together](#why-this-note-ties-the-whole-series-together)
- [REST — The Convention Behind Most APIs](#rest--the-convention-behind-most-apis)
- [Setting Up API Routes](#setting-up-api-routes)
- [Building an API Controller](#building-an-api-controller)
- [Why You Shouldn't Return Models Directly](#why-you-shouldnt-return-models-directly)
- [API Resources — Transforming Models Into JSON](#api-resources--transforming-models-into-json)
- [Resource Collections](#resource-collections)
- [Conditional Fields With `when()`](#conditional-fields-with-when)
- [Including Relationships in a Resource](#including-relationships-in-a-resource)
- [Pagination in APIs](#pagination-in-apis)
- [Authenticating the API With Sanctum](#authenticating-the-api-with-sanctum)
- [A Full Worked Example: Login → Token → Protected Request](#a-full-worked-example-login--token--protected-request)
- [Consistent Error Responses](#consistent-error-responses)
- [CORS — Letting Your Frontend Actually Reach the API](#cors--letting-your-frontend-actually-reach-the-api)
- [API Versioning](#api-versioning)
- [Rate Limiting Your API](#rate-limiting-your-api)
- [Quick Revision](#quick-revision)

---

## Why This Note Ties the Whole Series Together

Every previous note contributes directly to building a real API:

| From Note | Used For |
|---|---|
| 02 — Routing | `routes/api.php`, `Route::apiResource()` |
| 03 — Controllers | `--api` resource controllers |
| 05/06 — Database & Eloquent | The actual data being served |
| 08 — Validation | Validating incoming API requests |
| 09 — Auth | Sanctum token authentication |
| 10 — Authorization | Policies controlling who can do what via the API |
| 11 — File Storage | Handling image/file uploads through the API |

This note is where all of it comes together into working, real endpoints.

---

## REST — The Convention Behind Most APIs

- **REST (Representational State Transfer)**: a widely-used convention for designing APIs, where each URL represents a "resource" (a noun, like `articles`), and the HTTP verb (from note 02) determines the action taken on it.

| HTTP Method | Endpoint | Action | Success Status |
|---|---|---|---|
| GET | `/api/articles` | List all articles | 200 OK |
| GET | `/api/articles/{id}` | Get one specific article | 200 OK |
| POST | `/api/articles` | Create a new article | 201 Created |
| PUT/PATCH | `/api/articles/{id}` | Update an article | 200 OK |
| DELETE | `/api/articles/{id}` | Delete an article | 204 No Content |

- **Resource names are plural nouns** (`/articles`, not `/article` or `/getArticles`) — this is a strong, widely-followed convention, not a hard rule, but breaking it makes your API feel unfamiliar to other developers.

**Key HTTP status codes to know:**

| Code | Meaning | When |
|---|---|---|
| `200 OK` | Success | Successful GET/PUT/PATCH |
| `201 Created` | Success, new resource created | Successful POST |
| `204 No Content` | Success, nothing to return | Successful DELETE |
| `401 Unauthorized` | Not authenticated | Missing/invalid token |
| `403 Forbidden` | Authenticated, but not allowed | Failed a Policy check (note 10) |
| `404 Not Found` | Resource doesn't exist | Bad ID, wrong URL |
| `422 Unprocessable Entity` | Validation failed | Bad input data (note 08) |
| `429 Too Many Requests` | Rate limited | Too many requests too fast (note 02) |
| `500 Internal Server Error` | Server-side bug | Unhandled exception |

> 💡 **Tip:** Using the *correct* status code (not just always returning `200` with an `"error": true` field buried in the JSON body) is one of the clearest signals of a well-built, senior-quality API — frontend code, monitoring tools, and other developers all rely on status codes to behave correctly automatically.

---

## Setting Up API Routes

From note 02, enable the API routes file if you haven't already:

```bash
php artisan install:api
```

This creates `routes/api.php` and installs Sanctum. Every route here is automatically prefixed with `/api`.

```php
use App\Http\Controllers\Api\ArticleController;

Route::apiResource('articles', ArticleController::class);
```

Recall from note 03 — `apiResource` registers exactly 5 routes (no `create`/`edit`, since there's no HTML form to show in an API):

| Method | URL | Controller Method |
|---|---|---|
| GET | `/api/articles` | `index` |
| POST | `/api/articles` | `store` |
| GET | `/api/articles/{article}` | `show` |
| PUT/PATCH | `/api/articles/{article}` | `update` |
| DELETE | `/api/articles/{article}` | `destroy` |

---

## Building an API Controller

```bash
php artisan make:controller Api/ArticleController --api --model=Article
```

```php
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Article;
use Illuminate\Http\Request;

class ArticleController extends Controller
{
    public function index()
    {
        return Article::latest()->paginate(15);
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|max:255',
            'body' => 'required',
        ]);

        $article = Article::create($validated);

        return response()->json($article, 201);
    }

    public function show(Article $article)
    {
        return $article;
    }

    public function update(Request $request, Article $article)
    {
        $validated = $request->validate([
            'title' => 'sometimes|required|max:255',
            'body' => 'sometimes|required',
        ]);

        $article->update($validated);

        return $article;
    }

    public function destroy(Article $article)
    {
        $article->delete();

        return response()->json(null, 204);
    }
}
```

- **`return $article;`**: Laravel automatically converts an Eloquent Model (or Collection) returned from a controller into a JSON response — you don't need to manually call `response()->json()` every time, though doing so explicitly (as in `store`) lets you control the status code.
- **`'sometimes|required'`** (in `update`): only validates the field **if it was actually sent** in the request — appropriate for a `PATCH`-style partial update where the client might only send the fields they're changing.

---

## Why You Shouldn't Return Models Directly

The controller above returns raw Models for simplicity, but this has real problems in production code:

- **Over-exposure**: returning `$article` directly sends **every** column in the database table — including things like internal flags, timestamps you don't want exposed, or (worse, on a `User` model) the hashed password field.
- **No control over shape**: if your frontend needs `author_name` but your database column is `user_id`, a raw Model can't bridge that gap.
- **Tight coupling**: if you rename a database column, every API consumer's code breaks immediately, since the JSON structure was a direct mirror of your database structure.

> ⚠️ **Warning:** This is a genuine, common real-world security mistake — returning `User::all()` directly from an endpoint, without hiding the `password` and `remember_token` columns, is a classic beginner vulnerability. **API Resources** (next section) exist specifically to solve this.

---

## API Resources — Transforming Models Into JSON

- **API Resource**: a dedicated class that sits between your Eloquent Model and the JSON actually sent to the client, giving you full, explicit control over exactly what fields appear and how they're formatted.

```bash
php artisan make:resource ArticleResource
```

This generates `app/Http/Resources/ArticleResource.php`:

```php
namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class ArticleResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'excerpt' => str($this->body)->limit(150),
            'published' => (bool) $this->published_at,
            'created_at' => $this->created_at->toDateTimeString(),
        ];
    }
}
```

- **`toArray()`**: defines exactly what the final JSON output looks like — a deliberate, explicit list, not "everything in the database row."
- **`$this->id`, `$this->title`**: inside a Resource, `$this` refers to the underlying Model instance being transformed — you can read any of its real attributes or relationships.
- Notice `excerpt` isn't even a real database column — Resources let you compute and shape fields freely, not just rename existing ones.

**Using it in a controller:**

```php
public function show(Article $article)
{
    return new ArticleResource($article);
}
```

> 💡 **Tip:** From this point forward, "API development" essentially means "Controllers do the querying, Resources do the formatting" — a clean separation that keeps your JSON output stable even if your database structure changes later.

---

## Resource Collections

For returning **multiple** records (like from `index()`), wrap them so the JSON includes proper structure for a list, not just a single item:

```php
public function index()
{
    return ArticleResource::collection(Article::latest()->paginate(15));
}
```

- **`::collection()`**: a static method available on every Resource class — automatically wraps each individual Model in the collection through your `toArray()` logic.

**Example JSON output:**

```json
{
    "data": [
        { "id": 1, "title": "First Post", "excerpt": "...", "published": true, "created_at": "2026-01-15 10:00:00" },
        { "id": 2, "title": "Second Post", "excerpt": "...", "published": false, "created_at": "2026-01-16 09:30:00" }
    ],
    "links": { "first": "...", "last": "...", "next": null, "prev": null },
    "meta": { "current_page": 1, "total": 2, "per_page": 15 }
}
```

- Notice the automatic top-level **`"data"`** wrapper — this is a deliberate Laravel convention, and the `links`/`meta` blocks appear automatically when the underlying query is paginated.

> 💡 **Tip:** For a dedicated collection class with custom collection-level metadata beyond the automatic pagination info, generate one explicitly with `php artisan make:resource ArticleCollection --collection`.

---

## Conditional Fields With `when()`

Sometimes a field should only appear under certain conditions — e.g., showing an article's view count only to its own author.

```php
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'title' => $this->title,
        'view_count' => $this->when(
            $request->user()?->id === $this->user_id,
            $this->view_count
        ),
    ];
}
```

- **`$this->when($condition, $value)`**: includes the key in the final JSON only if `$condition` is `true` — otherwise, the key is **omitted entirely** from the response (not included as `null`).

---

## Including Relationships in a Resource

```php
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'title' => $this->title,
        'author' => new UserResource($this->whenLoaded('user')),
    ];
}
```

- **`whenLoaded('user')`**: only includes the related `user` data if that relationship was already **eager loaded** (note 07) on the query — preventing an accidental N+1 query problem from being triggered just by building the JSON response.

**In the controller, you'd eager load it explicitly:**

```php
public function index()
{
    return ArticleResource::collection(
        Article::with('user')->latest()->paginate(15)
    );
}
```

> ⚠️ **Warning:** If you reference `$this->user` directly in a Resource **without** `whenLoaded()` and without eager loading it in the controller, Eloquent will lazy-load it — silently re-introducing the N+1 problem from note 07, just hidden one layer deeper inside your Resource class instead of your controller.

---

## Pagination in APIs

Recall `->paginate()` from note 06 — it works seamlessly with Resources, as shown above. On the client side, the response's `meta`/`links` blocks tell your frontend everything it needs to build "next page" controls without any extra endpoints.

```php
Article::paginate(15);       // Page-number based (page=1, page=2...)
Article::simplePaginate(15); // Lighter — only knows "is there a next page", no total count
Article::cursorPaginate(15); // Cursor-based — better performance on very large tables
```

| Method | Knows Total Count? | Best For |
|---|---|---|
| `paginate()` | Yes | Most apps — shows "Page 3 of 20" |
| `simplePaginate()` | No | Simple "Load More" buttons, slightly faster |
| `cursorPaginate()` | No | Very large datasets, infinite scroll, best raw performance |

---

## Authenticating the API With Sanctum

Recall Sanctum's concepts from note 09 — here's the full, hands-on setup for token-based auth (the mobile-app/separate-frontend use case).

**1. Issue a token on login:**

```php
// app/Http/Controllers/Api/AuthController.php
use Illuminate\Support\Facades\Auth;

public function login(Request $request)
{
    $credentials = $request->validate([
        'email' => 'required|email',
        'password' => 'required',
    ]);

    if (! Auth::attempt($credentials)) {
        return response()->json(['message' => 'Invalid credentials'], 401);
    }

    $user = Auth::user();
    $token = $user->createToken('api-token')->plainTextToken;

    return response()->json([
        'user' => $user,
        'token' => $token,
    ]);
}
```

**2. Protect routes with the `auth:sanctum` middleware:**

```php
Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('articles', ArticleController::class);
    Route::get('/user', fn (Request $request) => $request->user());
});
```

**3. The frontend sends the token on every subsequent request:**

```
Authorization: Bearer 1|abcdef123456...
```

**4. Logging out (revoking the current token):**

```php
public function logout(Request $request)
{
    $request->user()->currentAccessToken()->delete();

    return response()->json(['message' => 'Logged out']);
}
```

---

## A Full Worked Example: Login → Token → Protected Request

Putting it together, here's what a frontend developer consuming your API would actually do:

```javascript
// 1. Log in and receive a token
const loginRes = await fetch('https://yourapp.test/api/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
});
const { token } = await loginRes.json();

// 2. Use the token on every subsequent request
const articlesRes = await fetch('https://yourapp.test/api/articles', {
    headers: { Authorization: `Bearer ${token}` },
});
const articles = await articlesRes.json();
```

> 💡 **Tip:** This exact flow — login endpoint issues a token, frontend stores it, frontend attaches it as a Bearer header on every request — is precisely what you'll implement in the real-world project at the end of this series.

---

## Consistent Error Responses

A senior-level API returns errors in a **predictable, consistent shape**, so frontend code can handle them generically instead of guessing per-endpoint.

**Validation errors** (automatic, from note 08 — a `422` with this shape by default):

```json
{
    "message": "The title field is required.",
    "errors": {
        "title": ["The title field is required."]
    }
}
```

**A custom error response, matching that same convention:**

```php
return response()->json([
    'message' => 'Article not found.',
], 404);
```

**Global exception handling** (in `bootstrap/app.php`, modern Laravel):

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (ModelNotFoundException $e, Request $request) {
        if ($request->is('api/*')) {
            return response()->json(['message' => 'Resource not found.'], 404);
        }
    });
})
```

- This intercepts a specific exception type globally and formats it consistently as JSON, **only** for API requests — regular web routes still get Laravel's normal HTML error pages.

> 💡 **Tip:** Deciding on one consistent error JSON shape (`{ "message": "...", "errors": {...} }`) early in a project, and sticking to it everywhere, saves enormous frontend debugging time later — this is a real architectural decision senior developers make deliberately, not an afterthought.

---

## CORS — Letting Your Frontend Actually Reach the API

- **CORS (Cross-Origin Resource Sharing)**: a browser security mechanism that, by default, **blocks** a webpage on one domain (`localhost:3000`, your React app) from making requests to a different domain (`localhost:8000`, your Laravel API) — unless the API explicitly allows it.

**Configuration in `config/cors.php`:**

```php
'paths' => ['api/*'],
'allowed_methods' => ['*'],
'allowed_origins' => ['http://localhost:3000', 'https://your-frontend-domain.com'],
'allowed_headers' => ['*'],
'supports_credentials' => true,
```

- **`allowed_origins`**: the exact list of frontend domains permitted to call this API. Attempting a request from any other origin is silently blocked by the **browser itself** (not even your server sees it) — this is client-side enforcement, not something your PHP code can bypass or control after the fact.
- **`supports_credentials`**: must be `true` if you're using Sanctum's cookie-based SPA authentication (from note 09) rather than Bearer tokens.

> ⚠️ **Warning:** A misconfigured `allowed_origins` is one of the single most common "why can't my frontend talk to my API" beginner problems — the request often fails completely silently in the Network tab with a vague CORS error in the browser console, and no error at all reaches your Laravel logs, since Laravel never even received the request.

---

## API Versioning

As your API evolves, breaking changes (renaming a field, changing a response shape) will eventually happen — versioning lets you make those changes without breaking every existing frontend/mobile client overnight.

```php
// routes/api.php
Route::prefix('v1')->group(function () {
    Route::apiResource('articles', Api\V1\ArticleController::class);
});

Route::prefix('v2')->group(function () {
    Route::apiResource('articles', Api\V2\ArticleController::class);
});
```

- Resulting URLs: `/api/v1/articles` and `/api/v2/articles` — old clients keep working against `v1` indefinitely while new clients adopt `v2`.

> 💡 **Tip:** You don't need to over-engineer versioning from day one on a small personal project — but understanding *why* it exists (and the simple URL-prefix pattern above) is genuinely expected senior-level knowledge for any API that has real external consumers.

---

## Rate Limiting Your API

Recall `throttle` middleware from note 02 and note 09 — APIs need this more than web apps do, since they're easier to hit programmatically at high volume.

```php
Route::middleware(['auth:sanctum', 'throttle:api'])->group(function () {
    Route::apiResource('articles', ArticleController::class);
});
```

- The `api` throttle group is pre-configured in Laravel (default: 60 requests/minute per user) — customize it in `bootstrap/app.php` or a Service Provider if needed.

---

## Quick Revision

- REST conventions map HTTP verbs to actions on plural-noun resource URLs (`GET /api/articles`, `POST /api/articles`); use the **correct HTTP status code** for every response, not just always `200`.
- `php artisan install:api` sets up `routes/api.php` and Sanctum; `Route::apiResource()` registers the 5 standard JSON CRUD routes.
- **Never return raw Eloquent Models directly** in a real API — it over-exposes database columns (including sensitive ones) and tightly couples your JSON shape to your database schema.
- **API Resources** (`php artisan make:resource`) solve this: a `toArray()` method gives you full, explicit control over the JSON shape, computed fields, conditional fields (`when()`), and safe relationship inclusion (`whenLoaded()`, paired with eager loading in the controller to avoid N+1).
- `Resource::collection()` wraps lists properly, automatically including pagination `meta`/`links` when combined with `paginate()`.
- **Sanctum** token auth: issue a token on login (`createToken()->plainTextToken`), protect routes with `auth:sanctum` middleware, client sends it back as `Authorization: Bearer <token>`, revoke with `currentAccessToken()->delete()`.
- Keep error responses in one **consistent JSON shape** across your whole API — handle this globally via exception rendering rather than ad-hoc per controller.
- **CORS** (`config/cors.php`, `allowed_origins`) is a browser-enforced security check that must explicitly allow your frontend's domain, or requests silently fail before even reaching Laravel — the most common "my API isn't working" beginner trap.
- **API versioning** (`/api/v1/...`) lets you evolve your API without breaking existing consumers — simple URL-prefixing is enough for most real projects.
- Always rate-limit API routes (`throttle:api`) — APIs are far more exposed to automated abuse than a normal web app.
- Next up (note 13): **Testing in Laravel** — writing automated tests (including for these exact API endpoints) using Pest/PHPUnit.