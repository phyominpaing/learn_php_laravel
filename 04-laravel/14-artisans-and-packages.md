# Laravel 14 — Artisan CLI & Packages

Artisan is Laravel's built-in command-line tool for generating code, managing the database, and running maintenance tasks, and Composer packages are the third-party libraries that extend what your app can do.

## Table of Contents

- [What Artisan Is and How to Explore It](#what-artisan-is-and-how-to-explore-it)
- [The `make:*` Generators](#the-make-generators)
- [Inspecting Your App](#inspecting-your-app)
- [Database Commands](#database-commands)
- [Cache & Optimization Commands](#cache--optimization-commands)
- [The `config:cache` and `env()` Trap](#the-configcache-and-env-trap)
- [Maintenance Mode](#maintenance-mode)
- [Writing Your Own Artisan Commands](#writing-your-own-artisan-commands)
- [Arguments, Options & Interactive Input](#arguments-options--interactive-input)
- [Command Output Helpers](#command-output-helpers)
- [The Task Scheduler](#the-task-scheduler)
- [Composer: The Package Manager](#composer-the-package-manager)
- [Versions & Semantic Versioning](#versions--semantic-versioning)
- [`composer.json` vs `composer.lock`](#composerjson-vs-composerlock)
- [Installing & Publishing Package Assets](#installing--publishing-package-assets)
- [How to Choose a Package Safely](#how-to-choose-a-package-safely)
- [Packages Every Laravel Developer Should Know](#packages-every-laravel-developer-should-know)
- [Artisan Command Cheatsheet](#artisan-command-cheatsheet)
- [Quick Revision](#quick-revision)

---

## What Artisan Is and How to Explore It

- **Artisan**: the command-line interface included with every Laravel project. It is the `artisan` file at your project root (note 01), and every command is run as `php artisan <command>`.
- **CLI (Command-Line Interface)**: a text-based way of controlling a program by typing commands instead of clicking buttons.

You never need to memorize every command. You only need to know how to **discover** them:

```bash
php artisan list                  # show every available command
php artisan list make             # show only commands starting with "make"
php artisan help migrate          # show full help for one command (all its options)
php artisan migrate --help        # same thing, different style
```

- **Argument**: a required or optional value after the command name (`make:model Product`, where `Product` is the argument).
- **Option**: a flag starting with `--` that changes behavior (`--force`, `--seed`). Some have shortcuts (`-m` instead of `--migration`).

> 💡 **Tip:** When you forget what a command does or which flags it has, run `php artisan help <command>`. Experienced developers do this constantly. Knowing *how to look it up* matters more than memorizing.

---

## The `make:*` Generators

Generators create correctly-named, correctly-placed files with boilerplate already filled in. You have already used several. Here is the full set you'll use regularly:

| Command | Creates | Covered In |
|---|---|---|
| `make:model Name` | Eloquent model (`-m -f -s -c` add extras, `--all` adds everything) | Note 06 |
| `make:migration create_x_table` | Database migration | Note 05 |
| `make:controller NameController` | Controller (`--resource`, `--api`, `--invokable`, `--model=`) | Note 03 |
| `make:request StoreXRequest` | Form Request (validation + authorize) | Note 08 |
| `make:resource XResource` | API Resource (JSON transformer) | Note 12 |
| `make:policy XPolicy --model=X` | Authorization policy | Note 10 |
| `make:middleware Name` | Middleware class | Note 09 |
| `make:seeder XSeeder` | Database seeder | Note 05 |
| `make:factory XFactory` | Model factory | Note 05 |
| `make:rule Name` | Custom validation rule | Note 08 |
| `make:component Name` | Blade component (class + view) | Note 04 |
| `make:test NameTest --pest` | Test file | Note 13 |
| `make:command Name` | Custom Artisan command | This note |
| `make:job Name` | Queued job | Note 16 |
| `make:event` / `make:listener` | Event and listener | Note 16 |
| `make:mail Name` | Mailable class | Note 19 |
| `make:notification Name` | Notification class | Note 19 |
| `make:provider Name` | Service provider | Note 15 |

> 💡 **Tip:** Because these commands follow naming conventions, a team's code stays consistent automatically. A new developer can predict where `PostPolicy` lives without asking anyone.

---

## Inspecting Your App

These commands let you **see** what Laravel knows about your application, which is invaluable for debugging.

```bash
php artisan about              # PHP/Laravel versions, environment, cache status, drivers
php artisan route:list         # every registered route (note 02)
php artisan model:show Product # columns, types, relationships, and observers for a model
php artisan db:show            # database connection info, tables, sizes
php artisan db:table users     # one table's columns, indexes, and foreign keys
php artisan env                # which environment is active (local, production...)
php artisan config:show database   # view the final merged config values for "database"
```

- **`about`**: first command to run on an unfamiliar project or a misbehaving server. It shows at a glance which environment, debug mode, and cache drivers are active.
- **`model:show`**: prints a model's table, attributes, and relationships, so you can verify what Eloquent actually sees.

> 💡 **Tip:** `php artisan about` on a production server is a quick way to confirm `APP_DEBUG` is `false` and the right environment is loaded. This takes 5 seconds and prevents a classic security mistake.

---

## Database Commands

You met most of these in note 05. Here is the full database toolbox in one place:

| Command | Purpose | Safe in Production? |
|---|---|---|
| `migrate` | Run pending migrations | Yes (with `--force`, since Laravel asks for confirmation in production) |
| `migrate:status` | Show which migrations ran | Yes |
| `migrate:rollback` | Undo the last batch | Careful |
| `migrate:fresh` | Drop ALL tables and re-migrate | **Never** |
| `migrate:fresh --seed` | Same, then seed | **Never** |
| `db:seed` | Run seeders | Careful |
| `db:wipe` | Drop all tables, views, and types | **Never** |
| `db:monitor` | Check connection counts | Yes |

> ⚠️ **Warning:** In production, Laravel prompts "Do you really wish to run this command?" for destructive commands. The `--force` flag skips that prompt, which is correct inside an automated deploy script for `php artisan migrate --force`, but **never** use `--force` casually with `migrate:fresh` on anything that holds real data.

---

## Cache & Optimization Commands

Laravel can pre-compute and store things it would otherwise rebuild on every request. This makes production faster.

```bash
php artisan config:cache     # merge all config files into one cached file
php artisan route:cache      # compile routes into one file (note 02)
php artisan view:cache       # pre-compile all Blade templates (note 04)
php artisan event:cache      # cache event/listener mappings
php artisan optimize         # runs config, event, route, and view caching together
```

And the matching "undo" commands:

```bash
php artisan config:clear
php artisan route:clear
php artisan view:clear
php artisan cache:clear      # clears the application cache (note 17)
php artisan optimize:clear   # clears ALL of the above in one go
```

| You Changed... | Run This |
|---|---|
| `.env` or a `config/*.php` file | `config:clear` (or `optimize:clear`) |
| A route | `route:clear` |
| A Blade file (while views are cached) | `view:clear` |
| Cached data (app cache) behaving oddly | `cache:clear` |
| "I don't know, something is stale" | `optimize:clear` |

> 💡 **Tip:** On local development, you generally **don't** cache anything, so your changes show immediately. On production deploys, run `php artisan optimize` as part of the deploy script. This ties directly into the deployment series you already studied.

---

## The `config:cache` and `env()` Trap

This is one of the most famous senior-level Laravel gotchas.

```php
// ❌ WRONG: calling env() directly inside your app code
$key = env('STRIPE_SECRET');

// ✅ RIGHT: read the value through a config file
// config/services.php
'stripe' => [
    'secret' => env('STRIPE_SECRET'),
],

// then in app code:
$key = config('services.stripe.secret');
```

- When you run `php artisan config:cache`, Laravel merges all config into one file and **stops loading `.env`**. After that, any `env()` call made *outside* a `config/` file returns `null`.

> ⚠️ **Warning:** This bug is invisible in local development (where config isn't cached) and then breaks production the moment you cache config: payments fail, emails don't send, API keys are "missing." The rule is simple and absolute: **only call `env()` inside `config/*.php` files; everywhere else, use `config('file.key')`.**

---

## Maintenance Mode

Take the site offline briefly (for example during a risky deploy) while showing visitors a friendly page.

```bash
php artisan down                          # site returns a 503 "Service Unavailable" page
php artisan down --secret="my-bypass-key" # you can still visit /my-bypass-key to access the site
php artisan down --retry=60               # tells browsers/bots to retry after 60 seconds
php artisan up                            # bring the site back online
```

- **`--secret`**: lets *you* (and your team) keep using the site while everyone else sees the maintenance page, which is useful for verifying a deploy before reopening to the public.

---

## Writing Your Own Artisan Commands

Any repetitive task (cleaning old data, generating reports, syncing with an API) can become a command. Commands can be run by hand, by the scheduler, or on a server cron job.

```bash
php artisan make:command DeleteExpiredTokens
```

This creates `app/Console/Commands/DeleteExpiredTokens.php`:

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\DB;

class DeleteExpiredTokens extends Command
{
    protected $signature = 'app:delete-expired-tokens {--days=30}';

    protected $description = 'Delete personal access tokens older than the given number of days';

    public function handle(): int
    {
        $days = (int) $this->option('days');

        $deleted = DB::table('personal_access_tokens')
            ->where('created_at', '<', now()->subDays($days))
            ->delete();

        $this->info("Deleted {$deleted} expired token(s).");

        return Command::SUCCESS;
    }
}
```

- **`$signature`**: the command's name and what input it accepts, written in a compact syntax. This one is run as `php artisan app:delete-expired-tokens --days=60`.
- **`$description`**: the sentence shown in `php artisan list`.
- **`handle()`**: the method that runs when the command executes. Put your logic here.
- **`return Command::SUCCESS;`**: returns exit code `0`. Return `Command::FAILURE` (exit code `1`) on error. Schedulers and CI pipelines use the exit code to know whether the command worked.

> 💡 **Tip:** In modern Laravel (11+), commands inside `app/Console/Commands/` are **auto-discovered**. No manual registration needed. Prefixing your own commands with `app:` is a common convention that keeps them grouped separately from framework commands in `php artisan list`.

---

## Arguments, Options & Interactive Input

**Signature syntax cheatsheet:**

| Signature Piece | Meaning | Example Call |
|---|---|---|
| `{user}` | Required argument | `php artisan x 5` |
| `{user?}` | Optional argument | `php artisan x` |
| `{user=guest}` | Optional argument with default | `php artisan x` |
| `{--force}` | Boolean flag (present or not) | `php artisan x --force` |
| `{--days=30}` | Option with a value and default | `php artisan x --days=7` |
| `{--Q|queue}` | Option with a shortcut | `php artisan x -Q` |
| `{ids*}` | Array of arguments | `php artisan x 1 2 3` |

**Reading input inside `handle()`:**

```php
$userId = $this->argument('user');
$force  = $this->option('force');
```

**Asking the person running the command questions:**

```php
$name = $this->ask('What is the user\'s name?');
$secret = $this->secret('Enter the admin password');   // input hidden
if ($this->confirm('This deletes data. Continue?')) {
    // proceed
}
$role = $this->choice('Pick a role', ['admin', 'editor', 'viewer'], 2);
```

> ⚠️ **Warning:** A command that asks questions will **hang forever** if run by the scheduler or a cron job, because no human is there to answer. Commands meant for automation must work entirely from arguments/options with sensible defaults, or use `--no-interaction`.

---

## Command Output Helpers

```php
$this->info('Everything went well.');     // green
$this->warn('Careful with this.');        // yellow
$this->error('Something failed.');        // red
$this->line('Plain text.');
$this->newLine();                          // blank line

$this->table(
    ['ID', 'Name'],
    User::select('id', 'name')->get()->toArray()
);

$this->withProgressBar($users, function ($user) {
    // process each user, a progress bar advances automatically
});
```

> 💡 **Tip:** For long-running commands, a progress bar and clear `info`/`error` messages turn a mysterious silent script into something a teammate (or future you) can trust.

---

## The Task Scheduler

Real apps need recurring jobs: send daily reports, clean old data, expire subscriptions. Traditionally, each task needs its own server **cron** entry, which gets messy and lives outside your code.

- **Cron**: a Linux program that runs commands at scheduled times.
- **Task Scheduler**: Laravel's way to define *all* your recurring tasks in PHP code, in your project (so they are version-controlled with Git), while the server only needs **one** cron entry.

**Define schedules in `routes/console.php`:**

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('app:delete-expired-tokens --days=30')->daily();

Schedule::command('app:send-weekly-report')->weeklyOn(1, '08:00');  // Mondays at 08:00

Schedule::call(function () {
    DB::table('failed_jobs')->where('failed_at', '<', now()->subWeek())->delete();
})->dailyAt('03:00');

Schedule::job(new ProcessPendingOrders)->everyFiveMinutes();
```

- **`Schedule::command(...)`**: schedules an Artisan command.
- **`Schedule::call(...)`**: schedules a closure.
- **`Schedule::job(...)`**: dispatches a queued job (note 16).

**Frequency methods:**

| Method | Runs |
|---|---|
| `->everyMinute()` | Every minute |
| `->everyFiveMinutes()` | Every 5 minutes |
| `->hourly()` | Top of every hour |
| `->daily()` | Every day at midnight |
| `->dailyAt('13:30')` | Every day at 1:30 PM |
| `->weekly()` | Sundays at midnight |
| `->weeklyOn(1, '8:00')` | Mondays at 8:00 |
| `->monthly()` | First of each month |
| `->cron('0 */6 * * *')` | Any custom cron expression |

**Important modifiers (senior-level):**

```php
Schedule::command('app:sync-inventory')
    ->hourly()
    ->withoutOverlapping()      // don't start if the previous run is still going
    ->onOneServer()             // in a multi-server setup, run on only ONE server
    ->runInBackground()         // don't block other scheduled tasks
    ->timezone('Asia/Yangon')   // interpret the time in this timezone
    ->emailOutputOnFailure('ops@example.com');
```

| Modifier | Why It Matters |
|---|---|
| `withoutOverlapping()` | Prevents two copies of a slow task colliding and corrupting data |
| `onOneServer()` | Prevents duplicate emails/charges when you run multiple web servers |
| `runInBackground()` | A slow task won't delay the others |
| `timezone()` | Avoids "why did this run at 2 AM?" confusion |
| `when(fn () => ...)` | Run only if a condition is true |
| `environments(['production'])` | Run only in specific environments |

**The ONE cron entry the server needs (production):**

```bash
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

- This tells the server: "every minute, run `schedule:run`." Laravel then checks its own schedule and runs whichever tasks are due.

**On your local machine** (no cron needed):

```bash
php artisan schedule:work     # runs in the foreground, checking every minute
php artisan schedule:list     # shows all scheduled tasks and when they run next
```

> ⚠️ **Warning:** A scheduled task silently never runs if the production cron entry was never added. When "my scheduled task isn't running," check this first, before debugging your code. `php artisan schedule:list` confirms Laravel *knows* about your task; only the cron entry makes it actually fire.

---

## Composer: The Package Manager

- **Composer**: PHP's dependency manager (note 01). It downloads libraries ("packages") your project needs, along with the libraries *those* need, and keeps them organized.
- **Package**: a reusable library published on **Packagist** (packagist.org, Composer's public registry). Names look like `vendor/package`, for example `spatie/laravel-permission`.
- **Dependency**: any package your project relies on.

**Everyday commands:**

```bash
composer require spatie/laravel-permission      # add a package to your project
composer require --dev laravel/pint             # add a package only needed for development
composer remove spatie/laravel-permission       # remove a package
composer install                                # install exactly what composer.lock says
composer update                                 # update packages within allowed version ranges
composer outdated                               # list packages with newer versions available
composer audit                                  # check installed packages for known security issues
composer dump-autoload                          # rebuild the class autoload map
```

| Command | When to Use |
|---|---|
| `composer require x` | Adding a new package |
| `composer install` | Setting up a project you cloned, or deploying (uses the lock file exactly) |
| `composer update` | Deliberately upgrading dependencies on your dev machine |
| `composer audit` | Regularly (and in CI) to catch vulnerable packages |

**Production install:**

```bash
composer install --no-dev --optimize-autoloader
```

- **`--no-dev`**: skips development-only packages (testing tools, debuggers), making the production install smaller and safer.
- **`--optimize-autoloader`**: pre-builds a faster class lookup map.

**Autoloading (PSR-4):** In `composer.json`, this block tells PHP how class names map to folders:

```json
"autoload": {
    "psr-4": {
        "App\\": "app/"
    }
}
```

- **PSR-4**: a PHP community standard saying a class's **namespace** mirrors its **folder path**. `App\Models\Product` must live at `app/Models/Product.php`. This is why a typo in a namespace or file name causes "Class not found" errors.

> 💡 **Tip:** After adding a brand-new class by hand (not via `make:`) or changing namespaces, if PHP says "Class not found," run `composer dump-autoload`. It rebuilds the map.

---

## Versions & Semantic Versioning

Package versions look like `2.4.1`, called **semantic versioning (SemVer)**: `MAJOR.MINOR.PATCH`.

| Part | Increases When... | Safe to Upgrade? |
|---|---|---|
| **Patch** (`2.4.1` → `2.4.2`) | Bug fixes only | Almost always |
| **Minor** (`2.4.x` → `2.5.0`) | New features, backward compatible | Usually |
| **Major** (`2.x` → `3.0.0`) | Breaking changes allowed | Read the upgrade guide first |

In `composer.json`:

```json
"require": {
    "php": "^8.2",
    "laravel/framework": "^12.0",
    "spatie/laravel-permission": "^6.0"
}
```

- **`^6.0` (caret)**: allows any version `>= 6.0.0` and `< 7.0.0`. Composer can update minor and patch versions but will never jump to a new major version automatically.

> ⚠️ **Warning:** Never run `composer update` on a production server. Dependencies could change underneath you, with untested results. Updates happen on your **dev machine**, get tested, then the updated `composer.lock` is committed and deployed.

---

## `composer.json` vs `composer.lock`

| File | Contains | Edited By |
|---|---|---|
| `composer.json` | The packages you *want* and acceptable version ranges (`^6.0`) | You (and `composer require`) |
| `composer.lock` | The **exact** versions actually installed (`6.4.1`) plus their dependencies | Composer only |

- `composer install` reads the **lock file**, so every developer and every server gets the *identical* versions.
- `composer update` re-resolves versions from `composer.json` and **rewrites** the lock file.

> 💡 **Tip:** **Always commit `composer.lock` to Git** for applications. It guarantees "it works on my machine" also means "it works on the server and on my teammate's machine." (`vendor/` is the opposite: never commit it, since it's regenerated from the lock file.)

---

## Installing & Publishing Package Assets

Many packages ship their own config files, migrations, or views. You **publish** them into your project so you can customize them.

```bash
composer require spatie/laravel-permission

php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
php artisan migrate
```

- **Service Provider**: a class (note 15) that a package uses to register its features with Laravel. Most packages auto-register through **package auto-discovery**, so installing them is just `composer require`.
- **`vendor:publish`**: copies a package's config/migration/view files from `vendor/` into your own project folders (`config/`, `database/migrations/`, `resources/views/vendor/`). You can then edit *your copy* safely, since `vendor/` itself is never edited (note 01).
- **`--tag=config`**: publish only one category of the package's files, when it defines tags.

> ⚠️ **Warning:** Never edit files inside `vendor/`. The next `composer install` overwrites your changes. Publish, then edit the published copy, or extend the package's classes in your own code.

---

## How to Choose a Package Safely

Every package you add is code you didn't write running inside your app, with access to your database and secrets. Senior engineers vet packages carefully.

| Check | Why |
|---|---|
| **Is it actively maintained?** (recent commits and releases) | Abandoned packages break on new Laravel/PHP versions |
| **Does it support your Laravel and PHP versions?** | Avoids upgrade dead ends |
| **How widely used is it?** (downloads, GitHub stars, known maintainers) | Popular packages get more review and faster security fixes |
| **Open issues and how maintainers respond** | Signals quality and support |
| **License** (MIT is the most permissive) | Avoids legal trouble in commercial work |
| **Do you really need it?** | Writing 20 lines yourself beats adding a dependency |
| **Run `composer audit`** | Detects known vulnerabilities |

> ⚠️ **Warning:** Typosquatting is real: malicious packages with names almost identical to popular ones (for example, a one-letter difference). Always copy the exact package name from the official docs, and double-check the vendor name before running `composer require`.

---

## Packages Every Laravel Developer Should Know

| Package | Purpose | Related Note |
|---|---|---|
| **laravel/sanctum** | API token and SPA authentication | 09, 12 |
| **laravel/fortify** | Headless auth backend including 2FA | 09 |
| **laravel/breeze** | Simple auth scaffolding with views | 09 |
| **spatie/laravel-permission** | Roles and permissions | 10 |
| **laravel/pint** | Automatic code style fixing (official) | 15 |
| **pestphp/pest** | Testing framework | 13 |
| **barryvdh/laravel-debugbar** | Debug toolbar showing queries, timings, memory (dev only) | 07 (spot N+1) |
| **laravel/telescope** | Detailed local debugging dashboard (requests, queries, jobs, mail) | 16 |
| **laravel/horizon** | Dashboard and supervision for Redis queues | 16 |
| **laravel/socialite** | "Login with Google/GitHub" OAuth | 09 |
| **laravel/scout** | Full-text search integration | 17 |
| **intervention/image** | Image resizing and processing | 11 |
| **larastan/larastan** | Static analysis that finds bugs without running code | 15 |
| **barryvdh/laravel-dompdf** | Generate PDFs | n/a |
| **maatwebsite/excel** | Import/export Excel and CSV | n/a |

> 💡 **Tip:** You don't need all of these. Start with the ones tied to problems you're actually solving, and install the dev-tools (**Pint**, **Debugbar**, **Pest**) with `--dev`. Reaching for a well-maintained package instead of reinventing PDF generation or permissions is a sign of good judgment, but only after confirming it passes the checklist above.

---

## Artisan Command Cheatsheet

Every command from the whole series so far, in one place:

| Category | Command | What It Does |
|---|---|---|
| **Run** | `serve` | Start local dev server |
| | `tinker` | Interactive PHP console (note 06) |
| | `about` | Environment summary |
| **Routes** | `route:list` | Show all routes |
| | `route:cache` / `route:clear` | Cache/clear routes |
| **Database** | `migrate` | Run migrations |
| | `migrate:status` | See what ran |
| | `migrate:rollback` | Undo last batch |
| | `migrate:fresh --seed` | Rebuild DB with data (dev only) |
| | `db:seed` | Run seeders |
| | `db:show` / `db:table` | Inspect database |
| **Models** | `model:show Name` | Inspect a model |
| **Generate** | `make:model/controller/request/resource/policy/...` | Scaffolding |
| **Cache** | `config:cache`, `view:cache`, `event:cache` | Production speedups |
| | `optimize` / `optimize:clear` | Cache all / clear all |
| | `cache:clear` | Clear application cache |
| **Storage** | `storage:link` | Public file symlink (note 11) |
| **Security** | `key:generate` | Create the app encryption key |
| **API** | `install:api` | Set up API routes and Sanctum (note 02) |
| **Scheduler** | `schedule:run` | Run due tasks (cron calls this) |
| | `schedule:work` | Local dev scheduler |
| | `schedule:list` | List scheduled tasks |
| **Queue** | `queue:work` | Process queued jobs (note 16) |
| **Maintenance** | `down` / `up` | Maintenance mode on/off |
| **Test** | `test` | Run your test suite (note 13) |
| **Package** | `vendor:publish` | Copy package files into your project |

---

## Quick Revision

- **Artisan** is Laravel's CLI (`php artisan ...`). Discover commands with `php artisan list` and `php artisan help <command>`, since looking things up matters more than memorizing.
- `make:*` generators create correctly named and placed files with boilerplate (models, controllers, requests, resources, policies, middleware, jobs, mail, and more), keeping team code consistent.
- Inspection commands (`about`, `route:list`, `model:show`, `db:show`, `db:table`) let you see what Laravel actually knows about your app. Run `about` on a server to confirm `APP_DEBUG` is off.
- Never run `migrate:fresh` or `db:wipe` on real data. `migrate --force` is the correct, safe-in-deploy-script form for applying new migrations.
- Production speed comes from `php artisan optimize` (config + route + view + event caches); `optimize:clear` undoes it all. Run the right clear command whenever config, routes, or views seem stale.
- **The `env()` trap:** after `config:cache`, `env()` outside `config/` files returns `null`. Only call `env()` in config files and read values everywhere else with `config('file.key')`.
- `php artisan down --secret=...` and `up` control maintenance mode, letting you keep access while visitors see a 503 page.
- Write **custom commands** with `make:command`: define `$signature` (name, arguments, options), `$description`, and `handle()`, returning `Command::SUCCESS` or `Command::FAILURE`. Commands that run via scheduler/cron must not ask interactive questions.
- The **Task Scheduler** lets you define all recurring tasks in `routes/console.php` using `Schedule::command/call/job` with frequency methods. The server needs exactly **one** cron entry: `* * * * * php artisan schedule:run`. A missing cron entry is the #1 reason scheduled tasks "never run."
- Use `withoutOverlapping()`, `onOneServer()`, `runInBackground()`, and `timezone()` for reliable, senior-grade scheduling. Use `schedule:work` locally and `schedule:list` to inspect.
- **Composer** manages packages: `require` to add, `install` to match the lock file exactly, `update` only on your dev machine, `audit` for security, `dump-autoload` when classes aren't found. Deploy with `composer install --no-dev --optimize-autoloader`.
- Versions follow **SemVer** (`MAJOR.MINOR.PATCH`); the caret `^6.0` allows minor/patch updates but never a new major.
- **Commit `composer.lock`** to Git (never `vendor/`). The lock file guarantees identical dependency versions everywhere. Never run `composer update` on production.
- `vendor:publish` copies a package's config/migrations/views into your project for safe customization. Never edit files inside `vendor/`.
- Vet every package: actively maintained, supports your versions, widely used, right license, and truly needed. Watch for typosquatted names and run `composer audit`.
- Next up (note 15): **Coding Standards & Real-World Project Structure**, including PSR standards, Laravel Pint, service classes, the repository pattern, DTOs, and clean architecture. This is the note that most directly separates mid-level from senior code.