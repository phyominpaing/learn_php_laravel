# PHP Predefined Variables

PHP comes with a set of **predefined variables** available automatically in every script — no need to declare or import them. The most important group is **Superglobals** — special arrays accessible everywhere in your code, regardless of scope. They are the backbone of handling web requests, sessions, user input, and server information.

---

## Table of Contents

1. [What are Predefined Variables?](#what-are-predefined-variables)
2. [What are Superglobals?](#what-are-superglobals)
3. [$GLOBALS](#globals)
4. [$_SERVER](#_server)
5. [$_GET](#_get)
6. [$_POST](#_post)
7. [$_FILES](#_files)
8. [$_REQUEST](#_request)
9. [$_SESSION](#_session)
10. [$_COOKIE](#_cookie)
11. [$_ENV](#_env)
12. [$php_errormsg](#php_errormsg)
13. [$http_response_header](#http_response_header)
14. [$argc & $argv](#argc--argv)
15. [Security Rules for Superglobals](#security-rules-for-superglobals)
16. [Quick Revision](#quick-revision)

---

## What are Predefined Variables?

- **Predefined variables** are variables that PHP creates and populates **automatically** before your script runs.
- You don't declare them — they just exist.
- They contain information about the request, server environment, user input, session data, and more.
- PHP's predefined variables fall into two groups:
  - **Superglobals** — available in **all scopes** (functions, classes, included files) without using the `global` keyword.
  - **Non-superglobal predefined variables** — available only in certain scopes (e.g., `$argc`/`$argv` in CLI scripts).

```php
<?php
// You never write this:
$_GET = [...];      // ❌ PHP already created it for you

// You just use it:
echo $_GET["name"]; // ✅ Available automatically
?>
```

---

## What are Superglobals?

- **Superglobals** are a special set of built-in arrays that are always accessible from **anywhere** in a PHP script — inside functions, classes, included files — without the `global` keyword.
- All superglobals start with `$_` (dollar sign + underscore).
- They are all **arrays** (except `$GLOBALS`).

### The Complete List of PHP Superglobals

| Superglobal | Contains |
|---|---|
| `$GLOBALS` | All global variables currently defined |
| `$_SERVER` | Web server and request environment info |
| `$_GET` | Data sent via URL query string |
| `$_POST` | Data sent via HTTP POST (forms) |
| `$_FILES` | Data about uploaded files |
| `$_REQUEST` | Combined `$_GET` + `$_POST` + `$_COOKIE` |
| `$_SESSION` | Session data for the current user |
| `$_COOKIE` | Cookie data sent by the browser |
| `$_ENV` | Environment variables from the OS |

> 💡 **Why "super"global?** Regular global variables need the `global` keyword to be accessed inside a function. Superglobals bypass this requirement — they're always in scope, no matter where your code runs.

```php
<?php
$username = "Phyo";  // Regular global variable

function test() {
    // global $username;  // ← needed for regular global variables
    echo $username;       // ❌ Undefined — regular globals need 'global'

    echo $_SERVER["PHP_SELF"];  // ✅ Works without anything — superglobal
}
?>
```

---

## $GLOBALS

- A special associative array that holds **references to all global variables** currently defined in the script.
- The variable name (without `$`) is the key; the value is the variable's value.
- Changes made via `$GLOBALS["var"]` affect the actual global variable.

```php
<?php
$name  = "Phyo";
$age   = 25;
$score = 99;

// Access global variables from anywhere via $GLOBALS
function showUser() {
    // No 'global' keyword needed
    echo $GLOBALS["name"];   // Output: Phyo
    echo $GLOBALS["age"];    // Output: 25

    // Modify a global variable
    $GLOBALS["score"] = 100;
}

showUser();
echo $score;  // Output: 100  ← modified via $GLOBALS

// $GLOBALS also contains itself!
echo $GLOBALS["GLOBALS"]["name"];  // Output: Phyo

// Listing all global variables
print_r(array_keys($GLOBALS));
// Array of all current global variable names
?>
```

> ⚠️ **Warning:** While `$GLOBALS` is convenient, using it extensively leads to **tightly coupled** code that's hard to test and debug. Always prefer passing variables as function parameters and returning results — use `$GLOBALS` only when truly necessary (e.g., in legacy code or specific framework patterns).

---

## $_SERVER

- Contains information about the **web server, current request, HTTP headers, and script environment**.
- One of the most important superglobals for backend development.
- Values are populated by the web server (Apache, Nginx, etc.) — not all keys are available in every environment.

```php
<?php
// Script and path information
echo $_SERVER["PHP_SELF"];        // /folder/script.php — current script path
echo $_SERVER["SCRIPT_FILENAME"]; // /var/www/html/folder/script.php — full file path
echo $_SERVER["DOCUMENT_ROOT"];   // /var/www/html — root directory of the web server

// Request information
echo $_SERVER["REQUEST_METHOD"];  // GET, POST, PUT, DELETE, etc.
echo $_SERVER["REQUEST_URI"];     // /folder/script.php?name=Phyo — full requested URI
echo $_SERVER["QUERY_STRING"];    // name=Phyo — everything after the "?"

// HTTP headers sent by the browser
echo $_SERVER["HTTP_HOST"];       // example.com — the domain
echo $_SERVER["HTTP_USER_AGENT"]; // Browser info: "Mozilla/5.0 ..."
echo $_SERVER["HTTP_REFERER"];    // URL of the page that linked here (if set)
echo $_SERVER["HTTP_ACCEPT_LANGUAGE"]; // en-US,en;q=0.9

// Client information
echo $_SERVER["REMOTE_ADDR"];     // User's IP address: e.g. 203.0.113.42
echo $_SERVER["REMOTE_PORT"];     // Port used by the client

// Server information
echo $_SERVER["SERVER_NAME"];     // example.com
echo $_SERVER["SERVER_ADDR"];     // Server's IP address
echo $_SERVER["SERVER_PORT"];     // 80 or 443
echo $_SERVER["SERVER_SOFTWARE"]; // Apache/2.4.51 or nginx/1.21.3

// HTTPS detection
echo $_SERVER["HTTPS"];           // "on" if HTTPS, empty/unset if HTTP
?>
```

---

### Common `$_SERVER` Practical Patterns

```php
<?php
// 1. Get the current full URL
function getCurrentUrl() {
    $protocol = (!empty($_SERVER["HTTPS"]) && $_SERVER["HTTPS"] !== "off") ? "https" : "http";
    $host     = $_SERVER["HTTP_HOST"];
    $uri      = $_SERVER["REQUEST_URI"];
    return "$protocol://$host$uri";
}
echo getCurrentUrl();
// Output: https://example.com/page.php?id=5

// 2. Check if the request is POST or GET
if ($_SERVER["REQUEST_METHOD"] === "POST") {
    echo "Form was submitted";
} else {
    echo "Page was visited via GET";
}

// 3. Get the visitor's real IP (considering proxies)
function getClientIP() {
    if (!empty($_SERVER["HTTP_X_FORWARDED_FOR"])) {
        return explode(",", $_SERVER["HTTP_X_FORWARDED_FOR"])[0];
    }
    return $_SERVER["REMOTE_ADDR"];
}
echo getClientIP();

// 4. Detect HTTPS
if (!empty($_SERVER["HTTPS"]) && $_SERVER["HTTPS"] === "on") {
    echo "Secure connection";
} else {
    echo "Not secure — redirect to HTTPS!";
}

// 5. Simple CSRF-style referrer check
function isFromOwnSite() {
    return isset($_SERVER["HTTP_REFERER"]) &&
           strpos($_SERVER["HTTP_REFERER"], $_SERVER["HTTP_HOST"]) !== false;
}
?>
```

### Most Useful `$_SERVER` Keys at a Glance

| Key | Contains |
|---|---|
| `PHP_SELF` | Path of the current script |
| `REQUEST_METHOD` | HTTP method (`GET`, `POST`, etc.) |
| `REQUEST_URI` | Full URI including query string |
| `QUERY_STRING` | Part after `?` in the URL |
| `HTTP_HOST` | Domain name (e.g., `example.com`) |
| `HTTPS` | `"on"` if HTTPS, empty if HTTP |
| `REMOTE_ADDR` | Visitor's IP address |
| `HTTP_USER_AGENT` | Browser/client identity string |
| `HTTP_REFERER` | URL that linked to this page |
| `SERVER_NAME` | Server's hostname |
| `DOCUMENT_ROOT` | Web server's root directory |

---

## $_GET

- Contains data sent in the **URL query string** — the part after `?` in a URL.
- Used for sending small amounts of **non-sensitive** data (search terms, page numbers, filters, IDs).
- Data is **visible in the URL** — never use GET for passwords or sensitive info.

```php
<?php
// URL: https://example.com/search.php?query=php&page=2&sort=asc

echo $_GET["query"];   // Output: php
echo $_GET["page"];    // Output: 2
echo $_GET["sort"];    // Output: asc

// Safe access — use ?? to provide a default if the key doesn't exist
$query = $_GET["query"] ?? "";
$page  = $_GET["page"]  ?? 1;
$sort  = $_GET["sort"]  ?? "asc";

// Always sanitize before using
$query = htmlspecialchars($_GET["query"] ?? "", ENT_QUOTES, "UTF-8");

// Check if a key exists before using it
if (isset($_GET["id"])) {
    $id = (int) $_GET["id"];  // Cast to int for numeric IDs
    echo "Showing product #$id";
}

// Practical use: pagination
// URL: page.php?page=3
$currentPage = max(1, (int)($_GET["page"] ?? 1));  // Always at least 1
echo "Page: $currentPage";  // Output: Page: 3
?>
```

> ⚠️ **Warning:** **Never trust `$_GET` data directly.** Users can type anything into the URL bar. Always:
> - Validate the data (is it the right type/format?)
> - Sanitize for output with `htmlspecialchars()`
> - Use parameterized queries when inserting into databases (never concatenate `$_GET` values directly into SQL)

---

## $_POST

- Contains data sent via **HTTP POST method** — typically from HTML form submissions.
- Data is sent in the **request body**, not the URL — not visible in the browser address bar.
- Still not encrypted unless using HTTPS — use HTTPS for any sensitive data.

```php
<?php
// HTML form (in another file):
// <form method="POST" action="login.php">
//   <input name="username" type="text">
//   <input name="password" type="password">
//   <button type="submit">Login</button>
// </form>

// login.php
if ($_SERVER["REQUEST_METHOD"] === "POST") {

    // Safe access with null coalescing
    $username = $_POST["username"] ?? "";
    $password = $_POST["password"] ?? "";

    // Basic validation
    if (empty($username) || empty($password)) {
        die("Username and password are required.");
    }

    // Sanitize for display
    $safeUsername = htmlspecialchars($username, ENT_QUOTES, "UTF-8");
    echo "Welcome, $safeUsername!";

    // For passwords — use password_hash() (NEVER store plain text)
    $hashedPassword = password_hash($password, PASSWORD_DEFAULT);

    // For DB queries — use prepared statements (never use $_POST directly in SQL!)
    // $stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
    // $stmt->execute([$username]);
}
?>
```

---

### GET vs POST — When to Use Which

| Feature | `$_GET` | `$_POST` |
|---|---|---|
| Data location | URL query string (visible) | Request body (hidden from URL) |
| Bookmarkable | ✅ Yes | ❌ No |
| Data size limit | ~2000 characters | Much larger (server-dependent) |
| Browser history | ✅ Saved in history | ❌ Not saved |
| Safe for sensitive data | ❌ Never | ✅ Better (still needs HTTPS) |
| Use for | Search, filters, pagination, IDs | Login, registration, form submissions |

---

## $_FILES

- Contains information about **files uploaded** via an HTML form.
- The form must have `enctype="multipart/form-data"` and the input must be `type="file"`.
- Each uploaded file's info is stored as a nested array.

```php
<?php
// HTML form:
// <form method="POST" enctype="multipart/form-data">
//   <input type="file" name="photo">
//   <button type="submit">Upload</button>
// </form>

// Accessing a single file upload
print_r($_FILES["photo"]);
// Array (
//   [name]     => profile.jpg          ← original filename from user's computer
//   [type]     => image/jpeg           ← MIME type reported by browser
//   [tmp_name] => /tmp/phpAbc123       ← temporary file path on the server
//   [error]    => 0                    ← 0 means no error (UPLOAD_ERR_OK)
//   [size]     => 204800               ← file size in bytes
// )

// Complete, safe file upload handler
function handleUpload(string $inputName, string $uploadDir, array $allowedTypes, int $maxBytes): string|false {
    if (!isset($_FILES[$inputName]) || $_FILES[$inputName]["error"] !== UPLOAD_ERR_OK) {
        return false;
    }

    $file    = $_FILES[$inputName];
    $tmpPath = $file["tmp_name"];

    // Validate MIME type using finfo (don't trust $_FILES["type"] — it can be faked!)
    $finfo    = finfo_open(FILEINFO_MIME_TYPE);
    $mimeType = finfo_file($finfo, $tmpPath);
    finfo_close($finfo);

    if (!in_array($mimeType, $allowedTypes, true)) {
        return false;  // Rejected — wrong file type
    }

    // Validate file size
    if ($file["size"] > $maxBytes) {
        return false;  // Rejected — too large
    }

    // Generate a safe unique filename (never use the original filename directly!)
    $ext         = pathinfo($file["name"], PATHINFO_EXTENSION);
    $safeFile    = bin2hex(random_bytes(16)) . "." . strtolower($ext);
    $destination = rtrim($uploadDir, "/") . "/" . $safeFile;

    // Move from temp location to final destination
    if (!move_uploaded_file($tmpPath, $destination)) {
        return false;  // Move failed
    }

    return $safeFile;  // Return the saved filename
}

$saved = handleUpload(
    "photo",
    "/var/www/uploads",
    ["image/jpeg", "image/png", "image/webp"],
    5 * 1024 * 1024  // 5 MB
);

echo $saved ? "Uploaded: $saved" : "Upload failed";

// Upload error codes
$errorCodes = [
    UPLOAD_ERR_OK         => "No error",
    UPLOAD_ERR_INI_SIZE   => "File exceeds upload_max_filesize in php.ini",
    UPLOAD_ERR_FORM_SIZE  => "File exceeds MAX_FILE_SIZE in the HTML form",
    UPLOAD_ERR_PARTIAL    => "File only partially uploaded",
    UPLOAD_ERR_NO_FILE    => "No file was uploaded",
    UPLOAD_ERR_NO_TMP_DIR => "Missing temporary folder",
    UPLOAD_ERR_CANT_WRITE => "Failed to write file to disk",
    UPLOAD_ERR_EXTENSION  => "A PHP extension stopped the upload",
];

$errorCode = $_FILES["photo"]["error"] ?? -1;
echo $errorCodes[$errorCode] ?? "Unknown error";
?>
```

> ⚠️ **Critical Security Warning:** File uploads are one of the most dangerous features in PHP. Always:
> - Validate the **actual** MIME type with `finfo` (not `$_FILES["type"]` — users can fake it)
> - **Never** use the original filename from `$_FILES["name"]` — generate your own safe filename
> - Store uploaded files **outside** the web root or disable PHP execution in the upload folder
> - Set maximum upload size in both PHP (`upload_max_filesize`) and your form

---

## $_REQUEST

- A **combined** superglobal that merges `$_GET`, `$_POST`, and `$_COOKIE` into one array.
- The merge order (and which overwrites which on key collision) is controlled by the `request_order` setting in `php.ini` (default: `GP` — GET then POST, POST overwrites).

```php
<?php
// Accessing data without caring if it came from GET or POST
$username = $_REQUEST["username"] ?? "";
echo $username;

// URL: page.php?name=Phyo
// POST body: name=Alice
// $_REQUEST["name"] would be "Alice" (POST wins by default)
?>
```

> ⚠️ **Warning:** Avoid using `$_REQUEST` in most cases. Its ambiguity makes code harder to understand and debug — you can't tell whether the data came from a form (POST) or the URL (GET). Prefer `$_GET` or `$_POST` explicitly so your intent is always clear. `$_REQUEST` is occasionally useful for simple scripts where the source doesn't matter.

---

## $_SESSION

- Stores **user-specific data** across multiple pages and requests.
- Unlike cookies (which live in the browser), session data is stored **on the server** — the browser only stores a small session ID cookie to identify the user.
- You must call `session_start()` at the **top of every page** that uses sessions — before any output.

```php
<?php
session_start();  // ← Must be first, before any HTML or echo

// Setting session variables
$_SESSION["user_id"]   = 42;
$_SESSION["username"]  = "Phyo";
$_SESSION["logged_in"] = true;
$_SESSION["role"]      = "admin";

echo $_SESSION["username"];  // Output: Phyo
?>
```

```php
<?php
// dashboard.php — a different page in the same session
session_start();

if (!isset($_SESSION["logged_in"]) || $_SESSION["logged_in"] !== true) {
    header("Location: login.php");  // Redirect if not logged in
    exit();
}

echo "Welcome, " . htmlspecialchars($_SESSION["username"]) . "!";
echo "Your role: " . $_SESSION["role"];
?>
```

```php
<?php
// logout.php
session_start();

// Method 1: remove specific session variables
unset($_SESSION["username"]);
unset($_SESSION["logged_in"]);

// Method 2: destroy the entire session (proper logout)
$_SESSION = [];             // Clear all session data
session_destroy();          // Destroy the session on the server
setcookie(session_name(), "", time() - 3600, "/");  // Clear the session cookie

header("Location: login.php");
exit();
?>
```

> 💡 **How sessions work under the hood:**
> 1. `session_start()` creates or resumes a session
> 2. PHP generates a unique **session ID** (e.g., `abc123xyz789`)
> 3. The session ID is stored as a cookie in the user's browser (named `PHPSESSID` by default)
> 4. Session DATA is stored on the **server** (in files or a database)
> 5. On the next request, the browser sends back the session ID cookie
> 6. PHP uses that ID to find and load the right session data

> ⚠️ **Security Tips for Sessions:**
> - Always call `session_regenerate_id(true)` after a successful login to prevent **session fixation attacks**
> - Set `session.cookie_httponly = true` in `php.ini` to prevent JavaScript from reading the session cookie
> - Set `session.cookie_secure = true` in `php.ini` to send the session cookie only over HTTPS

---

## $_COOKIE

- Contains **cookie data** sent by the browser in the current request.
- Cookies are small pieces of data stored in the **user's browser**, sent back to the server with every request.
- Set cookies with the `setcookie()` function — they appear in `$_COOKIE` only on the **next request** (not the same one).

```php
<?php
// Setting a cookie
setcookie(
    "theme",          // Name
    "dark",           // Value
    time() + 86400,   // Expiry: 1 day from now (86400 = 24 * 60 * 60 seconds)
    "/",              // Path: available on all pages
    "",               // Domain: current domain
    true,             // Secure: HTTPS only
    true              // HttpOnly: not accessible via JavaScript
);

// Reading a cookie (available on the NEXT request after setting)
$theme = $_COOKIE["theme"] ?? "light";  // default to "light" if not set
echo $theme;  // Output: dark

// Deleting a cookie (set its expiry in the past)
setcookie("theme", "", time() - 3600, "/");
?>
```

### Session vs Cookie

| Feature | `$_SESSION` | `$_COOKIE` |
|---|---|---|
| Stored | On the **server** | In the **browser** |
| Accessible to user | ❌ No (user sees only the session ID) | ✅ Yes (user can read and modify) |
| Size limit | Server disk/memory | ~4KB per cookie |
| Expires | When browser closes (or configured timeout) | Set explicitly with expiry time |
| Security | More secure (data server-side) | Less secure (data client-side) |
| Use for | Login state, cart, sensitive data | Preferences, remember-me, themes |

---

## $_ENV

- Contains **environment variables** passed from the server's operating system to PHP.
- Environment variables are key-value pairs set in the OS (or Docker, hosting panel, `.env` files loaded by a library).
- Commonly used to store **sensitive configuration** (database credentials, API keys) outside of your code.

```php
<?php
// Accessing environment variables
echo $_ENV["HOME"];          // /home/username (on Linux/macOS)
echo $_ENV["PATH"];          // System PATH variable
echo $_ENV["DB_HOST"];       // Your custom env variable
echo $_ENV["DB_PASSWORD"];   // Database password from environment

// Safer alternative — getenv() works even when variables_order in php.ini
// doesn't include 'E', so it's more reliable than $_ENV directly
echo getenv("DB_HOST");
echo getenv("HOME");

// Practical use: loading config from environment
$dbConfig = [
    "host"     => getenv("DB_HOST")     ?: "localhost",
    "user"     => getenv("DB_USER")     ?: "root",
    "password" => getenv("DB_PASSWORD") ?: "",
    "name"     => getenv("DB_NAME")     ?: "myapp",
];
?>
```

```bash
# Setting environment variables before running PHP (Linux/macOS)
export DB_HOST=localhost
export DB_PASSWORD=secret123
php index.php

# Or inline
DB_HOST=localhost DB_PASSWORD=secret123 php index.php
```

> 💡 **Best Practice:** In modern PHP projects, environment variables are typically managed using a `.env` file (loaded by a library like `vlucas/phpdotenv`) rather than relying on `$_ENV` directly. This approach is more portable and works consistently across different server environments.

```php
<?php
// Example using vlucas/phpdotenv (a common package)
require __DIR__ . "/vendor/autoload.php";

$dotenv = Dotenv\Dotenv::createImmutable(__DIR__);
$dotenv->load();

// Now getenv() and $_ENV work with values from your .env file
echo getenv("DB_HOST");
?>
```

---

## $php_errormsg

- Contains the **text of the last error message** generated by PHP.
- Only available in the local scope where the error occurred.
- Only populated when `track_errors` is enabled in `php.ini` — which is **deprecated** in PHP 7.2 and **removed** in PHP 8.0.

```php
<?php
// OLD WAY — only worked with track_errors=On (removed in PHP 8)
@file_get_contents("nonexistent.txt");
echo $php_errormsg;  // ⚠️ Deprecated/removed — don't use this

// ✅ MODERN WAY — use error_get_last() instead
@file_get_contents("nonexistent.txt");
$error = error_get_last();
if ($error !== null) {
    echo "Error: " . $error["message"];
    // Output: Error: failed to open stream: No such file or directory
}
?>
```

> ⚠️ **Note:** `$php_errormsg` is no longer available in PHP 8+. Use `error_get_last()` for the same purpose. It's included here for completeness since you may encounter it in legacy code.

---

## $http_response_header

- After making an **HTTP request** using PHP's file functions (like `file_get_contents()` with a URL), this variable gets populated with the **HTTP response headers** from that request.
- Automatically created in the local scope after a successful HTTP file function call.

```php
<?php
// After using file_get_contents() with an HTTP URL
$response = file_get_contents("https://httpbin.org/get");

// $http_response_header is automatically populated
print_r($http_response_header);
// Array (
//   [0] => HTTP/1.1 200 OK
//   [1] => Content-Type: application/json
//   [2] => Content-Length: 308
//   ...
// )

// Practical use: check the HTTP status code
$statusLine = $http_response_header[0];
echo $statusLine;  // Output: HTTP/1.1 200 OK

// Extract just the status code
preg_match('/\d{3}/', $statusLine, $matches);
echo $matches[0];  // Output: 200
?>
```

> 💡 **Note:** `$http_response_header` is a niche variable most commonly seen in legacy or simple scripts. In modern PHP, HTTP requests are typically made using **cURL** (`curl_*` functions) or the **Guzzle** HTTP library, both of which give you more control over headers and responses.

---

## $argc & $argv

- **CLI-only** variables — only available when running PHP from the **command line**, not through a web server.
- `$argc` — **argument count**: the number of arguments passed to the script (always at least `1` — the script name itself counts).
- `$argv` — **argument vector**: an array of all the arguments, where `$argv[0]` is always the script name.

```php
<?php
// script.php
echo "Number of arguments: " . $argc . "\n";
print_r($argv);
```

```bash
# Running from the command line:
php script.php hello world 123

# Output:
# Number of arguments: 4
# Array
# (
#     [0] => script.php   ← always the script name
#     [1] => hello
#     [2] => world
#     [3] => 123
# )
```

```php
<?php
// Practical use: a simple CLI tool
// Usage: php backup.php --database mydb --output /var/backups

if ($argc < 2) {
    echo "Usage: php " . $argv[0] . " [options]\n";
    exit(1);
}

// Parse arguments
$args = [];
for ($i = 1; $i < $argc; $i++) {
    if (str_starts_with($argv[$i], "--")) {
        $key = ltrim($argv[$i], "--");
        $args[$key] = $argv[$i + 1] ?? true;
        $i++;  // Skip the next value (already consumed)
    }
}

$database = $args["database"] ?? "default_db";
$output   = $args["output"]   ?? "/tmp";

echo "Backing up database: $database\n";
echo "Output directory: $output\n";
// Output:
// Backing up database: mydb
// Output directory: /var/backups
?>
```

> 💡 **Tip:** `$argc` and `$argv` are the PHP equivalent of `process.argv` in Node.js or `sys.argv` in Python. They're essential for writing CLI tools, cron scripts, and server-side utilities in PHP.

---

## Security Rules for Superglobals

Since superglobals contain data from the outside world (users, browsers, URLs), they are the **primary attack surface** of any PHP application. Follow these rules religiously:

### Rule 1 — Never Trust User Input

```php
<?php
// ❌ Dangerous — using raw superglobal data directly
echo $_GET["name"];                   // XSS vulnerability
$id = $_GET["id"];
$sql = "SELECT * FROM users WHERE id = $id";  // SQL injection!

// ✅ Safe — sanitize output, use prepared statements
echo htmlspecialchars($_GET["name"] ?? "", ENT_QUOTES, "UTF-8");
$id  = (int)($_GET["id"] ?? 0);
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$id]);
?>
```

---

### Rule 2 — Validate Before Using

```php
<?php
function getValidatedPage(): int {
    $page = $_GET["page"] ?? 1;

    // Type check + range check
    if (!is_numeric($page) || (int)$page < 1) {
        return 1;  // Default to page 1 if invalid
    }

    return min((int)$page, 1000);  // Cap at 1000
}

function getValidatedRole(): string {
    $allowed = ["admin", "editor", "viewer"];
    $role    = $_POST["role"] ?? "";

    if (!in_array($role, $allowed, true)) {
        return "viewer";  // Default to lowest privilege
    }

    return $role;
}
?>
```

---

### Rule 3 — Use `??` for Safe Access

```php
<?php
// ❌ Causes Notice/Warning if key doesn't exist
echo $_GET["name"];

// ✅ Always use null coalescing to provide a safe default
echo $_GET["name"]    ?? "";
echo $_GET["page"]    ?? 1;
echo $_SESSION["uid"] ?? null;
echo $_COOKIE["pref"] ?? "default";
?>
```

---

### Rule 4 — Never Store Sensitive Data in Cookies or `$_GET`

```php
<?php
// ❌ WRONG — user can see and modify this in their browser
setcookie("user_role", "admin");    // User can change to "superadmin"!
$_GET["is_admin"] = true;           // Trivially bypassed in the URL

// ✅ CORRECT — sensitive data lives in the session (server-side)
$_SESSION["user_role"] = "admin";   // User can't tamper with this
?>
```

---

### Superglobal Security Summary

| Superglobal | Risk Level | Key Practice |
|---|---|---|
| `$_GET` | 🔴 High | Sanitize for output, validate types, never use in SQL directly |
| `$_POST` | 🔴 High | Same as GET; use `password_hash()` for passwords |
| `$_FILES` | 🔴 Very High | Validate MIME type with `finfo`, generate new filenames |
| `$_COOKIE` | 🟠 Medium | Validate before use; never store sensitive data |
| `$_SESSION` | 🟢 Lower | Regenerate ID on login; destroy properly on logout |
| `$_SERVER` | 🟡 Medium | Some values can be spoofed (e.g., `HTTP_REFERER`) |
| `$_ENV` / `$_REQUEST` | 🟡 Medium | Prefer `getenv()`; avoid `$_REQUEST` |

---

## Quick Revision

- **Predefined variables** are created by PHP automatically before your script runs — no need to declare them.
- **Superglobals** (`$_GET`, `$_POST`, etc.) are available **everywhere** — functions, classes, included files — no `global` keyword needed.
- **`$GLOBALS`** — holds all global variables by name as keys; modifying it modifies the actual globals.
- **`$_SERVER`** — server, request, and environment info. Most useful: `REQUEST_METHOD`, `HTTP_HOST`, `REMOTE_ADDR`, `HTTPS`, `REQUEST_URI`.
- **`$_GET`** — URL query string data. Visible in the URL, bookmarkable, limited size. Never use for passwords.
- **`$_POST`** — form submission data sent in the request body. Hidden from URL. Still needs HTTPS for true security.
- **`$_FILES`** — file upload data. Always validate MIME type with `finfo`, never use original filename, always check error code.
- **`$_REQUEST`** — merges GET + POST + COOKIE. Avoid it — prefer explicit `$_GET` or `$_POST` for clarity.
- **`$_SESSION`** — server-side user data. Requires `session_start()` at the top. Use `session_regenerate_id(true)` after login. Destroy on logout.
- **`$_COOKIE`** — browser-stored key-value pairs. Set with `setcookie()`, available on the next request. Use `Secure` and `HttpOnly` flags always.
- **`$_ENV`** — OS environment variables. Prefer `getenv()` for reliability; use `.env` files with `phpdotenv` in modern projects.
- **`$php_errormsg`** — removed in PHP 8. Use `error_get_last()` instead.
- **`$http_response_header`** — populated after `file_get_contents()` on an HTTP URL; contains response headers.
- **`$argc` / `$argv`** — CLI only. `$argc` = argument count, `$argv[0]` = script name, `$argv[1+]` = user arguments.
- **Golden security rule:** treat every superglobal value as **untrusted enemy input**. Validate type and format, sanitize for output, use prepared statements for DB queries, and never store sensitive data in cookies or GET parameters.