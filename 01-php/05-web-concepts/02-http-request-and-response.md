# Modern Web Communication — Part 1: HTTP, Requests & Responses

Understanding how clients and servers talk to each other is the **most fundamental skill** for any full stack developer. Every button click, form submission, API call, and page load is built on this foundation. This series breaks it down from the ground up.

---

## Table of Contents

1. [The Big Picture — How the Web Communicates](#the-big-picture--how-the-web-communicates)
2. [What is HTTP?](#what-is-http)
3. [HTTP Versions — 1.0, 1.1, 2, 3](#http-versions--10-11-2-3)
4. [The Request-Response Cycle](#the-request-response-cycle)
5. [Anatomy of an HTTP Request](#anatomy-of-an-http-request)
   - [Request Line](#request-line)
   - [Request Headers](#request-headers)
   - [Request Body](#request-body)
6. [HTTP Methods (Verbs)](#http-methods-verbs)
7. [Anatomy of an HTTP Response](#anatomy-of-an-http-response)
   - [Status Line](#status-line)
   - [Response Headers](#response-headers)
   - [Response Body](#response-body)
8. [HTTP Status Codes](#http-status-codes)
9. [Quick Revision](#quick-revision)

---

## The Big Picture — How the Web Communicates

Every interaction on the web follows the same fundamental pattern — a **client** asks for something and a **server** responds.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        THE WEB COMMUNICATION CYCLE                  │
│                                                                     │
│   ┌──────────┐    1. HTTP Request     ┌──────────────────────────┐  │
│   │          │ ─────────────────────► │                          │  │
│   │  CLIENT  │                        │        SERVER            │  │
│   │ (Browser)│ ◄───────────────────── │  (Web + App + Database)  │  │
│   │          │    2. HTTP Response    │                          │  │
│   └──────────┘                        └──────────────────────────┘  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

This cycle happens **dozens of times** for a single web page load — once for the HTML, then again for each CSS file, JavaScript file, image, font, and API call.

> 💡 **Open your browser's DevTools (F12) → Network tab** and reload any website. You'll see every single request and response in real time — this is the full stack developer's window into the communication layer.

---

## What is HTTP?

- **HTTP** stands for **HyperText Transfer Protocol**.
- It is the **language** that clients (browsers, mobile apps, other services) and servers use to communicate.
- HTTP is a **stateless** protocol — each request is completely independent. The server does not remember the previous request by default.
- HTTP defines:
  - **How requests are formatted** (what the client sends)
  - **How responses are formatted** (what the server sends back)
  - **What methods are available** (GET, POST, PUT, DELETE, etc.)
  - **What status codes mean** (200, 404, 500, etc.)

```
HTTP is the agreed-upon "grammar" of the web.
Without it, a browser wouldn't know how to ask for a page,
and a server wouldn't know how to respond.
```

---

## HTTP Versions — 1.0, 1.1, 2, 3

Each version improved speed, security, or efficiency. As a full stack developer, you should know why they exist.

### HTTP/1.0 (1996)

- Each request opens a **new TCP connection** — and closes it after the response.
- Loading a page with 10 images = 11 separate connections = very slow.

```
Request 1: Open connection → GET HTML → Close connection
Request 2: Open connection → GET style.css → Close connection
Request 3: Open connection → GET image.jpg → Close connection
... (11 times total — very inefficient)
```

---

### HTTP/1.1 (1997 — Still widely used)

- Introduced **persistent connections** (keep-alive) — one TCP connection can handle multiple requests.
- Introduced **chunked transfer encoding** — servers can start sending data before knowing the total size.
- Still limited: requests must be handled **in order** (head-of-line blocking).

```
One connection:
→ GET HTML    ← Response
→ GET CSS     ← Response
→ GET JS      ← Response
→ GET image   ← Response
(Sequential, but on the same connection — much better)
```

---

### HTTP/2 (2015 — Modern standard)

- **Multiplexing** — multiple requests and responses can travel simultaneously over a **single connection**, in parallel. No more waiting.
- **Header compression** — HTTP/2 compresses repetitive headers, reducing overhead.
- **Server Push** — server can proactively send resources the client will need, before the client asks.
- **Binary protocol** — more efficient than HTTP/1.1's text-based format.

```
One connection, all at once:
→ GET HTML ─────────────────────► ← Response
→ GET CSS  ──────────────────►    ← Response
→ GET JS   ────────────────►      ← Response
→ GET img  ──────────────►        ← Response
(Parallel — much faster page load)
```

---

### HTTP/3 (2022 — Newest)

- Built on **QUIC** instead of TCP — a UDP-based protocol designed to reduce latency.
- Solves the "TCP head-of-line blocking" problem at the transport layer.
- **Always encrypted** (TLS 1.3 built-in) — no separate step.
- Faster connection setup — 0-RTT (zero round-trip time) for returning visitors.

### HTTP Versions at a Glance

| Version | Year | Key Feature | Connection |
|---|---|---|---|
| HTTP/1.0 | 1996 | Basic protocol | New TCP per request |
| HTTP/1.1 | 1997 | Persistent connections | One TCP, sequential |
| HTTP/2 | 2015 | Multiplexing, header compression | One TCP, parallel |
| HTTP/3 | 2022 | QUIC (UDP-based), 0-RTT | QUIC, parallel + faster |

> 💡 **Tip:** As a full stack developer, you don't usually configure HTTP versions manually — your web server (Nginx, Apache) and hosting provider handle it. But knowing the differences explains why modern sites feel faster and helps you debug network performance issues.

---

## The Request-Response Cycle

Every single web interaction follows this cycle. Let's trace it step by step with a real example.

**Scenario:** You type `https://shop.example.com/products?page=2` and press Enter.

```
STEP 1 — DNS Resolution
  Your browser asks DNS: "What's the IP for shop.example.com?"
  DNS responds: "93.184.216.34"

STEP 2 — TCP Connection
  Browser establishes a TCP connection to 93.184.216.34 on port 443 (HTTPS)

STEP 3 — TLS Handshake
  Browser and server negotiate encryption (TLS)
  Server presents its SSL certificate
  Secure encrypted channel is established

STEP 4 — HTTP Request Sent
  Browser sends the HTTP request:
    GET /products?page=2 HTTP/1.1
    Host: shop.example.com
    Accept: text/html
    Cookie: session_id=abc123

STEP 5 — Server Processes Request
  Nginx (web server) receives it
  Hands off to PHP-FPM (application server)
  PHP runs the products controller
  PHP queries MySQL database for page 2 of products
  PHP generates HTML

STEP 6 — HTTP Response Sent
  HTTP/1.1 200 OK
  Content-Type: text/html; charset=UTF-8
  Content-Length: 8492
  [HTML body follows...]

STEP 7 — Browser Renders
  Browser receives the HTML
  Parses it — discovers CSS/JS/image URLs
  Fires more HTTP requests for each asset (repeating from Step 4)
  Renders the final page
```

> 💡 **All of this happens in milliseconds.** A well-optimized website completes the full cycle — including DNS, TCP, TLS, the request, server processing, and response — in under 200ms. This is what "performance optimization" is fundamentally about.

---

## Anatomy of an HTTP Request

An HTTP request has three parts: the **request line**, **headers**, and optionally a **body**.

```
┌─────────────────────────────────────────┐
│              HTTP REQUEST               │
├─────────────────────────────────────────┤
│  REQUEST LINE                           │
│  GET /products?page=2 HTTP/1.1          │
├─────────────────────────────────────────┤
│  HEADERS                                │
│  Host: shop.example.com                 │
│  Accept: text/html,application/json     │
│  Accept-Language: en-US,en;q=0.9        │
│  Accept-Encoding: gzip, deflate, br     │
│  Authorization: Bearer eyJhbGci...      │
│  Cookie: session_id=abc123              │
│  User-Agent: Mozilla/5.0 ...            │
├─────────────────────────────────────────┤
│  BODY (empty for GET, present for POST) │
│  (POST body would appear here)          │
└─────────────────────────────────────────┘
```

---

### Request Line

The very first line of every HTTP request. Contains three pieces:

```
METHOD   PATH+QUERY        HTTP_VERSION
GET    /products?page=2    HTTP/1.1
POST   /api/users          HTTP/2
DELETE /api/posts/42       HTTP/1.1
```

- **Method** — what action to perform (GET, POST, PUT, DELETE, etc.)
- **Path** — the resource being requested (`/products`, `/api/users/42`)
- **Query String** — optional key=value pairs after `?` (`?page=2&sort=price`)
- **HTTP Version** — which version of the protocol is being used

---

### Request Headers

Headers are **key: value pairs** that carry metadata about the request. They tell the server about the client, what formats it accepts, authentication credentials, and more.

```http
Host: shop.example.com
```
- **Required in HTTP/1.1+** — tells the server which domain is being requested (one server can host many domains).

```http
Accept: text/html, application/json, */*
```
- What content types the client can handle — server uses this for **content negotiation**.

```http
Accept-Language: en-US,en;q=0.9,my;q=0.8
```
- Preferred languages for the response content (`q` = quality/priority score).

```http
Accept-Encoding: gzip, deflate, br
```
- Compression formats the client supports — server can compress the response to save bandwidth.

```http
Content-Type: application/json
```
- The **format of the request body** (only relevant for POST/PUT/PATCH requests).

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```
- Authentication credentials — often a JWT token or Basic auth.

```http
Cookie: session_id=abc123; theme=dark; cart_id=xyz789
```
- All cookies for this domain, sent automatically by the browser with every request.

```http
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
```
- Identifies the client software (browser, version, OS).

```http
Referer: https://shop.example.com/category/shoes
```
- The URL of the page that linked to the current request (note: the header name has a historical typo — it's "Referer" not "Referrer").

```http
Cache-Control: no-cache
```
- Instructions for caching behavior — `no-cache` means "revalidate before using cached version".

```http
Origin: https://app.example.com
```
- The origin of the request — critical for **CORS** (Cross-Origin Resource Sharing) checks.

---

### Request Body

- Only present in **POST, PUT, PATCH** requests (and sometimes DELETE).
- GET requests **never** have a body.
- The `Content-Type` header tells the server how to parse the body.

```http
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 67

{
  "name": "Phyo",
  "email": "phyo@example.com",
  "password": "secret123"
}
```

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 32

username=phyo&password=secret123
```

```http
POST /upload HTTP/1.1
Host: example.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxk

------WebKitFormBoundary7MA4YWxk
Content-Disposition: form-data; name="file"; filename="photo.jpg"
Content-Type: image/jpeg

[binary file data here]
------WebKitFormBoundary7MA4YWxk--
```

### Common Request Body Content-Types

| Content-Type | Use Case | Format |
|---|---|---|
| `application/json` | API requests | `{"key": "value"}` |
| `application/x-www-form-urlencoded` | HTML form POST | `name=Phyo&age=25` |
| `multipart/form-data` | File uploads | Mixed text + binary |
| `text/plain` | Raw text | Plain text |
| `application/xml` | Legacy APIs, SOAP | `<root><key>value</key></root>` |

---

## HTTP Methods (Verbs)

HTTP methods define **what action** is being requested. They are fundamental to RESTful API design.

```php
<?php
// In PHP — reading the method from the request
$method = $_SERVER["REQUEST_METHOD"];
// "GET", "POST", "PUT", "DELETE", "PATCH", "HEAD", "OPTIONS"
?>
```

### GET

- **Retrieve** data from the server.
- **No body** — data goes in the URL query string.
- **Safe** — should never modify server data.
- **Idempotent** — calling it 10 times has the same result as calling it once.
- **Cacheable** — browsers and proxies can cache GET responses.

```http
GET /products?category=shoes&page=2 HTTP/1.1
Host: shop.example.com
```

---

### POST

- **Send data** to the server — typically to **create** a new resource.
- Has a **request body** with the data.
- **Not idempotent** — sending the same POST twice might create two resources.
- **Not cacheable** by default.

```http
POST /api/products HTTP/1.1
Content-Type: application/json

{"name": "Nike Shoes", "price": 99.99}
```

---

### PUT

- **Replace** an existing resource entirely — send the complete updated version.
- **Idempotent** — same PUT request repeated = same result.

```http
PUT /api/products/42 HTTP/1.1
Content-Type: application/json

{"name": "Nike Shoes v2", "price": 109.99, "stock": 50}
```

---

### PATCH

- **Partially update** an existing resource — only send what changed.
- More efficient than PUT when only updating one or two fields.

```http
PATCH /api/products/42 HTTP/1.1
Content-Type: application/json

{"price": 89.99}
```

---

### DELETE

- **Remove** a resource from the server.
- **Idempotent** — deleting the same resource twice has the same final result (it's gone).

```http
DELETE /api/products/42 HTTP/1.1
Authorization: Bearer eyJhbGci...
```

---

### HEAD

- Exactly like GET — but the server **returns only headers, no body**.
- Used to check if a resource exists, get its size, or check when it was last modified — without downloading it.

```http
HEAD /large-file.zip HTTP/1.1
Host: downloads.example.com
```

---

### OPTIONS

- Asks the server **what methods are allowed** for a resource.
- Critical for **CORS preflight requests** — browsers automatically send OPTIONS before cross-origin POST/PUT/DELETE.

```http
OPTIONS /api/users HTTP/1.1
Origin: https://app.example.com
Access-Control-Request-Method: POST
```

```http
Response:
Allow: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
```

### HTTP Methods Summary

| Method | Action | Has Body | Idempotent | Safe | Cacheable |
|---|---|---|---|---|---|
| GET | Read/retrieve | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes |
| POST | Create | ✅ Yes | ❌ No | ❌ No | ❌ No |
| PUT | Replace entirely | ✅ Yes | ✅ Yes | ❌ No | ❌ No |
| PATCH | Partial update | ✅ Yes | ❌ No | ❌ No | ❌ No |
| DELETE | Remove | Optional | ✅ Yes | ❌ No | ❌ No |
| HEAD | Headers only | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes |
| OPTIONS | Capabilities | ❌ No | ✅ Yes | ✅ Yes | ❌ No |

> 💡 **Idempotent** means: calling the same operation multiple times produces the same result as calling it once. GET, PUT, DELETE, HEAD, and OPTIONS are idempotent. POST is not — that's why browser shows "resubmit this form?" when you refresh after a POST.

---

## Anatomy of an HTTP Response

An HTTP response also has three parts: the **status line**, **headers**, and **body**.

```
┌─────────────────────────────────────────┐
│             HTTP RESPONSE               │
├─────────────────────────────────────────┤
│  STATUS LINE                            │
│  HTTP/1.1 200 OK                        │
├─────────────────────────────────────────┤
│  HEADERS                                │
│  Content-Type: text/html; charset=UTF-8 │
│  Content-Length: 8492                   │
│  Cache-Control: max-age=3600            │
│  Set-Cookie: session_id=abc123; ...     │
│  X-Powered-By: PHP/8.3                  │
│  Date: Sat, 28 Jun 2026 10:30:00 GMT    │
├─────────────────────────────────────────┤
│  BODY                                   │
│  <!DOCTYPE html>                        │
│  <html>...</html>                       │
│  (or JSON, image bytes, file data, etc) │
└─────────────────────────────────────────┘
```

---

### Status Line

```
HTTP/1.1   200   OK
version   code  reason phrase
```

- **HTTP Version** — the protocol version used for this response.
- **Status Code** — a 3-digit number indicating the result.
- **Reason Phrase** — a human-readable description of the status code.

---

### Response Headers

```http
Content-Type: application/json; charset=UTF-8
```
- What format the response body is in — the client uses this to decide how to process it.

```http
Content-Length: 1234
```
- Size of the response body in bytes — lets the client know when it has received everything.

```http
Cache-Control: public, max-age=3600
```
- Tells the client (and intermediate caches) how long to cache this response (`3600` seconds = 1 hour).

```http
Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Strict; Path=/
```
- Instructs the browser to store a cookie — sent back with every future request to this domain.

```http
Location: https://shop.example.com/new-page
```
- Used with **redirect status codes** (301, 302) to tell the client where to go next.

```http
Access-Control-Allow-Origin: https://app.example.com
```
- CORS header — tells the browser which origins are allowed to read this response.

```http
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d"
```
- A unique fingerprint of the response content — used for cache validation.

```http
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
```
- Security headers — protect against common attacks like clickjacking, MIME sniffing, and XSS.

```php
<?php
// Setting response headers in PHP
header("Content-Type: application/json; charset=UTF-8");
header("Cache-Control: no-cache, no-store, must-revalidate");
header("Access-Control-Allow-Origin: https://app.example.com");

// Setting security headers
header("X-Content-Type-Options: nosniff");
header("X-Frame-Options: DENY");
header("Strict-Transport-Security: max-age=31536000; includeSubDomains");

// Setting a cookie via header
header("Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Strict; Path=/");

// Sending a redirect
header("Location: /dashboard");
exit();  // Always exit after a redirect!
?>
```

---

### Response Body

- The actual **content** being sent back — can be HTML, JSON, XML, an image, a file, etc.
- The `Content-Type` header tells the client how to interpret it.

```json
// JSON API response body
{
  "status": "success",
  "data": {
    "users": [
      { "id": 1, "name": "Phyo", "email": "phyo@example.com" },
      { "id": 2, "name": "Alice", "email": "alice@example.com" }
    ]
  },
  "meta": {
    "total": 2,
    "page": 1,
    "per_page": 10
  }
}
```

```html
<!-- HTML response body -->
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Products — Shop</title>
</head>
<body>
  <h1>Products</h1>
  <div class="product-grid">...</div>
</body>
</html>
```

---

## HTTP Status Codes

Status codes are grouped into **five classes** by their first digit. Every backend developer must know these by heart.

### 1xx — Informational

Rarely seen directly — tells the client the request is being processed.

| Code | Name | Meaning |
|---|---|---|
| `100` | Continue | Server received request headers; client should proceed sending body |
| `101` | Switching Protocols | Server is switching protocols (e.g., HTTP → WebSocket) |

---

### 2xx — Success

The request was received, understood, and fulfilled.

| Code | Name | When to use |
|---|---|---|
| `200` | OK | Standard success — GET, PUT, PATCH returned data |
| `201` | Created | POST succeeded and a new resource was created |
| `202` | Accepted | Request accepted for processing, but not yet complete (async tasks) |
| `204` | No Content | Success, but no body to return — common for DELETE responses |

```php
<?php
// Setting status codes in PHP
http_response_code(200);  // OK
http_response_code(201);  // Created — after successful POST
http_response_code(204);  // No Content — after DELETE

// Or inline with header()
header("HTTP/1.1 201 Created");
?>
```

---

### 3xx — Redirection

The client needs to take further action — usually go to a different URL.

| Code | Name | When to use |
|---|---|---|
| `301` | Moved Permanently | Resource has permanently moved to a new URL — browsers cache this |
| `302` | Found | Temporary redirect — go here for now, but don't cache it |
| `303` | See Other | After POST — redirect to a GET to prevent double-submission |
| `304` | Not Modified | Cache is still valid — use your cached version |
| `307` | Temporary Redirect | Like 302, but method must stay the same (POST stays POST) |
| `308` | Permanent Redirect | Like 301, but method must stay the same |

```php
<?php
// Common redirect patterns in PHP

// Permanent redirect (301) — for moved pages, old URLs
header("HTTP/1.1 301 Moved Permanently");
header("Location: https://example.com/new-page");
exit();

// Temporary redirect (302) — for login redirects etc.
header("Location: /login");
exit();

// Post-Redirect-Get pattern (303) — after form submission
header("HTTP/1.1 303 See Other");
header("Location: /success");
exit();
?>
```

> 💡 **Post-Redirect-Get (PRG)** is a critical pattern: after a successful POST (form submit), always redirect to a GET page. This prevents the "resubmit form?" dialog if the user refreshes, and prevents duplicate form submissions.

---

### 4xx — Client Errors

The request itself is wrong — the client made a mistake.

| Code | Name | When to use |
|---|---|---|
| `400` | Bad Request | Malformed request, invalid data, missing required fields |
| `401` | Unauthorized | Not authenticated — "you need to log in first" |
| `403` | Forbidden | Authenticated but not allowed — "you don't have permission" |
| `404` | Not Found | Resource doesn't exist at this URL |
| `405` | Method Not Allowed | HTTP method not supported (e.g., DELETE on a read-only endpoint) |
| `408` | Request Timeout | Server waited too long for the client |
| `409` | Conflict | Conflict with current state (e.g., duplicate email on registration) |
| `410` | Gone | Resource permanently removed (stronger than 404 — tells crawlers to stop) |
| `422` | Unprocessable Entity | Request is well-formed but fails validation (common in REST APIs) |
| `429` | Too Many Requests | Rate limit exceeded |

```php
<?php
// 401 vs 403 — important distinction
// 401 = "Who are you?" → Not logged in
// 403 = "I know who you are, but NO." → Logged in but no permission

function requireAuth(): void {
    if (!isset($_SESSION["user_id"])) {
        http_response_code(401);
        echo json_encode(["error" => "Authentication required"]);
        exit();
    }
}

function requireAdmin(): void {
    requireAuth();
    if ($_SESSION["role"] !== "admin") {
        http_response_code(403);
        echo json_encode(["error" => "Admin access required"]);
        exit();
    }
}
?>
```

---

### 5xx — Server Errors

The server failed — the request was valid, but something went wrong on the server side.

| Code | Name | When to use |
|---|---|---|
| `500` | Internal Server Error | Generic server error — unexpected exception, crash |
| `501` | Not Implemented | Method not supported by the server at all |
| `502` | Bad Gateway | Reverse proxy got an invalid response from upstream server |
| `503` | Service Unavailable | Server is down for maintenance or overloaded |
| `504` | Gateway Timeout | Reverse proxy timed out waiting for upstream server |

```php
<?php
// Simple error handling in PHP — return appropriate status codes
try {
    $data = fetchFromDatabase($id);

    if ($data === null) {
        http_response_code(404);
        echo json_encode(["error" => "Resource not found"]);
        exit();
    }

    http_response_code(200);
    echo json_encode(["data" => $data]);

} catch (DatabaseException $e) {
    http_response_code(503);
    echo json_encode(["error" => "Database temporarily unavailable"]);
    // Log the actual error internally — never expose it to the client!
    error_log($e->getMessage());
    exit();

} catch (Throwable $e) {
    http_response_code(500);
    echo json_encode(["error" => "Internal server error"]);
    error_log($e->getMessage());
    exit();
}
?>
```

> ⚠️ **Security Warning:** Never expose internal error details (stack traces, database errors, file paths) in your response body — they give attackers valuable information. Log them server-side and return a generic message to the client.

### Status Codes Cheat Sheet

```
2xx = ✅ Success
  200 OK | 201 Created | 204 No Content

3xx = ↩️ Redirect
  301 Permanent | 302 Temporary | 303 See Other | 304 Not Modified

4xx = ❌ Client's Fault
  400 Bad Request | 401 Unauthorized | 403 Forbidden | 404 Not Found
  409 Conflict | 422 Validation Error | 429 Rate Limited

5xx = 💥 Server's Fault
  500 Internal Error | 502 Bad Gateway | 503 Unavailable | 504 Timeout
```

---

## Quick Revision

- **HTTP** is the protocol (language) that defines how clients and servers communicate — stateless, meaning each request is independent.
- **HTTP versions:** 1.0 (new connection per request), 1.1 (persistent connections), 2 (multiplexing — parallel requests), 3 (QUIC-based, always encrypted, faster).
- Every web interaction follows the **request-response cycle:** DNS lookup → TCP connection → TLS handshake → HTTP request → server processing → HTTP response → browser rendering.
- An **HTTP request** has three parts: request line (method + path + version), headers (metadata), body (data — POST/PUT only, never GET).
- **HTTP methods:** GET (read), POST (create), PUT (replace), PATCH (partial update), DELETE (remove), HEAD (headers only), OPTIONS (capabilities/CORS preflight).
- **Idempotent** = calling the same operation multiple times gives the same result. GET, PUT, DELETE, HEAD are idempotent. POST is not.
- An **HTTP response** has: status line (version + code + reason), headers (metadata), body (content).
- Key **request headers:** `Host`, `Accept`, `Content-Type`, `Authorization`, `Cookie`, `User-Agent`, `Origin`.
- Key **response headers:** `Content-Type`, `Set-Cookie`, `Location`, `Cache-Control`, `Access-Control-Allow-Origin`, security headers.
- **Status codes:** 2xx success, 3xx redirect, 4xx client error, 5xx server error. Must-know: 200, 201, 204, 301, 302, 303, 304, 400, 401, 403, 404, 409, 422, 429, 500, 502, 503.
- **401 vs 403:** 401 = "not logged in", 403 = "logged in but not allowed."
- **POST-Redirect-GET (PRG):** after a POST form submission, always redirect to GET to prevent double-submission.
- Never expose internal error details in responses — log them server-side, return a generic message to the client.