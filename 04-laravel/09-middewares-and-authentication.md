# Laravel 09 — Middleware & Authentication

Middleware is code that runs before or after a request reaches your controller, and authentication is the system that identifies which user (if any) is making that request.

## Table of Contents

- [What Middleware Actually Is](#what-middleware-actually-is)
- [Built-In Middleware You'll Use Constantly](#built-in-middleware-youll-use-constantly)
- [Creating Custom Middleware](#creating-custom-middleware)
- [Registering & Assigning Middleware](#registering--assigning-middleware)
- [Middleware Parameters](#middleware-parameters)
- [Terminable Middleware](#terminable-middleware)
- [Authentication vs Authorization](#authentication-vs-authorization)
- [Guards & the `auth` Config](#guards--the-auth-config)
- [Building Auth From Scratch (Understanding the Pieces)](#building-auth-from-scratch-understanding-the-pieces)
- [Laravel Breeze & Fortify (Scaffolding Options)](#laravel-breeze--fortify-scaffolding-options)
- [Session-Based Auth vs Token-Based Auth](#session-based-auth-vs-token-based-auth)
- [Laravel Sanctum for API Authentication](#laravel-sanctum-for-api-authentication)
- [Two-Factor Authentication (2FA)](#two-factor-authentication-2fa)
- [Password Hashing — Why You Never Roll Your Own](#password-hashing--why-you-never-roll-your-own)
- [Quick Revision](#quick-revision)

---

## What Middleware Actually Is

Recall the request lifecycle diagram from note 03:

```
Request → Middleware → Router → Controller → Response
```

- **Middleware**: a class that sits in this pipeline and can inspect, modify, block, or log a request **before** it reaches your controller — and can also modify the **response** on its way back out.
- Middleware is how Laravel implements cross-cutting concerns — logic that applies broadly across many routes (auth checks, logging, CORS headers) without repeating that logic in every single controller method.

**Mental model:** picture middleware as a series of security checkpoints/gates a request must pass through, one by one, before finally reaching your controller.

```
Request → [Gate 1: Is HTTPS?] → [Gate 2: Logged in?] → [Gate 3: Rate limit OK?] → Controller
```

> 💡 **Tip:** If you've done frontend work with Express.js middleware or Next.js middleware, this is the exact same concept, just in PHP — a function/class that runs "in between" the request arriving and your actual route logic.

---

## Built-In Middleware You'll Use Constantly

| Middleware | Purpose |
|---|---|
| `auth` | Blocks the request unless the user is logged in (redirects guests to login) |
| `guest` | The inverse — blocks the request if the user IS logged in (used on login/register pages) |
| `verified` | Requires the user's email to be verified |
| `throttle:60,1` | Rate limits — max 60 requests per 1 minute |
| `auth:sanctum` | Requires a valid Sanctum API token (for API routes) |
| `can:edit-post` | Requires the user to pass a specific authorization check (note 10) |
| `signed` | Requires the URL to have a valid cryptographic signature (used for things like email verification links) |

```php
Route::get('/dashboard', function () {
    // ...
})->middleware('auth');

Route::get('/login', function () {
    // ...
})->middleware('guest');
```

---

## Creating Custom Middleware

```bash
php artisan make:middleware EnsureUserIsSubscribed
```

This generates `app/Http/Middleware/EnsureUserIsSubscribed.php`:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserIsSubscribed
{
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()?->is_subscribed) {
            return redirect('/subscribe');
        }

        return $next($request);
    }
}
```

- **`handle(Request $request, Closure $next)`**: every middleware's core method. `$request` is the incoming request; `$next` represents "the rest of the pipeline" (the next middleware, or ultimately, the controller).
- **`return $next($request);`**: this is the crucial line — it passes control forward. If you don't call this, the request **stops here** and never reaches the controller.
- **`return redirect('/subscribe');`** (instead of calling `$next`): this is how middleware **blocks** a request — by returning its own response instead of passing control along.

> ⚠️ **Warning:** Forgetting to call `$next($request)` (or accidentally returning something before it) will silently make every route using this middleware stop working, with no obvious error — a common debugging trap for beginners writing their first custom middleware.

**Logic that should run *after* the response** (rare, but good to know exists):

```php
public function handle(Request $request, Closure $next): Response
{
    $response = $next($request);

    // Runs AFTER the controller has already produced a response
    Log::info('Response sent for: '.$request->path());

    return $response;
}
```

---

## Registering & Assigning Middleware

Modern Laravel (11+) registers global and grouped middleware inside `bootstrap/app.php`:

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'subscribed' => \App\Http\Middleware\EnsureUserIsSubscribed::class,
    ]);
})
```

- **`->alias([...])`**: gives your middleware class a short string name, so you can reference it in routes as `'subscribed'` instead of typing the full class path every time.

**Applying middleware to a route:**

```php
Route::get('/premium-content', function () {
    // ...
})->middleware('subscribed');

// Multiple middleware at once
Route::get('/admin', function () {
    // ...
})->middleware(['auth', 'verified', 'subscribed']);
```

**Applying middleware to an entire group** (from note 02):

```php
Route::middleware(['auth', 'subscribed'])->group(function () {
    Route::get('/dashboard', ...);
    Route::get('/settings', ...);
});
```

**Applying middleware inside a Controller's constructor** (an alternative, class-level approach):

```php
public function __construct()
{
    $this->middleware('auth');
    $this->middleware('subscribed')->only(['store', 'update']);
}
```

- **`->only([...])`**: restricts the middleware to specific methods on that controller only.
- **`->except([...])`**: the inverse — applies to every method except the ones listed.

---

## Middleware Parameters

Middleware can receive extra arguments, like `throttle:60,1` above.

```php
public function handle(Request $request, Closure $next, string $role): Response
{
    if ($request->user()?->role !== $role) {
        abort(403);
    }

    return $next($request);
}
```

```php
Route::get('/admin', ...)->middleware('role:admin');
```

- The string after the colon (`admin`) is passed in as the `$role` parameter — this pattern lets one middleware class be reused flexibly for different roles.

---

## Terminable Middleware

For logic that should run **after** the response has already been sent to the browser (so it doesn't slow down what the user sees) — e.g., logging or cleanup tasks.

```php
public function terminate(Request $request, Response $response): void
{
    // Runs after the response is sent to the browser
}
```

> 💡 **Tip:** This is a genuinely senior-level detail — most beginners never need `terminate()`, but knowing it exists matters when you eventually need to do post-response bookkeeping (analytics logging, cleanup) without adding latency to the user's actual request.

---

## Authentication vs Authorization

These two terms are constantly confused — get this distinction locked in now:

| Term | Question It Answers | Example |
|---|---|---|
| **Authentication** | "Who are you?" | Logging in with email/password |
| **Authorization** | "What are you allowed to do?" | Can THIS user edit THIS specific post? |

- Authentication happens first (note 09, this note); authorization happens after, on a per-action basis (note 10).

---

## Guards & the `auth` Config

- **Guard**: defines *how* users get authenticated for a given request — e.g., checking a session cookie vs. checking an API token.

`config/auth.php`:

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'sanctum',
        'provider' => 'users',
    ],
],
```

- **`web` guard**: used for normal browser sessions (cookie-based) — this is Laravel's default.
- **`api` guard**: used for stateless API requests via Sanctum tokens.
- **`provider`**: tells the guard which Eloquent Model represents a "user" (by default, `App\Models\User`) — useful if your app ever needs multiple distinct user types (e.g., `Admin` and `Customer` as separate tables).

```php
auth()->user();              // Get the currently authenticated user (default guard)
auth('api')->user();         // Get the currently authenticated user for a SPECIFIC guard
auth()->check();             // true/false — is anyone logged in?
auth()->id();                // The logged-in user's ID, or null
```

---

## Building Auth From Scratch (Understanding the Pieces)

Before reaching for a scaffolding package, it's worth seeing what "logging in" actually does under the hood:

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;

public function login(Request $request)
{
    $credentials = $request->validate([
        'email' => 'required|email',
        'password' => 'required',
    ]);

    if (Auth::attempt($credentials)) {
        $request->session()->regenerate();
        return redirect()->intended('/dashboard');
    }

    return back()->withErrors(['email' => 'Invalid credentials.']);
}
```

- **`Auth::attempt($credentials)`**: looks up a user by email, then checks the given password against the stored hash (using `Hash::check` internally) — returns `true`/`false`.
- **`$request->session()->regenerate()`**: issues a fresh session ID after login — a deliberate security measure preventing **session fixation attacks** (where an attacker tricks a victim into using a known session ID, then hijacks it after they log in).
- **`redirect()->intended('/dashboard')`**: sends the user to whatever page they originally tried to visit before being redirected to login (falls back to `/dashboard` if there wasn't one).

**Logging out:**

```php
public function logout(Request $request)
{
    Auth::logout();
    $request->session()->invalidate();
    $request->session()->regenerateToken();
    return redirect('/');
}
```

- **`session()->invalidate()`**: destroys the current session data.
- **`session()->regenerateToken()`**: rotates the CSRF token too, since it was tied to the now-dead session.

> ⚠️ **Warning:** Skipping `regenerate()` on login or `invalidate()`/`regenerateToken()` on logout are subtle, real security mistakes — not just theoretical. This is exactly the kind of detail that separates copy-pasted auth code from someone who actually understands what it's doing.

---

## Laravel Breeze & Fortify (Scaffolding Options)

Writing all of the above by hand for every project is repetitive — Laravel offers official starter packages:

| Package | What It Gives You |
|---|---|
| **Breeze** | Minimal, ready-made login/register/password-reset routes, controllers, AND Blade views — great for learning, easy to fully customize afterward |
| **Fortify** | The same backend logic as Breeze (routes, controllers, 2FA support) but with **no views included** — a "headless" auth backend, meant for when you're building your own frontend (React, Vue, or a separate API consumer) |
| **Jetstream** | A fuller-featured starter kit built on top of Fortify — adds teams, API tokens, profile management UI |

```bash
composer require laravel/breeze --dev
php artisan breeze:install
```

```bash
composer require laravel/fortify
php artisan vendor:publish --provider="Laravel\Fortify\FortifyServiceProvider"
php artisan migrate
```

> 💡 **Tip:** For your end-of-series project — a Laravel API backend talking to a separate React/Next.js frontend — **Fortify** (headless, no Blade views) is the more appropriate fit compared to Breeze, since you won't be using Laravel's own Blade login pages at all.

---

## Session-Based Auth vs Token-Based Auth

| | Session-Based Auth | Token-Based Auth |
|---|---|---|
| How identity is tracked | A cookie holding a session ID | A token sent in an `Authorization` header |
| Typical use case | Traditional server-rendered Blade apps | APIs consumed by a separate frontend/mobile app |
| State | Stateful — server remembers the session | Stateless — server verifies the token fresh each time |
| Laravel tool | Built-in (`web` guard) | Sanctum |

> 💡 **Tip:** Since your end goal is a React/Next.js frontend talking to a Laravel API, you'll primarily be working with **token-based auth via Sanctum** — but understanding session auth first (above) makes Sanctum far easier to reason about, since Sanctum can actually use EITHER approach depending on setup.

---

## Laravel Sanctum for API Authentication

**Sanctum**: Laravel's official lightweight package for authenticating API requests, supporting two distinct use cases.

**Use Case 1 — SPA Authentication** (your React/Next.js frontend, served from a trusted first-party domain):
- Uses Laravel's normal cookie-based session system under the hood — no manual token handling in your frontend code at all.
- Requires your frontend and backend to share the same top-level domain (or be configured for it) since it relies on cookies.

**Use Case 2 — API Tokens** (mobile apps, or third-party API consumers):

```php
// Issuing a token, e.g. after a successful login
$token = $user->createToken('mobile-app')->plainTextToken;
```

```php
// Every subsequent request from the client includes:
// Authorization: Bearer <token>
```

**Protecting routes with Sanctum:**

```php
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

**Revoking tokens:**

```php
$user->tokens()->delete();             // Revoke ALL of this user's tokens
$request->user()->currentAccessToken()->delete(); // Revoke just the current one (logout)
```

**Token abilities** (scoped permissions per token — an advanced, senior-relevant feature):

```php
$token = $user->createToken('mobile-app', ['products:read'])->plainTextToken;

// Later, in a controller or middleware:
if ($request->user()->tokenCan('products:read')) {
    // ...
}
```

- **Token abilities**: let a single user issue different tokens with different permission scopes — e.g., a read-only integration token vs. a full-access mobile app token, without needing separate user accounts.

> 💡 **Tip:** We'll set up Sanctum properly, end-to-end, in note 12 (API Development) when we build real API endpoints for the final project. This section is your conceptual foundation for that.

---

## Two-Factor Authentication (2FA)

You specifically asked about this — here's a full working example using **Laravel Fortify**, which has 2FA support built in.

- **2FA (Two-Factor Authentication)**: requiring a second proof of identity beyond just a password — typically a time-based one-time code generated by an authenticator app (Google Authenticator, Authy).
- **TOTP (Time-based One-Time Password)**: the standard algorithm behind most authenticator apps — generates a new 6-digit code every 30 seconds, derived from a shared secret key plus the current time.

**Enabling the feature in `config/fortify.php`:**

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::twoFactorAuthentication([
        'confirm' => true,
        'confirmPassword' => true,
    ]),
],
```

- **`Features::twoFactorAuthentication()`**: turns on Fortify's built-in 2FA routes and logic.
- **`'confirm' => true`**: requires the user to confirm 2FA setup by entering a valid code once before it's fully activated (prevents accidentally locking yourself out with a typo'd setup).

**The user flow, step by step:**

1. **Enabling 2FA** — the user requests it be turned on:

```php
// POST /user/two-factor-authentication
// Fortify generates a secret key and stores it (encrypted) on the user record
```

2. **Displaying the QR code** — the user scans this with their authenticator app:

```blade
{!! auth()->user()->twoFactorQrCodeSvg() !!}
```

- **`twoFactorQrCodeSvg()`**: generates a scannable QR code encoding the secret key, so the user doesn't have to manually type a long string into their authenticator app.

3. **Confirming setup** — the user enters the current 6-digit code from their app to prove it's working:

```php
// POST /user/confirmed-two-factor-authentication
// Body: { "code": "123456" }
```

4. **Recovery codes** — Fortify also generates a set of one-time backup codes:

```php
auth()->user()->recoveryCodes();
```

- **Recovery codes**: single-use backup codes given to the user in case they lose access to their authenticator app — without these, a lost phone could permanently lock someone out of their own account.

5. **Logging in with 2FA enabled** — after a correct password, Fortify redirects to a second "enter your code" screen instead of logging the user straight in:

```php
// POST /login (email + password)
//   → if 2FA is enabled, redirects to a challenge screen instead of dashboard

// POST /two-factor-challenge
// Body: { "code": "123456" }  OR  { "recovery_code": "..." }
```

**Disabling 2FA:**

```php
// DELETE /user/two-factor-authentication
```

> ⚠️ **Warning:** Always show recovery codes to the user exactly once, right after setup, and make clear they should save them somewhere safe. There's no way to display them again later without regenerating a brand-new set (which invalidates the old ones).

> 💡 **Tip:** For **SMS-based** OTP (a text message code instead of an authenticator app) rather than TOTP, you'd combine this concept with an SMS provider like Twilio — that full implementation, including generating and verifying a short-lived numeric code, is covered in note 19 (Mail, SMS & OTP Integration).

---

## Password Hashing — Why You Never Roll Your Own

```php
use Illuminate\Support\Facades\Hash;

$hashed = Hash::make('plain-text-password');
Hash::check('plain-text-password', $hashed); // true
```

- **Hashing**: a one-way transformation — you can verify a password matches, but you can never reverse a hash back into the original password.
- Laravel uses **bcrypt** by default (configurable to Argon2), both intentionally slow algorithms — this is a deliberate defense: it makes brute-force password-guessing attacks computationally expensive, even if your database is ever leaked.

> ⚠️ **Warning:** Never store passwords in plain text, and never write your own hashing logic. Laravel's `Hash` facade (and the underlying `$user->password` cast to `'hashed'` on the Model, in modern Laravel) already handles this correctly and securely — there's no legitimate reason to do it differently.

---

## Quick Revision

- **Middleware** intercepts requests before they reach a controller (and can act on the response afterward) — the standard place for auth checks, rate limiting, and logging.
- A middleware's `handle()` method must call `$next($request)` to let the request continue, or return its own response to block it.
- Register middleware aliases in `bootstrap/app.php`, then apply them with `->middleware('name')` on routes, route groups, or in a controller's constructor with `->only()`/`->except()`.
- **Authentication** answers "who are you?"; **Authorization** (note 10) answers "what can you do?" — don't conflate them.
- **Guards** (`web`, `api`, etc., in `config/auth.php`) define *how* a request proves identity — session cookie vs. API token.
- `Auth::attempt()`, `session()->regenerate()` on login, and `session()->invalidate()` + `regenerateToken()` on logout are the real security-relevant building blocks under any auth scaffolding.
- **Breeze** gives you full Blade views for auth; **Fortify** is the same backend logic, headless (no views) — the better fit when your frontend is a separate SPA like React/Next.js.
- **Sanctum** handles both SPA (cookie-based) and mobile/API (Bearer token) authentication, including scoped **token abilities** — full hands-on setup comes in note 12.
- **2FA via Fortify** uses TOTP (authenticator app codes), a scannable QR code for setup, one-time **recovery codes** for backup access, and a second "challenge" step during login.
- Passwords are always hashed (bcrypt/Argon2 by default via `Hash::make()`), never stored or compared in plain text — this is non-negotiable, senior-level or not.
- Next up (note 10): **Authorization** — Policies and Gates, controlling exactly what a logged-in user is allowed to do.