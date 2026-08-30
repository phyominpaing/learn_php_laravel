# Laravel 01 — Introduction & Project Setup

Laravel is a free, open-source **PHP web framework** that gives you a structured, batteries-included way to build backend applications — routing, database access, authentication, and more — instead of writing everything from raw PHP.

## Table of Contents

- [What Laravel Actually Is](#what-laravel-actually-is)
- [Why Use a Framework Instead of Plain PHP](#why-use-a-framework-instead-of-plain-php)
- [The MVC Pattern](#the-mvc-pattern)
- [Prerequisites Before Installing](#prerequisites-before-installing)
- [Installing PHP, Composer, and Laravel](#installing-php-composer-and-laravel)
- [Running the Development Server](#running-the-development-server)
- [The `.env` File](#the-env-file)
- [Full Folder Structure Deep-Dive](#full-folder-structure-deep-dive)
- [Laravel vs Plain PHP — Comparison](#laravel-vs-plain-php--comparison)
- [Quick Revision](#quick-revision)

---

## What Laravel Actually Is

- **Framework**: a pre-built skeleton of code and conventions that handles common, repetitive backend problems (routing URLs, talking to a database, rendering HTML, handling forms) so you only write the logic specific to *your* app.
- **PHP**: the server-side scripting language Laravel is written in and built on top of. You still need to know PHP — Laravel doesn't replace it, it organizes it.
- **Composer**: PHP's package manager. Laravel itself is installed and updated through Composer, and almost every Laravel package (email, PDF generation, auth, etc.) is pulled in the same way.
- **Artisan**: Laravel's built-in command-line tool. You'll use it constantly to generate files, run migrations, and start the dev server.

> 💡 **Tip:** If you've used Next.js, think of Laravel as PHP's equivalent of "a framework with strong opinions about where things go" — similar spirit to how Next.js organizes pages/, app/, and api routes.

---

## Why Use a Framework Instead of Plain PHP

In raw PHP, you'd manually:
- Parse the URL to figure out which page to show
- Escape every database query yourself to avoid SQL injection
- Handle sessions, cookies, and login state by hand
- Rebuild form validation logic in every single form

Laravel gives you all of this out of the box, tested and secure by default.

> ⚠️ **Warning:** Frameworks reduce boilerplate, but they don't remove the need to understand PHP fundamentals (arrays, functions, OOP, namespaces). If PHP basics are shaky, Laravel's "magic" will feel confusing instead of helpful.

---

## The MVC Pattern

Laravel is built around **MVC (Model–View–Controller)** — a way of separating concerns in your app.

| Layer | Responsibility | Laravel Location |
|---|---|---|
| **Model** | Represents and interacts with a database table (a "User", a "Post") | `app/Models` |
| **View** | The HTML/output shown to the user | `resources/views` |
| **Controller** | The logic that receives a request, talks to the Model, and returns a View | `app/Http/Controllers` |

- **Model**: a PHP class that maps to one database table and lets you query/insert/update it using PHP instead of raw SQL.
- **View**: a template file (Blade, Laravel's templating language) that outputs HTML, optionally with dynamic data injected in.
- **Controller**: the "traffic cop" — receives an incoming request (e.g., "show post #5"), asks the Model for the data, and hands it to a View to display.

**Typical flow of a request:**

```
Browser Request → Route → Controller → Model (database) → View (HTML) → Response back to Browser
```

> 💡 **Tip:** You don't have to fully understand MVC yet — it will click once we build actual routes and controllers in notes 02–03. Just remember: Model = data, View = display, Controller = logic connecting them.

---

## Prerequisites Before Installing

Before installing Laravel, your machine needs:

- **PHP** (version 8.2 or higher for current Laravel versions)
- **Composer** (PHP's dependency/package manager)
- A database — we'll use **MySQL** (via **phpMyAdmin** for a GUI, covered in note 05)
- A code editor (VS Code recommended)
- Optional but common: **Node.js + npm** (for compiling frontend assets like CSS/JS inside a Laravel project)

> ⚠️ **Warning:** Laravel version requirements change over time (each major version bumps the minimum PHP version). Always check the [official docs](https://laravel.com/docs) for the PHP version required by the Laravel version you're installing.

---

## Installing PHP, Composer, and Laravel

**Step 1 — Check if PHP is installed:**

```bash
php -v
```

If not installed, install PHP for your OS (on Ubuntu/Linux servers, this is covered in depth in your `deploy-##` series — same PHP install steps apply locally).

**Step 2 — Check/install Composer:**

```bash
composer -v
```

If missing, install it from [getcomposer.org](https://getcomposer.org).

**Step 3 — Create a new Laravel project:**

```bash
composer create-project laravel/laravel my-app
```

- `composer create-project` — tells Composer to download a fresh copy of a package (Laravel) and scaffold it into a new folder.
- `laravel/laravel` — the official "skeleton" Laravel package on Packagist (Composer's package registry).
- `my-app` — the folder name your new project will be created in.

**Step 4 — Move into the project folder:**

```bash
cd my-app
```

---

## Running the Development Server

```bash
php artisan serve
```

- **`artisan`** — Laravel's command-line tool, located at the root of every Laravel project.
- **`serve`** — an Artisan command that starts a lightweight local PHP web server, by default at `http://127.0.0.1:8000`.

Open that URL in your browser — you should see the default Laravel welcome page.

> 💡 **Tip:** `php artisan serve` is only for local development. In production, a real web server (Nginx/Apache) serves the app — this was covered in your `deploy-##` notes.

---

## The `.env` File

Every Laravel project has a `.env` file at the root — this stores **environment-specific configuration** (database credentials, app URL, API keys) separately from your code.

```env
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:xxxxxxxxxxxxxxxxxxxxxxxx
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=my_app_db
DB_USERNAME=root
DB_PASSWORD=
```

- **`APP_ENV`**: tells Laravel what environment it's running in (`local`, `production`, etc.) — affects error display and optimizations.
- **`APP_KEY`**: a random encryption key auto-generated for your app; used to encrypt sessions and other sensitive data.
- **`APP_DEBUG`**: when `true`, shows detailed error pages (great for development). Must be `false` in production.
- **`DB_*` values**: the database connection details — we configure these fully in note 05.

> ⚠️ **Warning:** Never commit your real `.env` file to Git/GitHub — it contains secrets. Laravel ships with a `.env.example` (no real secrets) for this reason, and `.env` is git-ignored by default.

---

## Full Folder Structure Deep-Dive

When you open a fresh Laravel project, here's every top-level item and what it's for:

```
my-app/
├── app/
├── bootstrap/
├── config/
├── database/
├── public/
├── resources/
├── routes/
├── storage/
├── tests/
├── vendor/
├── .env
├── artisan
├── composer.json
└── composer.lock
```

### `app/` — Your application's core code
- **`app/Models/`**: Eloquent Model classes — each one usually maps to a database table (e.g., `User.php` → `users` table).
- **`app/Http/Controllers/`**: Controller classes — handle incoming requests and return responses.
- **`app/Http/Middleware/`**: Middleware classes — code that runs before/after a request reaches your controller (e.g., "is this user logged in?").
- **`app/Providers/`**: Service Providers — where Laravel's core services and your own app's services get "bootstrapped" (registered) when the app starts.

### `bootstrap/`
- Contains framework bootstrap files and a `cache/` folder Laravel uses to cache framework files for performance. You rarely touch this directly.

### `config/`
- Every `.php` file here holds configuration for one part of the app (`database.php`, `mail.php`, `app.php`, etc.). Most values here actually pull from your `.env` file behind the scenes.

### `database/`
- **`database/migrations/`**: version-controlled files that define your database table structures in PHP code (instead of manually creating tables in phpMyAdmin).
- **`database/seeders/`**: scripts to fill your database with sample/test data.
- **`database/factories/`**: blueprints for generating fake model data (useful for testing).

### `public/`
- The **only folder a web server should point to directly**. Contains `index.php` — the single entry point every request goes through — plus compiled CSS/JS and public assets like images.

### `resources/`
- **`resources/views/`**: Blade template files (`.blade.php`) — your HTML output.
- **`resources/css/` and `resources/js/`**: raw, uncompiled frontend source files (before being built by Vite/npm).

### `routes/`
- **`routes/web.php`**: routes for normal browser-facing pages (returns HTML views).
- **`routes/api.php`**: routes for API endpoints (returns JSON, typically for a frontend SPA or mobile app to consume).

### `storage/`
- **`storage/app/`**: user-uploaded files.
- **`storage/framework/`**: cache, session, and compiled view files Laravel generates automatically.
- **`storage/logs/`**: application error/log files (`laravel.log`) — the first place to check when something breaks.

### `tests/`
- Automated test files (unit tests, feature tests) — covered fully in note 13.

### `vendor/`
- All installed Composer packages/dependencies, including Laravel's own core framework code. Never edit this folder — it's regenerated by `composer install`.

### Root files
- **`artisan`**: the CLI entry script (`php artisan ...`).
- **`composer.json`**: lists your project's PHP dependencies and their versions (like `package.json` in Node).
- **`composer.lock`**: locks exact installed versions for consistency across machines — similar to `package-lock.json`.
- **`.env`**: local environment configuration (see above).

> 💡 **Tip:** As a frontend dev coming from React/Next.js: `routes/` ≈ your router config, `resources/views` ≈ your pages/components, `app/Models` ≈ your data-layer/API-client equivalent, and `public/` ≈ the `public/` folder you already know from React apps.

---

## Laravel vs Plain PHP — Comparison

| Aspect | Plain PHP | Laravel |
|---|---|---|
| Routing | Manual `if ($_SERVER['REQUEST_URI'] == ...)` logic | Declarative `routes/web.php` file |
| Database queries | Raw SQL strings, manual escaping | Eloquent ORM + Query Builder |
| Templating | Mixing PHP and HTML directly | Blade templates with clean syntax |
| Security (CSRF, XSS) | You implement it yourself | Built-in protections by default |
| Project structure | Whatever you decide | Enforced, consistent convention |
| Package management | Manual downloads/includes | Composer |

---

## Quick Revision

- Laravel is a PHP framework that organizes backend code using **MVC**: Models handle data, Views handle display, Controllers connect the two.
- You install Laravel through **Composer** (`composer create-project laravel/laravel my-app`), and run it locally with `php artisan serve`.
- The **`.env`** file holds environment-specific secrets and settings (database credentials, app key, debug mode) and should never be committed to Git.
- **`app/`** holds your core logic (Models, Controllers, Middleware), **`routes/`** defines URLs, **`resources/views/`** holds Blade templates, **`database/migrations/`** defines your table structures in code, and **`public/`** is the only folder the web server exposes directly.
- **`artisan`** is Laravel's CLI tool — you'll use it constantly to generate files and run commands.
- Never edit the **`vendor/`** folder — it's managed entirely by Composer.
- Next up (note 02): Laravel **routing** — how URLs map to code.