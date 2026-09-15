# Laravel 10 — Authorization: Policies & Gates

Authorization is the system that decides whether an already-identified user is allowed to perform a specific action, using either simple closures (Gates) or structured classes tied to a model (Policies).

## Table of Contents

- [Authentication vs Authorization — Recap](#authentication-vs-authorization--recap)
- [Gates: Simple, Closure-Based Checks](#gates-simple-closure-based-checks)
- [Policies: Structured, Model-Based Checks](#policies-structured-model-based-checks)
- [Creating a Policy](#creating-a-policy)
- [Standard Policy Methods](#standard-policy-methods)
- [Registering & Auto-Discovery of Policies](#registering--auto-discovery-of-policies)
- [Authorizing in Controllers](#authorizing-in-controllers)
- [Authorizing in Blade](#authorizing-in-blade)
- [Authorizing in Form Requests](#authorizing-in-form-requests)
- [Authorizing With Middleware](#authorizing-with-middleware)
- [The `before()` Hook (Super-Admin Pattern)](#the-before-hook-super-admin-pattern)
- [Policy Responses With Custom Messages](#policy-responses-with-custom-messages)
- [Gates vs Policies — When to Use Which](#gates-vs-policies--when-to-use-which)
- [Roles & Permissions (Beyond Core Laravel)](#roles--permissions-beyond-core-laravel)
- [Quick Revision](#quick-revision)

---

## Authentication vs Authorization — Recap

From note 09:

| Term | Question | Example |
|---|---|---|
| Authentication | "Who are you?" | Logging in |
| Authorization | "What can you do?" | Can THIS user edit THIS specific post? |

Authorization is always the **second** check — it assumes you already know who's asking, and now needs to decide if they're allowed to do the specific thing they're trying to do.

---

## Gates: Simple, Closure-Based Checks

- **Gate**: a simple, closure-based authorization rule, not tied to any specific Model — best for broad, one-off checks (e.g., "can this user view the admin dashboard at all?").

**Defining a Gate** (typically in `app/Providers/AppServiceProvider.php`'s `boot()` method):

```php
use Illuminate\Support\Facades\Gate;

Gate::define('view-admin-dashboard', function ($user) {
    return $user->role === 'admin';
});
```

**Checking a Gate:**

```php
if (Gate::allows('view-admin-dashboard')) {
    // Allowed
}

if (Gate::denies('view-admin-dashboard')) {
    // Not allowed
}

// Check as a SPECIFIC user, not necessarily the currently logged-in one
if (Gate::forUser($someUser)->allows('view-admin-dashboard')) {
    // ...
}
```

> 💡 **Tip:** Gates are best for actions that don't map to a specific database record — "can you access this feature at all?" rather than "can you edit this exact row?" For that second kind of question, reach for a Policy instead.

---

## Policies: Structured, Model-Based Checks

- **Policy**: a dedicated PHP class that groups every authorization rule for one specific Model — the standard, senior-recommended approach for any real CRUD feature (posts, orders, products, comments — anything users create/edit/delete).

Policies solve a real organizational problem: without them, "can this user edit this post?" logic gets scattered as inline `if` statements across every controller method that touches posts. A Policy centralizes it all in one place.

---

## Creating a Policy

```bash
php artisan make:policy PostPolicy --model=Post
```

- **`--model=Post`**: pre-fills the generated policy with the standard method signatures already type-hinted for the `Post` model, saving you the boilerplate.

This generates `app/Policies/PostPolicy.php`:

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    public function viewAny(User $user): bool
    {
        return true;
    }

    public function view(?User $user, Post $post): bool
    {
        return $post->is_published || $user?->id === $post->user_id;
    }

    public function create(User $user): bool
    {
        return $user->hasVerifiedEmail();
    }

    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

- Every method returns `true` (allowed) or `false` (denied) — plain, testable PHP logic.
- The **first parameter is always the currently authenticated `User`**; the second (where relevant) is the specific Model instance being acted on.

> 💡 **Tip:** Notice `view()` type-hints `?User $user` (nullable) — this correctly allows **guests** (not logged in at all) to view published posts, while still checking real ownership for unpublished ones. Small details like this — handling the "no user at all" case deliberately — are exactly what separates careful senior-level authorization code from code that just crashes on a guest visit.

---

## Standard Policy Methods

These method names aren't arbitrary — Laravel automatically maps them to specific Eloquent/controller actions:

| Method | Maps To | Typical Check |
|---|---|---|
| `viewAny` | Viewing a list of the resource | Can list all posts? |
| `view` | Viewing one specific record | Can view this exact post? |
| `create` | Creating a new record | Can create posts at all? |
| `update` | Editing an existing record | Owns this post, or is an editor? |
| `delete` | Deleting a record | Owns this post, or is an admin? |
| `restore` | Restoring a soft-deleted record | (Pairs with note 06's Soft Deletes) |
| `forceDelete` | Permanently deleting a soft-deleted record | Usually admin-only |

> ⚠️ **Warning:** `create` and `viewAny` don't take a Model instance as their second parameter — there's no specific record yet to check against. Forgetting this (and trying to type-hint a second parameter anyway) is a common mistake when first writing policies.

---

## Registering & Auto-Discovery of Policies

Modern Laravel (11+) **automatically discovers** policies as long as naming convention is followed: `App\Models\Post` → `App\Policies\PostPolicy`. You usually don't need to register anything manually.

If your policy doesn't follow that convention, register it explicitly (in a Service Provider's `boot()` method):

```php
use Illuminate\Support\Facades\Gate;

Gate::policy(Post::class, PostPolicy::class);
```

---

## Authorizing in Controllers

**Method 1 — `$this->authorize()`** (throws a 403 automatically on failure):

```php
public function update(Request $request, Post $post)
{
    $this->authorize('update', $post);

    // If we reach this line, authorization already passed
    $post->update($request->validated());
}
```

- **`authorize('update', $post)`**: looks up the `PostPolicy`, calls its `update()` method with the current user and `$post`, and automatically throws an `AuthorizationException` (→ HTTP 403 Forbidden) if it returns `false`.

**Method 2 — Checking manually with `Gate::allows()` (works for Policies too):**

```php
if (Gate::allows('update', $post)) {
    // ...
}
```

**Method 3 — Checking directly on the User model:**

```php
if ($request->user()->can('update', $post)) {
    // ...
}
```

> 💡 **Tip:** `$this->authorize()` is the cleanest choice inside controllers — it fails loudly and automatically with the correct HTTP status, without you writing your own `if/abort(403)` boilerplate every time.

---

## Authorizing in Blade

```blade
@can('update', $post)
    <a href="{{ route('posts.edit', $post) }}">Edit</a>
@endcan

@cannot('update', $post)
    <p>You cannot edit this post.</p>
@endcannot
```

- **`@can`/`@cannot`**: the Blade equivalent of `$user->can()` — hides or shows UI elements based on the same Policy logic, so your buttons/links only ever appear when the action is actually allowed.

> ⚠️ **Warning:** `@can` in Blade only controls what's *displayed* — it is **not** a substitute for `$this->authorize()` in the controller. A user could still manually submit a form/request even if a button was hidden from them. Always enforce authorization server-side; treat Blade-level checks as a UX nicety only.

---

## Authorizing in Form Requests

Recall the `authorize()` method from note 08's Form Request classes — this is exactly where Policies commonly plug in:

```php
class UpdatePostRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('update', $this->route('post'));
    }

    public function rules(): array
    {
        return [
            'title' => 'required|max:255',
        ];
    }
}
```

- If `authorize()` returns `false`, Laravel automatically returns a 403 response **before** validation even runs — combining note 08 and note 10 into one clean, self-contained class.

---

## Authorizing With Middleware

```php
Route::put('/posts/{post}', [PostController::class, 'update'])
    ->middleware('can:update,post');
```

- **`'can:update,post'`**: `update` is the Policy method to check; `post` refers to the route parameter (via Route Model Binding from note 02) that gets passed as the Model instance.

---

## The `before()` Hook (Super-Admin Pattern)

A very common real-world need: letting admins bypass every single policy check, without repeating `|| $user->isAdmin()` in every single method.

```php
class PostPolicy
{
    public function before(User $user, string $ability): bool|null
    {
        if ($user->isAdmin()) {
            return true; // Bypasses ALL other checks in this policy entirely
        }

        return null; // Falls through to the normal method logic below
    }

    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

- **`before()`**: runs automatically before every other method in the policy. Returning `true` or `false` short-circuits the whole check immediately; returning `null` lets the normal, specific method run as usual.

> 💡 **Tip:** This is the textbook senior-level pattern for a "super-admin can do anything" rule — write it once in `before()`, instead of scattering `isAdmin()` checks throughout every policy method in your app.

---

## Policy Responses With Custom Messages

Plain `true`/`false` doesn't let you explain *why* something was denied. For that, return a `Response` object instead:

```php
use Illuminate\Auth\Access\Response;

public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::deny('You do not own this post.');
}
```

- When denied this way, `$this->authorize()` throws an exception carrying your custom message instead of Laravel's generic "This action is unauthorized" text — useful for clearer error messages in an API response.

---

## Gates vs Policies — When to Use Which

| Scenario | Use |
|---|---|
| "Can this user access the admin panel at all?" | Gate |
| "Can this user edit THIS specific post?" | Policy |
| Logic tied to CRUD on a Model | Policy |
| A one-off, app-wide feature flag/permission | Gate |
| Reusable across MANY different models | Gate (or a shared trait/base policy) |

---

## Roles & Permissions (Beyond Core Laravel)

Everything above is Laravel's **built-in** authorization toolkit — enough for most apps. For more complex needs (many distinct roles, admin-configurable permissions, permission groups), most real-world/senior teams reach for a dedicated package rather than hand-rolling it:

- **Spatie Laravel-Permission**: the de facto standard package for role- and permission-based access control in Laravel, letting you assign named roles (`editor`, `admin`) and granular permissions (`edit-posts`, `delete-users`) to users, then check them (`$user->hasRole('admin')`, `$user->can('edit-posts')`) — and it integrates directly with the Gates/Policies system you just learned, rather than replacing it.

> 💡 **Tip:** Learning Gates/Policies first (as you just did) is the right order — packages like Spatie's build directly on top of these same concepts, so understanding the underlying mechanism makes the package immediately intuitive instead of feeling like unrelated magic.

---

## Quick Revision

- **Authorization** (this note) always comes after **Authentication** (note 09) — it assumes you already know who the user is.
- **Gates** are simple closures for broad, non-model-specific checks (`Gate::define()`, `Gate::allows()`/`denies()`) — defined in a Service Provider.
- **Policies** (`php artisan make:policy PostPolicy --model=Post`) group all authorization logic for one Model into standard methods: `viewAny`, `view`, `create`, `update`, `delete`, `restore`, `forceDelete`.
- Modern Laravel auto-discovers policies by naming convention (`Post` → `PostPolicy`) — manual registration is only needed for non-standard naming.
- Use `$this->authorize('update', $post)` in controllers (auto-403 on failure), `@can`/`@cannot` in Blade (UI-only, never a substitute for server-side checks), `can:update,post` in route middleware, or `$user->can()` in a Form Request's `authorize()`.
- `before()` lets a policy short-circuit every check at once — the standard pattern for a "super-admin bypasses everything" rule.
- Return a `Response` object (`Response::allow()`/`deny('message')`) instead of plain booleans when you need a custom denial message, especially useful for APIs.
- For complex, admin-configurable role/permission systems beyond simple ownership checks, the Spatie Laravel-Permission package is the real-world standard, built directly on top of Gates/Policies.
- Next up (note 11): **File Storage & Uploads** — handling images and files with Laravel's filesystem abstraction.