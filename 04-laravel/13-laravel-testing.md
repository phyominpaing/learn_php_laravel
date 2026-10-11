# Laravel 13 — Testing

Automated testing means writing code that runs your application's code and checks the results for you, so you can change things confidently without manually clicking through the app to see what broke.

## Table of Contents

- [Why Test At All](#why-test-at-all)
- [Feature Tests vs Unit Tests](#feature-tests-vs-unit-tests)
- [Pest vs PHPUnit](#pest-vs-phpunit)
- [Test Folder Structure & Running Tests](#test-folder-structure--running-tests)
- [Your First Test](#your-first-test)
- [The Test Database (and a Dangerous Mistake)](#the-test-database-and-a-dangerous-mistake)
- [Factories in Tests](#factories-in-tests)
- [Testing HTTP Endpoints](#testing-http-endpoints)
- [Common Response Assertions](#common-response-assertions)
- [Testing Authentication](#testing-authentication)
- [Testing Validation](#testing-validation)
- [Testing Authorization (Policies)](#testing-authorization-policies)
- [Database Assertions](#database-assertions)
- [Testing File Uploads](#testing-file-uploads)
- [Fakes: Mail, Queue, Events, HTTP](#fakes-mail-queue-events-http)
- [Mocking Your Own Classes](#mocking-your-own-classes)
- [Datasets: One Test, Many Inputs](#datasets-one-test-many-inputs)
- [Unit Tests — When They Make Sense](#unit-tests--when-they-make-sense)
- [Architecture Tests](#architecture-tests)
- [Coverage, Parallel Runs & CI](#coverage-parallel-runs--ci)
- [TDD — Test-Driven Development](#tdd--test-driven-development)
- [What a Senior Test Suite Looks Like](#what-a-senior-test-suite-looks-like)
- [Quick Revision](#quick-revision)

---

## Why Test At All

Imagine your API has 40 endpoints. You change how `Article` validation works. Did you break something? Without tests, you must manually try all 40 endpoints in Postman. With tests, you type one command and know in seconds.

- **Automated test**: a small piece of code that performs an action in your app (call an endpoint, create a record) and **asserts** (checks) that the result is what you expect.
- **Regression**: a bug where something that used to work breaks after a new change. Tests exist mainly to catch regressions.
- **Refactoring safety**: tests let you restructure messy code (note 15) while proving the behavior didn't change.

> 💡 **Tip:** Think of tests as a safety net under a trapeze artist. You can try bolder moves (bigger refactors, upgrades) because falling no longer means disaster. This is why senior teams treat tests as part of the feature, not an optional extra.

---

## Feature Tests vs Unit Tests

Laravel splits tests into two folders for a reason.

| | Feature Test | Unit Test |
|---|---|---|
| Folder | `tests/Feature/` | `tests/Unit/` |
| Boots the full app? | Yes (routes, middleware, database, container) | No, tests one class in isolation |
| Typical target | An endpoint, a console command, a job | A pure calculation, a helper, a small service class |
| Speed | Slower | Very fast |
| Confidence it gives | High, mimics real usage | Narrow, only that one piece |

- **Feature test**: sends a real (simulated) HTTP request through the whole framework and checks the response and database.
- **Unit test**: calls one method or class directly with no framework involvement.

> 💡 **Tip:** In Laravel apps, write **mostly feature tests**. A single feature test that posts to `/api/articles` and checks the database row proves routing, validation, controller, model, and database all work together, which six mocked unit tests never could.

---

## Pest vs PHPUnit

Laravel supports two styles. Both run the same underlying engine.

| | PHPUnit | Pest |
|---|---|---|
| Style | Classes with methods | Plain functions with readable sentences |
| Boilerplate | More | Less |
| Default in new Laravel projects | Older default | Newer projects usually ship Pest, but the installer lets you choose |

**PHPUnit style:**

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_homepage_loads(): void
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```

**Pest style (same test):**

```php
<?php

it('loads the homepage', function () {
    $this->get('/')->assertStatus(200);
});
```

- **`it('...', function () {...})`**: Pest's way of defining a test, reading like a sentence: "it loads the homepage."
- **`test('...', function () {...})`**: an alias of `it()` when the sentence reads better without "it".

> 💡 **Tip:** The concepts (assertions, factories, fakes) are identical in both. This note uses **Pest** style, since it's shorter and easier to read. If your team uses PHPUnit, you can translate one-to-one: a Pest `it()` becomes a `test_` method on a class.

---

## Test Folder Structure & Running Tests

```
tests/
├── Feature/        ← HTTP/app-level tests (most of your tests)
├── Unit/           ← isolated class tests
├── Pest.php        ← Pest global configuration
└── TestCase.php    ← base class every test extends
```

**Create a test:**

```bash
php artisan make:test ArticleApiTest --pest          # feature test (Pest)
php artisan make:test PriceCalculatorTest --unit --pest   # unit test
```

**Run tests:**

```bash
php artisan test                      # run everything
php artisan test --filter=articles    # run tests whose name matches "articles"
php artisan test tests/Feature/ArticleApiTest.php   # run one file
php artisan test --parallel           # run tests in parallel processes (faster)
```

> 💡 **Tip:** Run tests constantly while developing, not just at the end. A failing test right after you wrote 10 lines is far easier to fix than one discovered after 500 lines.

---

## Your First Test

Global setup lives in `tests/Pest.php`. This line applies the base `TestCase` and automatically resets the database for every Feature test:

```php
<?php

use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->extend(Tests\TestCase::class)
    ->use(RefreshDatabase::class)
    ->in('Feature');
```

- **`extend(TestCase::class)`**: gives every test access to `$this->get()`, `$this->postJson()`, `$this->actingAs()`, etc.
- **`use(RefreshDatabase::class)`**: resets the database between tests (explained next).
- **`->in('Feature')`**: applies this to everything inside `tests/Feature`.

A real first test:

```php
<?php

use App\Models\Article;

it('returns a list of articles', function () {
    Article::factory()->count(3)->create();

    $response = $this->getJson('/api/articles');

    $response->assertOk()
        ->assertJsonCount(3, 'data');
});
```

The structure of almost every test follows **Arrange → Act → Assert**:

| Step | Meaning | In the example |
|---|---|---|
| **Arrange** | Prepare the data/state | `Article::factory()->count(3)->create()` |
| **Act** | Perform the action | `$this->getJson('/api/articles')` |
| **Assert** | Check the outcome | `assertOk()`, `assertJsonCount(...)` |

---

## The Test Database (and a Dangerous Mistake)

Tests create and delete lots of data. They must run against a **separate, throwaway database**, never your real one.

- **`RefreshDatabase`**: a trait that makes sure the schema is migrated and wraps each test in a transaction that is rolled back afterward, so data from one test never leaks into the next.

**Where the test database is configured: `phpunit.xml`**

```xml
<php>
    <env name="APP_ENV" value="testing"/>
    <env name="DB_CONNECTION" value="sqlite"/>
    <env name="DB_DATABASE" value=":memory:"/>
</php>
```

- **SQLite `:memory:`**: a database that lives entirely in RAM for the test run, extremely fast and automatically destroyed afterward.

> ⚠️ **Warning:** If `phpunit.xml` is misconfigured (or those lines are commented out) and your `.env` points to your real local MySQL database, `RefreshDatabase` will wipe your **real data**. Always confirm your test database setting before running the suite the first time. Never point tests at staging or production.

> ⚠️ **Warning:** SQLite and MySQL behave slightly differently (JSON functions, strict mode, certain index/constraint behaviors, date functions). If your app depends on MySQL-specific features, run your tests against a dedicated **MySQL test database** (for example `my_app_testing`) so tests match production. This is a real senior-level trade-off: SQLite is faster, MySQL is more faithful.

**Using a MySQL test database instead** (`phpunit.xml`):

```xml
<env name="DB_CONNECTION" value="mysql"/>
<env name="DB_DATABASE" value="my_app_testing"/>
```

---

## Factories in Tests

Recall factories from note 05. Tests are where they shine, since you can create realistic data in one line.

```php
$user = User::factory()->create();                  // saved to the DB
$user = User::factory()->make();                    // NOT saved, just an object
$articles = Article::factory()->count(5)->create();
$article = Article::factory()->for($user)->create();  // belongs to this user
```

**Factory states** are named variations, for readable tests:

```php
// database/factories/ArticleFactory.php
public function published(): static
{
    return $this->state(fn () => ['published_at' => now()]);
}
```

```php
$article = Article::factory()->published()->create();
```

- **`create()`** vs **`make()`**: `create()` inserts into the database; `make()` only builds the object in memory. Use `make()` when you just need the attribute values (for example, as POST data) without touching the database.

> 💡 **Tip:** Create only the data the test needs. A test that sets up 50 unrelated records is slow and hides what is actually being tested.

---

## Testing HTTP Endpoints

The base test class gives you a method for each HTTP verb:

| Method | Sends | Notes |
|---|---|---|
| `$this->get($uri)` | GET | Normal web request |
| `$this->post($uri, $data)` | POST | Form data |
| `$this->getJson($uri)` | GET | Adds JSON headers, ideal for APIs |
| `$this->postJson($uri, $data)` | POST | JSON body |
| `$this->putJson($uri, $data)` | PUT | JSON body |
| `$this->patchJson($uri, $data)` | PATCH | JSON body |
| `$this->deleteJson($uri)` | DELETE | |

```php
it('creates an article', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user, 'sanctum')
        ->postJson('/api/articles', [
            'title' => 'My First Post',
            'body'  => 'Hello world',
        ]);

    $response->assertCreated();   // 201
    $this->assertDatabaseHas('articles', ['title' => 'My First Post']);
});
```

> 💡 **Tip:** Always use the `...Json` variants for API tests. They send `Accept: application/json`, which makes Laravel return JSON error responses (such as a 422 for validation) instead of an HTML redirect, exactly like your real frontend would experience.

---

## Common Response Assertions

| Assertion | Checks |
|---|---|
| `assertOk()` | Status 200 |
| `assertCreated()` | Status 201 |
| `assertNoContent()` | Status 204 |
| `assertUnauthorized()` | Status 401 |
| `assertForbidden()` | Status 403 |
| `assertNotFound()` | Status 404 |
| `assertUnprocessable()` | Status 422 |
| `assertStatus(418)` | Any specific status |
| `assertJson([...])` | Response JSON contains this fragment |
| `assertJsonPath('data.title', 'X')` | A specific JSON path equals a value |
| `assertJsonCount(3, 'data')` | The `data` array holds 3 items |
| `assertJsonStructure([...])` | JSON has this shape (keys), ignoring values |
| `assertRedirect('/login')` | Redirected to a URL |

```php
$this->getJson('/api/articles/1')
    ->assertOk()
    ->assertJsonStructure([
        'data' => ['id', 'title', 'excerpt', 'published', 'created_at'],
    ]);
```

- **`assertJsonStructure`**: confirms the API contract (the shape) your frontend depends on. If someone accidentally renames `title` to `name`, this test fails immediately.

---

## Testing Authentication

**Web (session) login:**

```php
$this->actingAs($user)->get('/dashboard')->assertOk();
```

- **`actingAs($user)`**: pretends this user is logged in for the request, skipping the actual login steps. One of the most-used helpers in Laravel testing.

**API (Sanctum) login:**

```php
use Laravel\Sanctum\Sanctum;

Sanctum::actingAs(User::factory()->create(), ['*']);

$this->getJson('/api/user')->assertOk();
```

- **`Sanctum::actingAs($user, ['*'])`**: authenticates as that user via Sanctum. The second argument lists token abilities (from note 09); `['*']` means all abilities.

**Testing that unauthenticated access is blocked:**

```php
it('rejects guests', function () {
    $this->getJson('/api/articles')->assertUnauthorized();
});
```

**Testing the real login flow (token issuance):**

```php
it('logs in and returns a token', function () {
    $user = User::factory()->create(['password' => bcrypt('secret123')]);

    $this->postJson('/api/login', [
        'email' => $user->email,
        'password' => 'secret123',
    ])->assertOk()->assertJsonStructure(['user', 'token']);
});

it('rejects wrong credentials', function () {
    $user = User::factory()->create();

    $this->postJson('/api/login', [
        'email' => $user->email,
        'password' => 'wrong',
    ])->assertUnauthorized();
});
```

> 💡 **Tip:** Always test both the happy path and the failure path. Auth bugs usually hide in the failure path ("what happens with a wrong password? an expired token? a missing header?").

---

## Testing Validation

Recall note 08. Each important rule deserves a test.

```php
it('requires a title', function () {
    Sanctum::actingAs(User::factory()->create());

    $this->postJson('/api/articles', ['body' => 'No title here'])
        ->assertUnprocessable()
        ->assertJsonValidationErrors(['title']);
});
```

- **`assertJsonValidationErrors(['title'])`**: confirms the response contains a validation error for `title`.
- **`assertJsonMissingValidationErrors(['body'])`**: confirms a field did **not** have an error.

---

## Testing Authorization (Policies)

Recall note 10. Test both "allowed" and "denied" for every sensitive action.

```php
it('lets the owner update their article', function () {
    $owner = User::factory()->create();
    $article = Article::factory()->for($owner)->create();

    Sanctum::actingAs($owner);

    $this->putJson("/api/articles/{$article->id}", ['title' => 'Updated'])
        ->assertOk();
});

it('forbids other users from updating an article', function () {
    $article = Article::factory()->create();   // owned by someone else
    $stranger = User::factory()->create();

    Sanctum::actingAs($stranger);

    $this->putJson("/api/articles/{$article->id}", ['title' => 'Hacked'])
        ->assertForbidden();

    $this->assertDatabaseMissing('articles', ['title' => 'Hacked']);
});
```

> ⚠️ **Warning:** Authorization tests that only check the "allowed" case are nearly worthless. The real security value is in the **denied** case, proving a stranger cannot do the thing. Always assert the database was **not** changed too, not just that you got a 403.

---

## Database Assertions

| Assertion | Checks |
|---|---|
| `assertDatabaseHas('articles', [...])` | A row matching these values exists |
| `assertDatabaseMissing('articles', [...])` | No row matches |
| `assertDatabaseCount('articles', 3)` | The table has exactly 3 rows |
| `assertSoftDeleted($article)` | The row has `deleted_at` set (note 06) |
| `assertModelExists($article)` | The model's row still exists |
| `assertModelMissing($article)` | The model's row is gone |

```php
it('soft deletes an article', function () {
    $article = Article::factory()->create();
    Sanctum::actingAs($article->user);

    $this->deleteJson("/api/articles/{$article->id}")->assertNoContent();

    $this->assertSoftDeleted($article);
});
```

---

## Testing File Uploads

Recall note 11. You never want tests writing real files to your real disk or S3.

```php
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

it('uploads an avatar', function () {
    Storage::fake('public');                       // swap the disk for a temp fake
    Sanctum::actingAs($user = User::factory()->create());

    $file = UploadedFile::fake()->image('avatar.jpg', 200, 200);

    $this->postJson('/api/avatar', ['avatar' => $file])->assertOk();

    Storage::disk('public')->assertExists('avatars/' . $file->hashName());
});
```

- **`Storage::fake('public')`**: replaces the `public` disk with a temporary one that is cleaned up automatically.
- **`UploadedFile::fake()->image(...)`**: builds a fake image upload without needing a real file.
- **`assertExists` / `assertMissing`**: check whether a file ended up on the fake disk.

---

## Fakes: Mail, Queue, Events, HTTP

- **Fake**: Laravel's built-in stand-in for a real service. It records what *would* have happened, without actually sending an email, dispatching a job, or calling an external API.

```php
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\Queue;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Http;
```

**Mail** (full coverage in note 19):

```php
Mail::fake();

// ... perform the action that should send an email ...

Mail::assertSent(WelcomeMail::class);
Mail::assertNothingSent();
```

**Queue** (note 16):

```php
Queue::fake();

// ... action that dispatches a job ...

Queue::assertPushed(ProcessReport::class);
```

**Events:**

```php
Event::fake();

// ... action ...

Event::assertDispatched(OrderPlaced::class);
```

**External HTTP APIs** (payment gateways, SMS providers):

```php
Http::fake([
    'api.sms-provider.com/*' => Http::response(['status' => 'sent'], 200),
]);

// ... action that calls the SMS API ...

Http::assertSent(fn ($request) => $request->url() === 'https://api.sms-provider.com/send');
```

> ⚠️ **Warning:** A test that sends a real SMS or charges a real card every time the suite runs is a serious (and costly) mistake. Any code touching a third-party service must be faked in tests. You will use `Http::fake()` heavily in the SMS/OTP note (19).

> 💡 **Tip:** You can call `Http::preventStrayRequests()` at the start of a test (or globally in `tests/Pest.php`) so any request you forgot to fake throws an error, instead of silently hitting the real internet.

---

## Mocking Your Own Classes

When your code depends on a class you want to control, like a payment service, you can **mock** it (create a fake version with controlled behavior).

```php
use App\Services\PaymentService;
use Mockery\MockInterface;

it('marks an order paid when payment succeeds', function () {
    $this->mock(PaymentService::class, function (MockInterface $mock) {
        $mock->shouldReceive('charge')
            ->once()
            ->with(1000)
            ->andReturn(true);
    });

    // ... call the endpoint that uses PaymentService ...
});
```

- **`$this->mock(Class::class, ...)`**: replaces that class in Laravel's container for the duration of the test, so when your controller asks for `PaymentService`, it gets the mock.
- **`shouldReceive('charge')->once()->andReturn(true)`**: defines what the mock expects to be called with and what it returns.

> 💡 **Tip:** Mock at the **edges** of your system (external services, payment, SMS), not your own business logic. Over-mocking makes tests pass even when the real code is broken. This is also why the service-class pattern in note 15 helps: it gives you a clean seam to mock.

---

## Datasets: One Test, Many Inputs

Pest **datasets** run the same test with different inputs, avoiding copy-paste.

```php
it('rejects invalid emails', function (string $email) {
    $this->postJson('/api/register', [
        'name' => 'Test',
        'email' => $email,
        'password' => 'password123',
        'password_confirmation' => 'password123',
    ])->assertJsonValidationErrors(['email']);
})->with([
    'missing @' => 'not-an-email',
    'empty'     => '',
    'spaces'    => 'a b@c.com',
]);
```

- Each entry becomes its own separate test run, with the label shown in the output if it fails. (PHPUnit's equivalent is a *data provider*.)

---

## Unit Tests — When They Make Sense

Use unit tests for **pure logic** with no database or framework needed.

```php
// app/Support/PriceCalculator.php
class PriceCalculator
{
    public function withDiscount(float $price, int $percent): float
    {
        return round($price - ($price * $percent / 100), 2);
    }
}
```

```php
// tests/Unit/PriceCalculatorTest.php
use App\Support\PriceCalculator;

it('applies a percentage discount', function () {
    expect((new PriceCalculator)->withDiscount(200, 25))->toBe(150.0);
});
```

- **`expect($value)->toBe(...)`**: Pest's fluent assertion style. Common ones: `toBe()`, `toEqual()`, `toBeTrue()`, `toBeNull()`, `toHaveCount()`, `toContain()`.

> 💡 **Tip:** Don't unit-test controllers. Controllers just orchestrate framework features, so test them through feature tests where Laravel wires everything naturally.

---

## Architecture Tests

Pest can test the **structure** of your codebase, enforcing team rules automatically.

```php
arch('controllers do not touch the DB facade directly')
    ->expect('App\Http\Controllers')
    ->not->toUse('Illuminate\Support\Facades\DB');

arch('no debugging leftovers')
    ->expect(['dd', 'dump', 'ray'])
    ->not->toBeUsed();
```

- Such rules keep standards (from note 15) enforced by the machine instead of relying on code-review memory.

---

## Coverage, Parallel Runs & CI

- **Code coverage**: the percentage of your code lines executed while tests run. It shows what is *untested*, not whether tests are *good*.

```bash
php artisan test --coverage            # needs a coverage driver (Xdebug or PCOV)
php artisan test --coverage --min=80   # fail if coverage drops below 80%
php artisan test --parallel            # faster on larger suites
```

- **CI (Continuous Integration)**: a server (GitHub Actions, GitLab CI) that automatically runs your test suite on every push or pull request, blocking merges if tests fail.

A minimal GitHub Actions workflow:

```yaml
name: tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
      - run: composer install --no-interaction --prefer-dist
      - run: cp .env.example .env && php artisan key:generate
      - run: php artisan test
```

> ⚠️ **Warning:** Chasing 100% coverage is a trap. A test that merely executes lines without meaningful assertions gives false confidence. Aim for **meaningful tests of critical flows** (auth, payments, permissions, core business rules) rather than a vanity number.

---

## TDD — Test-Driven Development

- **TDD**: writing the test **before** the code, then writing just enough code to make it pass.

The cycle is called **Red → Green → Refactor**:

1. **Red**: write a test for a feature that doesn't exist yet. It fails.
2. **Green**: write the simplest code that makes it pass.
3. **Refactor**: clean up the code, re-running tests to make sure nothing broke.

> 💡 **Tip:** You don't have to practice strict TDD everywhere. A very practical habit: **when you find a bug, first write a failing test that reproduces it, then fix it.** That bug can then never silently return.

---

## What a Senior Test Suite Looks Like

| Habit | Why It Matters |
|---|---|
| Test behavior, not implementation | Tests survive refactors |
| One logical thing per test | A failure points to exactly one problem |
| Descriptive names (`it('forbids strangers from deleting an article')`) | The test list reads like documentation |
| Test failure paths (401, 403, 404, 422) | Most real bugs and security holes live there |
| Fake every external service | Fast, free, deterministic, safe |
| Independent tests | Order never matters; no test relies on another |
| Fast suite, run in CI on every push | Slow suites get ignored |
| Fix flaky tests immediately | A test that sometimes fails destroys trust |
| Tests for every bug fixed | Prevents regressions |

> 💡 **Tip:** A common beginner mistake is writing tests that depend on specific auto-increment IDs or on the current time. Use factories and `$this->travel()` / `Carbon::setTestNow()` to control time instead.

```php
$this->travelTo(now()->addDays(31));   // simulate 31 days later (e.g. test token or subscription expiry)
```

---

## Quick Revision

- Tests are automated checks that catch **regressions** and give you safety to change and refactor code.
- **Feature tests** (`tests/Feature`) boot the whole app and test endpoints end to end, so write mostly these. **Unit tests** (`tests/Unit`) test one isolated class and suit pure logic.
- **Pest** (readable `it('...')` functions) and **PHPUnit** (classes and methods) run the same underlying engine; concepts transfer one-to-one.
- Run tests with `php artisan test` (add `--filter`, `--parallel`, `--coverage`).
- Every test follows **Arrange → Act → Assert**.
- Use `RefreshDatabase` so tests never leak data into each other, and make sure `phpunit.xml` points at a **separate test database**, never your real one.
- SQLite `:memory:` is fast; a dedicated MySQL test database is more faithful to production. Choose deliberately.
- Create data with **factories** (`create()` saves, `make()` doesn't, states give readable variations).
- Use the `...Json` request helpers for APIs and assertions like `assertCreated()`, `assertForbidden()`, `assertUnprocessable()`, `assertJsonStructure()`, `assertDatabaseHas()`.
- Authenticate with `actingAs($user)` (sessions) or `Sanctum::actingAs($user)` (API tokens).
- Always test the **failure paths**: invalid input (422), unauthenticated (401), unauthorized (403), and confirm the database was not changed.
- **Fake** anything external or side-effecting: `Storage::fake()`, `Mail::fake()`, `Queue::fake()`, `Event::fake()`, `Http::fake()`. Never let tests hit real SMS, payment, or email services.
- Mock your own classes only at system edges; over-mocking hides real bugs.
- Datasets run one test with many inputs; architecture tests enforce code-structure rules automatically.
- Run the suite in **CI** on every push. Prioritize meaningful tests of critical flows over a vanity coverage number.
- When you find a bug, write a failing test first, then fix it, so it can never silently return.
- Next up (note 14): **Artisan CLI & Packages**, including writing your own custom Artisan commands, the scheduler, and a full command cheatsheet.