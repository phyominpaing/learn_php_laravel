# Laravel 08 — Validation & Form Requests

Validation is the process of checking incoming request data against a set of rules before your app trusts and saves it, protecting your database from bad or malicious input.

## Table of Contents

- [Why Validation Comes Before Everything Else](#why-validation-comes-before-everything-else)
- [Basic Validation in a Controller](#basic-validation-in-a-controller)
- [Common Validation Rules Reference](#common-validation-rules-reference)
- [Displaying Validation Errors in Blade](#displaying-validation-errors-in-blade)
- [Form Request Classes](#form-request-classes)
- [Custom Error Messages](#custom-error-messages)
- [Custom Validation Rules](#custom-validation-rules)
- [Conditional Validation Rules](#conditional-validation-rules)
- [Validating Arrays & Nested Data](#validating-arrays--nested-data)
- [Validating APIs (JSON Requests)](#validating-apis-json-requests)
- [Quick Revision](#quick-revision)

---

## Why Validation Comes Before Everything Else

Recall the warning from note 03 and note 06: **never trust raw request data**. Validation is exactly where that trust boundary is enforced — it's the checkpoint between "data a user typed" and "data your app is allowed to act on."

> ⚠️ **Warning:** Skipping validation doesn't just risk bad data (empty names, invalid emails) — it opens the door to real security issues, since unchecked input is the root cause of most web application vulnerabilities.

---

## Basic Validation in a Controller

The simplest approach — calling `->validate()` directly on the incoming `Request`:

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'name' => 'required|max:255',
        'email' => 'required|email|unique:users',
        'password' => 'required|min:8',
    ]);

    // $validated is now a clean array containing ONLY the validated fields
    User::create($validated);
}
```

- **`->validate([...])`**: runs the given rules against the request. If validation fails, Laravel **automatically** redirects the user back to the previous page with the errors attached — you don't write that redirect logic yourself.
- If validation passes, it returns an array containing only the fields you validated (a small, deliberate security benefit — extra unexpected fields get dropped, not passed through).
- Rules for one field are written as a single `|`-separated string, or alternatively as an array: `'password' => ['required', 'min:8']` (useful when a rule itself contains commas, like `in:one,two,three`).

> 💡 **Tip:** This inline style is fine for small, one-off forms. For anything with more than 3–4 fields, or rules you'll reuse, use a **Form Request** class (below) instead — it keeps controllers clean and is the standard senior-level approach.

---

## Common Validation Rules Reference

| Rule | Meaning |
|---|---|
| `required` | Field must be present and not empty |
| `nullable` | Field is allowed to be empty/absent |
| `email` | Must be a valid email format |
| `min:8` | Minimum length (strings) or value (numbers) |
| `max:255` | Maximum length (strings) or value (numbers) |
| `numeric` | Must be a number |
| `integer` | Must be a whole number |
| `string` | Must be a string |
| `boolean` | Must be `true`/`false`, `1`/`0` |
| `date` | Must be a valid date |
| `unique:users,email` | Value must not already exist in `users` table's `email` column |
| `exists:categories,id` | Value MUST already exist in that table/column |
| `confirmed` | Requires a matching `..._confirmation` field (e.g., `password_confirmation`) |
| `in:draft,published` | Value must be one of the listed options |
| `url` | Must be a valid URL |
| `file` | Must be an uploaded file |
| `image` | Must be an uploaded image file |
| `max:2048` (with `file`) | Max file size in kilobytes |
| `regex:/pattern/` | Must match a custom regular expression |

> 💡 **Tip:** `unique` and `exists` are your two most important database-aware rules — `unique` for "this shouldn't already exist" (new emails, usernames), `exists` for "this must already exist" (a `category_id` referencing a real category).

---

## Displaying Validation Errors in Blade

When validation fails and Laravel redirects back, it automatically "flashes" the errors into the session for that next page load.

```blade
@if($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<input type="text" name="email" value="{{ old('email') }}">
@error('email')
    <span class="text-danger">{{ $message }}</span>
@enderror
```

- **`$errors`**: automatically available in every Blade view without you passing it manually — Laravel shares it globally whenever errors exist.
- **`old('email')`**: repopulates a form field with the previously submitted value, so the user doesn't have to retype everything after a failed submission.
- **`@error('field') ... @enderror`**: a shortcut for showing an error message for one specific field, only if that field actually failed validation.

> 💡 **Tip:** Always use `old()` on your form inputs. Forgetting it is a classic beginner mistake that frustrates users — they fix one typo and lose everything else they typed.

---

## Form Request Classes

**Form Request**: a dedicated class that holds a request's validation rules (and optionally, authorization logic) — separating that concern entirely out of your controller.

```bash
php artisan make:request StoreProductRequest
```

This generates `app/Http/Requests/StoreProductRequest.php`:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreProductRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'name' => 'required|max:255',
            'price' => 'required|numeric|min:0',
            'category_id' => 'required|exists:categories,id',
        ];
    }
}
```

- **`authorize()`**: return `true` if any authenticated request is allowed to make this request, or add real logic (e.g., "only this product's owner can update it") — covered further in note 10.
- **`rules()`**: the exact same rule syntax as inline validation, just moved into its own method.

**Using it in a controller — this is the entire benefit:**

```php
use App\Http\Requests\StoreProductRequest;

public function store(StoreProductRequest $request)
{
    // Validation already ran automatically before this line executes!
    Product::create($request->validated());
}
```

- Simply **type-hinting** `StoreProductRequest` instead of the plain `Request` class is enough — Laravel automatically runs validation before your method body even starts. If it fails, the user is redirected back with errors, exactly like before, with zero extra code in the controller.
- **`$request->validated()`**: returns just the validated data array — the Form Request equivalent of what `$request->validate([...])` returned inline.

> 💡 **Tip:** This is genuinely one of the clearest signs of senior-quality Laravel code: controllers stay short and readable, because validation rules live in their own dedicated, reusable, testable class.

---

## Custom Error Messages

```php
public function messages(): array
{
    return [
        'name.required' => 'Please give your product a name.',
        'price.min' => 'Price cannot be negative.',
    ];
}
```

- Add this method inside a Form Request class (or pass a second array to inline `validate()`) to override Laravel's default generic messages with wording that fits your app.

**Customizing field display names** (used inside generic messages, e.g., "The :attribute field is required"):

```php
public function attributes(): array
{
    return [
        'category_id' => 'category',
    ];
}
```

---

## Custom Validation Rules

When built-in rules aren't enough — e.g., checking that a coupon code is still valid.

```bash
php artisan make:rule ValidCouponCode
```

```php
namespace App\Rules;

use Illuminate\Contracts\Validation\ValidationRule;
use Closure;

class ValidCouponCode implements ValidationRule
{
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if (! Coupon::where('code', $value)->where('active', true)->exists()) {
            $fail('The :attribute is not a valid or active coupon code.');
        }
    }
}
```

**Using it:**

```php
use App\Rules\ValidCouponCode;

'coupon_code' => ['required', new ValidCouponCode],
```

> 💡 **Tip:** Reach for a custom rule class whenever a validation check needs actual logic (a database lookup, checking against an external service) rather than a simple built-in keyword.

---

## Conditional Validation Rules

Sometimes a rule should only apply in certain situations.

```php
use Illuminate\Validation\Rule;

public function rules(): array
{
    return [
        'email' => Rule::when(
            ! auth()->check(),
            ['required', 'email', 'unique:users']
        ),
    ];
}
```

- **`Rule::when($condition, $rules)`**: applies the given rules only if the condition is `true` — here, only requiring an email if the user isn't already logged in.

**Requiring a field only if another field has a specific value:**

```php
'company_name' => 'required_if:account_type,business',
```

---

## Validating Arrays & Nested Data

Real forms often submit arrays — e.g., multiple order line items at once.

```php
$request->validate([
    'items' => 'required|array|min:1',
    'items.*.product_id' => 'required|exists:products,id',
    'items.*.quantity' => 'required|integer|min:1',
]);
```

- **`items.*.product_id`**: the `*` wildcard applies this rule to **every** element inside the `items` array — so all submitted line items get validated individually, without writing a loop yourself.

---

## Validating APIs (JSON Requests)

Form Requests work identically for JSON API requests — no code changes needed. The only real difference is what happens **on failure**:

| Request Type | On Validation Failure |
|---|---|
| Normal web form (`web.php`) | Redirects back with errors flashed to session |
| API/JSON request (`api.php`) | Automatically returns a `422 Unprocessable Entity` JSON response with error details |

- Laravel detects this automatically based on whether the request expects JSON (checked via headers) — you don't configure this manually.

> 💡 **Tip:** This is genuinely convenient — the exact same Form Request class works perfectly whether it's called from a Blade form or a React/Next.js frontend making a fetch request, which matters directly for the project we'll build at the end of this series.

---

## Quick Revision

- Validation is the trust boundary between raw user input and data your app acts on — never skip it, especially before saving to the database.
- `$request->validate([...])` is the quick inline approach; it auto-redirects on failure and returns only the validated fields on success.
- Key rules to know cold: `required`, `email`, `min`/`max`, `unique:table,column`, `exists:table,column`, `confirmed`, `in:...`.
- In Blade, `$errors` is globally available, `old('field')` repopulates inputs after a failed submission, and `@error('field') ... @enderror` shows a field-specific message.
- **Form Request classes** (`php artisan make:request Name`) move `rules()`, `authorize()`, `messages()`, and `attributes()` out of the controller — just type-hint the class in your method signature and validation runs automatically before your code executes.
- Write **custom rule classes** (`php artisan make:rule`) for validation logic beyond simple keywords (e.g., checking against another table's business logic).
- `Rule::when()` and `required_if` handle rules that should only apply conditionally.
- The `field.*.subfield` wildcard syntax validates every item inside a submitted array in one line.
- The exact same Form Request class works for both web forms and JSON API requests — Laravel automatically returns a 422 JSON response for API failures instead of a redirect.
- Next up (note 09): **Middleware & Authentication** — registration, login, Sanctum API tokens, and Two-Factor Authentication (2FA).