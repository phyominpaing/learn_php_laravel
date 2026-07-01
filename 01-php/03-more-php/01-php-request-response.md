# PHP Request & Response

This note focuses specifically on how **PHP itself** handles incoming requests and builds outgoing responses — the practical, code-level companion to the HTTP concepts covered in the backend notes. Everything here is things you'll write directly in `.php` files.

---

## Table of Contents

1. [Request](#request)
   - [HTML Form](#html-form)
   - [GET Request](#get-request)
   - [GET Request with Params](#get-request-with-params)
   - [POST Request](#post-request)
   - [File Request](#file-request)
   - [Postman](#postman)
   - [Request Header](#request-header)
2. [Response](#response)
   - [text/html](#texthtml)
   - [application/json](#applicationjson)
   - [image/jpeg](#imagejpeg)
   - [header("HTTP/1.1 statuscode some text")](#headerhttp11-statuscode-some-text)
   - [Location: somewhere](#location-somewhere)
3. [Putting It All Together](#putting-it-all-together)
4. [Quick Revision](#quick-revision)

---

## Request

A **request** is what the client (browser, Postman, mobile app, another server) sends to your PHP script. PHP gives you everything about the incoming request through superglobals (`$_GET`, `$_POST`, `$_FILES`, `$_SERVER`) and the raw input stream (`php://input`).

---

### HTML Form

- The most basic way a browser sends data to PHP — an HTML `<form>` element.
- The `method` attribute decides whether the data ends up in `$_GET` or `$_POST`.
- The `action` attribute is the URL the form submits to (can be left out to submit to the current page).

```html
<!-- form.html -->
<form method="POST" action="process.php">
    <label>Name: <input type="text" name="name"></label>
    <label>Email: <input type="email" name="email"></label>
    <button type="submit">Submit</button>
</form>
```

```php
<?php
// process.php — receives the form submission
$name  = $_POST["name"]  ?? "";
$email = $_POST["email"] ?? "";

echo "Received: $name, $email";
?>
```

> 💡 **Note:** A plain HTML form can only send `GET` or `POST` — there is no `method="PUT"` or `method="DELETE"` in HTML. For other methods, you need JavaScript (`fetch()`) or tools like Postman, covered below.

---

### GET Request

- A **GET request** asks the server to **retrieve** something. Data is sent through the **URL**, not the body.
- The simplest possible request — just visiting a URL in the browser is a GET request.

```php
<?php
// visited.php — accessed by simply going to: https://example.com/visited.php

echo "You visited this page with method: " . $_SERVER["REQUEST_METHOD"];
// Output: You visited this page with method: GET
?>
```

```html
<!-- A GET form -->
<form method="GET" action="search.php">
    <input type="text" name="query">
    <button type="submit">Search</button>
</form>
```

```php
<?php
// search.php
// Submitting the form above with "PHP" typed in → URL becomes:
// search.php?query=PHP

$query = $_GET["query"] ?? "";
echo "Searching for: " . htmlspecialchars($query);
?>
```

> 💡 **GET requests are visible, bookmarkable, and cacheable** — perfect for searches, filters, and pagination, but never for passwords or sensitive data (covered in the GET vs POST notes earlier).

---

### GET Request with Params

- **Params** (parameters) are the key=value pairs that appear after `?` in the URL, separated by `&`.
- This is how you pass multiple pieces of data through a GET request.

```
https://example.com/products.php?category=shoes&page=2&sort=price_asc
                                  └─────────┘ └─────┘ └──────────────┘
                                    param 1   param 2     param 3
```

```php
<?php
// products.php — handling multiple GET parameters

$category = $_GET["category"] ?? "all";
$page     = (int) ($_GET["page"] ?? 1);
$sort     = $_GET["sort"]     ?? "default";

echo "Category: $category\n";
echo "Page: $page\n";
echo "Sort: $sort\n";

// Output for the URL above:
// Category: shoes
// Page: 2
// Sort: price_asc
?>
```

```php
<?php
// Building a GET URL with parameters dynamically (e.g., for links/pagination)
$params = [
    "category" => "shoes",
    "page"     => 3,
    "sort"     => "price_asc",
];

$url = "products.php?" . http_build_query($params);
echo $url;
// Output: products.php?category=shoes&page=3&sort=price_asc

// http_build_query() automatically URL-encodes special characters
$params2 = ["query" => "men's shoes & boots"];
echo "search.php?" . http_build_query($params2);
// Output: search.php?query=men%27s+shoes+%26+boots
?>
```

> 💡 **Always use `http_build_query()`** when building URLs with dynamic values — it correctly encodes spaces, `&`, `?`, and other special characters so the URL stays valid.

---

### POST Request

- A **POST request** sends data in the **request body**, not the URL — used for creating resources, submitting forms, login, file uploads.
- Data is read via `$_POST` when the `Content-Type` is `application/x-www-form-urlencoded` or `multipart/form-data`.

```html
<form method="POST" action="register.php">
    <input type="text"     name="username">
    <input type="password" name="password">
    <input type="email"    name="email">
    <button type="submit">Register</button>
</form>
```

```php
<?php
// register.php
if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $username = trim($_POST["username"] ?? "");
    $password = $_POST["password"]      ?? "";
    $email    = trim($_POST["email"]    ?? "");

    // Basic validation
    if (empty($username) || empty($password) || empty($email)) {
        die("All fields are required.");
    }

    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        die("Invalid email address.");
    }

    // Hash the password — NEVER store plain text
    $hashedPassword = password_hash($password, PASSWORD_DEFAULT);

    echo "Registered: " . htmlspecialchars($username);
}
?>
```

### Reading JSON POST Bodies

- When a client sends `Content-Type: application/json` (common with `fetch()`, mobile apps, Postman), the data does **NOT** appear in `$_POST`.
- You must read it manually from the raw input stream `php://input`.

```php
<?php
// Reading a JSON request body — does NOT use $_POST!
header("Content-Type: application/json");

$rawBody = file_get_contents("php://input");
$data    = json_decode($rawBody, true);

if (json_last_error() !== JSON_ERROR_NONE) {
    http_response_code(400);
    echo json_encode(["error" => "Invalid JSON"]);
    exit();
}

$name  = $data["name"]  ?? "";
$email = $data["email"] ?? "";

echo json_encode(["received" => ["name" => $name, "email" => $email]]);

// Sent via Postman/fetch with:
// Content-Type: application/json
// Body: {"name": "Phyo", "email": "phyo@example.com"}
?>
```

> ⚠️ **Common Mistake:** Trying to access `$_POST["name"]` when the client sent JSON instead of form-encoded data — it will always be empty. Always check `Content-Type` and read `php://input` for JSON APIs.

---

### File Request

- Uploading a file requires the form to have `enctype="multipart/form-data"`.
- File data is accessed through the `$_FILES` superglobal — already covered in depth in the Predefined Variables notes, summarized here in the request context.

```html
<form method="POST" action="upload.php" enctype="multipart/form-data">
    <input type="file" name="document">
    <button type="submit">Upload</button>
</form>
```

```php
<?php
// upload.php
if (isset($_FILES["document"]) && $_FILES["document"]["error"] === UPLOAD_ERR_OK) {
    $tmpPath  = $_FILES["document"]["tmp_name"];
    $fileName = $_FILES["document"]["name"];
    $fileSize = $_FILES["document"]["size"];

    echo "Received file: $fileName ($fileSize bytes)\n";

    // Move it to a permanent location with a safe generated name
    $safeName = bin2hex(random_bytes(8)) . "_" . basename($fileName);
    move_uploaded_file($tmpPath, "/var/www/uploads/" . $safeName);

    echo "Saved as: $safeName";
} else {
    echo "Upload failed or no file sent.";
}
?>
```

> 💡 **Note:** Multiple file uploads use `name="documents[]"` (with square brackets) — this turns `$_FILES["documents"]` into a nested array of multiple files. See the Predefined Variables notes for the full secure upload pattern (MIME validation, safe filenames).

---

### Postman

- **Postman** is a GUI application for manually building and sending HTTP requests — without needing a browser or writing any frontend code.
- Extremely useful for testing your PHP backend/API **before** any frontend exists.

#### Building a Request in Postman

```
1. Choose the METHOD     → GET, POST, PUT, PATCH, DELETE, etc.
2. Enter the URL          → https://example.com/api/users
3. Set HEADERS (if needed)→ Content-Type, Authorization, etc.
4. Set the BODY (if POST/PUT) → raw JSON, form-data, x-www-form-urlencoded
5. Click "Send"
6. View the RESPONSE      → status code, headers, body
```

#### Example: Testing a JSON API Endpoint

```
Method: POST
URL:    https://example.com/api/users
Headers:
  Content-Type: application/json
  Authorization: Bearer eyJhbGci...
Body (raw → JSON):
  {
    "name": "Phyo",
    "email": "phyo@example.com"
  }
```

```php
<?php
// api/users.php — the PHP endpoint that handles this Postman request
header("Content-Type: application/json");

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $data = json_decode(file_get_contents("php://input"), true);

    http_response_code(201);
    echo json_encode([
        "status" => "created",
        "user"   => $data,
    ]);
}
?>
```

#### Why Postman Is Essential for Backend Developers

| Use Case | Why Postman Helps |
|---|---|
| Testing before frontend exists | Build and verify your API independently |
| Testing non-GET methods | HTML forms can't send PUT/DELETE — Postman can |
| Setting custom headers | Easily test `Authorization`, custom headers |
| Saving requests | Build a "collection" of all your API endpoints |
| Sharing with team | Export/share collections so others can test the same API |
| Automated testing | Write test scripts that run after each request |
| Environment variables | Switch between dev/staging/production URLs easily |

> 💡 **Tip:** Postman (and alternatives like Insomnia or the `curl` command) is how backend developers test their PHP endpoints in isolation — you don't need to build any HTML or JavaScript just to verify your PHP API logic works correctly.

---

### Request Header

- **Headers** carry metadata about the request — already covered in depth in the HTTP fundamentals notes. Here's how to **read** them in PHP specifically.

```php
<?php
// Method 1 — getallheaders() (most convenient, Apache/FPM/built-in server)
$headers = getallheaders();
print_r($headers);
// Array (
//   [Host]          => example.com
//   [Content-Type]  => application/json
//   [Authorization] => Bearer eyJhbGci...
//   [User-Agent]    => PostmanRuntime/7.32.3
// )

echo $headers["Authorization"] ?? "No auth header";

// Method 2 — via $_SERVER (always available, every header prefixed with HTTP_)
$contentType = $_SERVER["CONTENT_TYPE"]          ?? "";
$authHeader  = $_SERVER["HTTP_AUTHORIZATION"]    ?? "";
$userAgent   = $_SERVER["HTTP_USER_AGENT"]       ?? "";
$accept      = $_SERVER["HTTP_ACCEPT"]           ?? "";
$customHdr   = $_SERVER["HTTP_X_CUSTOM_HEADER"]  ?? ""; // X-Custom-Header → HTTP_X_CUSTOM_HEADER

echo $authHeader;
?>
```

#### Reading a Bearer Token from the Authorization Header

```php
<?php
function getBearerToken(): ?string {
    $authHeader = $_SERVER["HTTP_AUTHORIZATION"] ?? "";

    // Some servers strip the header — check getallheaders() as fallback
    if (empty($authHeader) && function_exists("getallheaders")) {
        $headers    = getallheaders();
        $authHeader = $headers["Authorization"] ?? "";
    }

    if (str_starts_with($authHeader, "Bearer ")) {
        return substr($authHeader, 7);  // Everything after "Bearer "
    }

    return null;
}

$token = getBearerToken();
if ($token === null) {
    http_response_code(401);
    echo json_encode(["error" => "Missing Authorization header"]);
    exit();
}

echo "Token received: $token";
?>
```

#### Sending Custom Headers from Postman

```
Headers tab in Postman:
  Key                   Value
  ─────────────────────────────────
  Content-Type           application/json
  Authorization          Bearer abc123token
  X-Request-ID            req-001
  X-Custom-Header         my-custom-value
```

```php
<?php
// Reading the custom header sent above
$requestId = $_SERVER["HTTP_X_REQUEST_ID"]     ?? "unknown";
$customVal = $_SERVER["HTTP_X_CUSTOM_HEADER"]  ?? "unknown";

echo "Request ID: $requestId\n";
echo "Custom value: $customVal\n";
?>
```

> 💡 **Naming pattern:** Any header `X-My-Header` becomes `$_SERVER["HTTP_X_MY_HEADER"]` — dashes become underscores, everything uppercase, prefixed with `HTTP_`. The exceptions are `Content-Type` and `Content-Length`, which become `$_SERVER["CONTENT_TYPE"]` and `$_SERVER["CONTENT_LENGTH"]` (no `HTTP_` prefix).

---

## Response

A **response** is what your PHP script sends back to the client. PHP gives you full control over the status code, headers, and body content.

---

### text/html

- The **default** content type — if you don't set a `Content-Type` header, PHP sends `text/html` automatically.
- Used for regular web pages — the browser parses and renders the output as HTML.

```php
<?php
// Explicit (though it's the default anyway)
header("Content-Type: text/html; charset=UTF-8");

echo "<!DOCTYPE html>";
echo "<html><head><title>My Page</title></head>";
echo "<body><h1>Hello, World!</h1></body></html>";
?>
```

```php
<?php
// Mixing PHP with HTML directly — very common for traditional pages
$username = "Phyo";
?>
<!DOCTYPE html>
<html>
<head><title>Welcome</title></head>
<body>
    <h1>Welcome, <?= htmlspecialchars($username) ?>!</h1>
    <p>Today is <?= date("F j, Y") ?></p>
</body>
</html>
```

> 💡 **Tip:** Always include `charset=UTF-8` explicitly when sending HTML — it ensures special characters (like Burmese text) display correctly, regardless of server defaults.

---

### application/json

- The standard content type for **API responses** — both REST APIs and most modern frontend frameworks (React, Vue) consume JSON.
- Always set this header explicitly — without it, some clients may misinterpret the response.

```php
<?php
header("Content-Type: application/json; charset=UTF-8");

$data = [
    "status" => "success",
    "user"   => [
        "id"    => 1,
        "name"  => "Phyo",
        "email" => "phyo@example.com",
    ],
];

echo json_encode($data, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
// Output: {"status":"success","user":{"id":1,"name":"Phyo","email":"phyo@example.com"}}
?>
```

```php
<?php
// A reusable JSON response helper — common pattern in PHP APIs
function jsonResponse(int $statusCode, array $data): never {
    http_response_code($statusCode);
    header("Content-Type: application/json; charset=UTF-8");
    echo json_encode($data, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
    exit();
}

// Usage throughout your API
jsonResponse(200, ["status" => "success", "data" => $users]);
jsonResponse(404, ["status" => "error", "message" => "User not found"]);
jsonResponse(422, ["status" => "error", "errors" => $validationErrors]);
?>
```

---

### image/jpeg

- Used when PHP needs to **serve an image directly** — either a static file or a dynamically generated image (charts, thumbnails, captchas).
- The `Content-Type` tells the browser to render the response as an image, not text.

```php
<?php
// Serving an existing image file through PHP (e.g., for access-controlled images)
$imagePath = "/var/www/private-uploads/photo.jpg";

if (file_exists($imagePath)) {
    header("Content-Type: image/jpeg");
    header("Content-Length: " . filesize($imagePath));
    readfile($imagePath);
    exit();
} else {
    http_response_code(404);
    echo "Image not found";
}
?>
```

```php
<?php
// Generating an image dynamically (e.g., a simple CAPTCHA or chart)
header("Content-Type: image/jpeg");

$image = imagecreate(200, 100);
$bgColor   = imagecolorallocate($image, 255, 255, 255); // white background
$textColor = imagecolorallocate($image, 0, 0, 0);        // black text

imagestring($image, 5, 50, 40, "Hello PHP", $textColor);

imagejpeg($image);  // Outputs the JPEG directly to the response body
imagedestroy($image);
?>
```

### Other Common Content-Types for Files

| Content-Type | Use For |
|---|---|
| `image/jpeg` | JPEG images |
| `image/png` | PNG images |
| `image/svg+xml` | SVG vector images |
| `application/pdf` | PDF documents |
| `text/css` | CSS files |
| `application/javascript` | JavaScript files |
| `text/plain` | Plain text |
| `application/octet-stream` | Generic binary / force download |

```php
<?php
// Forcing a file download instead of displaying it in the browser
header("Content-Type: application/octet-stream");
header("Content-Disposition: attachment; filename=\"report.pdf\"");
header("Content-Length: " . filesize("/var/www/files/report.pdf"));
readfile("/var/www/files/report.pdf");
exit();
?>
```

---

### header("HTTP/1.1 statuscode some text")

- PHP lets you set the HTTP status line **manually** using `header()` with the special HTTP status string format.
- This is the **older/manual** way — `http_response_code()` (covered in earlier notes) is the modern, simpler equivalent for most cases.

```php
<?php
// Manual status line format
header("HTTP/1.1 200 OK");
header("HTTP/1.1 201 Created");
header("HTTP/1.1 400 Bad Request");
header("HTTP/1.1 401 Unauthorized");
header("HTTP/1.1 403 Forbidden");
header("HTTP/1.1 404 Not Found");
header("HTTP/1.1 500 Internal Server Error");

// Equivalent modern approach — simpler, recommended
http_response_code(200);
http_response_code(201);
http_response_code(404);
?>
```

```php
<?php
// Practical example: a complete 404 handler
function send404(): never {
    header("HTTP/1.1 404 Not Found");
    header("Content-Type: text/html; charset=UTF-8");
    echo "<h1>404 — Page Not Found</h1>";
    echo "<p>The page you're looking for doesn't exist.</p>";
    exit();
}

// Practical example: API error with custom message
function apiError(int $code, string $reasonPhrase, string $message): never {
    header("HTTP/1.1 $code $reasonPhrase");
    header("Content-Type: application/json");
    echo json_encode(["error" => $message]);
    exit();
}

apiError(422, "Unprocessable Entity", "Email field is required");
?>
```

> ⚠️ **Important:** `header()` calls **must happen before any output** — even a single space or blank line before `<?php` in your file will cause a "headers already sent" error. Always call `header()` functions at the very top of your script logic, before any `echo`, `print`, or HTML output.

```php
<?php
// ❌ This causes "headers already sent" error
echo "Some output";
header("HTTP/1.1 404 Not Found");  // Too late! Already sent output.

// ✅ Correct order — headers FIRST, then output
header("HTTP/1.1 404 Not Found");
echo "Some output";
?>
```

### `header()` Status Line vs `http_response_code()`

| Method | Syntax | Notes |
|---|---|---|
| `header("HTTP/1.1 ...")` | Full manual control over version + code + reason phrase | More verbose, useful for non-standard reason phrases |
| `http_response_code(404)` | Just pass the number | Simpler, PHP fills in the standard reason phrase automatically |

```php
<?php
// Both achieve the same practical result for standard codes
header("HTTP/1.1 404 Not Found");  // Manual
http_response_code(404);            // Modern — PHP knows "Not Found" automatically

// http_response_code() is preferred in modern PHP code for its simplicity
?>
```

---

### Location: somewhere

- The `Location` header tells the **client to go to a different URL** — used for redirects.
- Almost always paired with a `3xx` status code (though PHP defaults to `302 Found` automatically if you don't set one).

```php
<?php
// Simple redirect — PHP automatically sends 302 Found
header("Location: /dashboard");
exit();  // ALWAYS exit() after a redirect — code below would still execute otherwise!

// Redirect to an external site
header("Location: https://www.google.com");
exit();

// Redirect with explicit status code — 301 Permanent Redirect
header("HTTP/1.1 301 Moved Permanently");
header("Location: /new-page-url");
exit();

// Redirect with explicit status code — 303 See Other (after POST, PRG pattern)
header("HTTP/1.1 303 See Other");
header("Location: /success");
exit();
?>
```

```php
<?php
// Practical example: redirect after login
session_start();

if (loginSuccessful($_POST["username"], $_POST["password"])) {
    $_SESSION["user_id"] = $userId;
    header("Location: /dashboard");
    exit();
} else {
    header("Location: /login?error=invalid_credentials");
    exit();
}
?>
```

```php
<?php
// Practical example: redirect with a return-to URL (preserve where the user wanted to go)
session_start();

if (!isset($_SESSION["user_id"])) {
    $currentUrl = $_SERVER["REQUEST_URI"];
    header("Location: /login?redirect=" . urlencode($currentUrl));
    exit();
}

// After successful login:
$redirectTo = $_GET["redirect"] ?? "/dashboard";
header("Location: " . $redirectTo);
exit();
?>
```

> ⚠️ **Critical Reminder:** Always call `exit()` (or `die()`) immediately after a `Location` redirect. Without it, PHP **continues executing the rest of the script** — the browser will eventually navigate away, but any code after the `header()` call still runs on the server, which can cause unexpected behavior (like double-processing data).

```php
<?php
// ❌ DANGEROUS — no exit() after redirect
header("Location: /dashboard");
deleteOldSession();   // ⚠️ This still runs! Even though the browser is leaving.
logActivity("redirected"); // ⚠️ This too!

// ✅ SAFE — exit() stops execution immediately
header("Location: /dashboard");
exit();
deleteOldSession();   // Never reached — correct!
?>
```

---

## Putting It All Together

A complete mini example showing request handling and response building together — a simple API endpoint.

```php
<?php
// api/login.php — handles both GET info and POST login

header("Content-Type: application/json; charset=UTF-8");

$method = $_SERVER["REQUEST_METHOD"];

if ($method === "OPTIONS") {
    // CORS preflight
    header("Access-Control-Allow-Methods: POST, OPTIONS");
    header("Access-Control-Allow-Headers: Content-Type");
    http_response_code(204);
    exit();
}

if ($method !== "POST") {
    http_response_code(405);  // Method Not Allowed
    echo json_encode(["error" => "Only POST is allowed on this endpoint"]);
    exit();
}

// Read JSON body (since this is an API, not an HTML form)
$body = json_decode(file_get_contents("php://input"), true);

$username = $body["username"] ?? "";
$password = $body["password"] ?? "";

if (empty($username) || empty($password)) {
    http_response_code(422);
    echo json_encode(["error" => "Username and password are required"]);
    exit();
}

// Simulate checking credentials
$validUser = ($username === "phyo" && $password === "secret123");

if (!$validUser) {
    http_response_code(401);
    echo json_encode(["error" => "Invalid username or password"]);
    exit();
}

// Success — issue a token-like response (no redirect, since this is an API)
http_response_code(200);
echo json_encode([
    "status" => "success",
    "token"  => bin2hex(random_bytes(16)),
    "user"   => ["username" => $username],
]);
?>
```

```php
<?php
// page-login.php — traditional HTML form version (uses Location redirect instead)
session_start();

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $username = $_POST["username"] ?? "";
    $password = $_POST["password"] ?? "";

    if ($username === "phyo" && $password === "secret123") {
        $_SESSION["user_id"] = 1;
        header("Location: /dashboard.php");  // Redirect — text/html flow
        exit();
    }

    header("Location: /login.php?error=1");
    exit();
}
?>
<!DOCTYPE html>
<html>
<body>
    <form method="POST">
        <input type="text"     name="username" placeholder="Username">
        <input type="password" name="password" placeholder="Password">
        <button type="submit">Login</button>
    </form>
</body>
</html>
```

> 💡 **The key difference shown above:** an **API endpoint** (consumed by JS/mobile/Postman) returns JSON with a status code — never a redirect. A **traditional page** (consumed by a browser navigating between pages) uses `Location` redirects and returns HTML.

---

## Quick Revision

- **HTML forms** send data via `method="GET"` (→ `$_GET`) or `method="POST"` (→ `$_POST`); HTML cannot natively send PUT/DELETE.
- **GET requests** put data in the URL — visible, bookmarkable, cacheable. Use for reads/searches, never sensitive data.
- **GET params** are `key=value` pairs after `?`, joined by `&`. Build them safely with `http_build_query()`.
- **POST requests** put data in the body — use `$_POST` for form-encoded data, but for **JSON** bodies you must read `file_get_contents("php://input")` and `json_decode()` it — `$_POST` will be empty for JSON.
- **File uploads** need `enctype="multipart/form-data"` on the form and are read via `$_FILES` — always validate MIME type and generate safe filenames.
- **Postman** lets you build and send any HTTP request (including PUT/DELETE/custom headers) without writing frontend code — essential for testing APIs in isolation.
- **Request headers** are read via `getallheaders()` or `$_SERVER["HTTP_*"]` (dashes → underscores, uppercase, `HTTP_` prefix — except `Content-Type`/`Content-Length`).
- **`text/html`** is the default response type for regular web pages.
- **`application/json`** is the standard for API responses — always set the header explicitly and use `json_encode()`.
- **`image/jpeg`** (and other image types) are used to serve or dynamically generate images — set `Content-Type` before any image output.
- **`header("HTTP/1.1 code text")`** manually sets the status line; **`http_response_code(code)`** is the simpler modern equivalent.
- **`Location:` header** triggers a redirect — **always call `exit()` immediately after**, or the rest of your script keeps executing on the server even though the browser is navigating away.
- **Golden rule:** `header()` calls must happen **before any output** (`echo`, HTML, even whitespace) — or you'll get a "headers already sent" error.
- **APIs return JSON + status codes.** **Traditional pages return HTML + redirects.** Know which one you're building.