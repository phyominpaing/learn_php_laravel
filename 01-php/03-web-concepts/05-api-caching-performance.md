# Modern Web Communication — Part 4: APIs, Caching, Performance & Developer Tools

This final part covers how modern apps are structured around APIs, how to make them fast with caching, how to understand real-world communication patterns, and the tools every full stack developer uses to inspect and debug web communication.

---

## Table of Contents

1. [REST APIs — Design Principles](#rest-apis--design-principles)
2. [API Versioning](#api-versioning)
3. [WebSockets — Real-Time Communication](#websockets--real-time-communication)
4. [Server-Sent Events (SSE)](#server-sent-events-sse)
5. [HTTP vs WebSocket vs SSE](#http-vs-websocket-vs-sse)
6. [CDN — Content Delivery Networks](#cdn--content-delivery-networks)
7. [HTTP Caching Deep Dive](#http-caching-deep-dive)
8. [Performance Essentials](#performance-essentials)
9. [Developer Tools for Full Stack Debugging](#developer-tools-for-full-stack-debugging)
10. [The Complete Full Stack Communication Map](#the-complete-full-stack-communication-map)
11. [Quick Revision](#quick-revision)

---

## REST APIs — Design Principles

**REST** (Representational State Transfer) is an architectural style for designing networked applications. It's the dominant pattern for modern web APIs. A REST API uses standard HTTP methods and treats everything as a **resource**.

### Core REST Principles

```
1. Client-Server Separation   — Frontend and backend are independent
2. Stateless                  — Each request contains all info needed; no server memory
3. Cacheable                  — Responses should declare if they can be cached
4. Uniform Interface          — Consistent, predictable URL structure + HTTP methods
5. Layered System             — Client doesn't know if it's talking to origin or proxy
```

### Resource-Based URLs

URLs should represent **nouns** (resources), not **verbs** (actions). The HTTP method IS the verb.

```
❌ BAD — verbs in URLs
GET  /getUsers
POST /createUser
POST /deleteUser?id=42
POST /updateUser

✅ GOOD — nouns in URLs, verbs are HTTP methods
GET    /users           → list all users
POST   /users           → create a new user
GET    /users/42        → get user with ID 42
PUT    /users/42        → replace user 42 entirely
PATCH  /users/42        → partially update user 42
DELETE /users/42        → delete user 42
```

### Nested Resources

```
GET    /users/42/orders           → all orders by user 42
GET    /users/42/orders/7         → specific order 7 by user 42
POST   /users/42/orders           → create an order for user 42
DELETE /users/42/orders/7         → delete order 7 of user 42
```

### Complete REST API Example in PHP

```php
<?php
// Simple REST API router in PHP
header("Content-Type: application/json");

$method = $_SERVER["REQUEST_METHOD"];
$path   = parse_url($_SERVER["REQUEST_URI"], PHP_URL_PATH);
$parts  = explode("/", trim($path, "/"));
// /api/users/42 → ["api", "users", "42"]

$resource = $parts[1] ?? "";
$id       = isset($parts[2]) ? (int)$parts[2] : null;

match (true) {
    // GET /api/users
    $method === "GET" && $resource === "users" && $id === null => listUsers(),
    // GET /api/users/42
    $method === "GET" && $resource === "users" && $id !== null => getUser($id),
    // POST /api/users
    $method === "POST" && $resource === "users" => createUser(),
    // PUT /api/users/42
    $method === "PUT" && $resource === "users" && $id !== null => updateUser($id),
    // DELETE /api/users/42
    $method === "DELETE" && $resource === "users" && $id !== null => deleteUser($id),
    // OPTIONS (CORS preflight)
    $method === "OPTIONS" => http_response_code(204),
    // 404
    default => notFound(),
};

function listUsers(): void {
    // GET with filtering, sorting, pagination via query params
    $page    = max(1, (int)($_GET["page"]     ?? 1));
    $perPage = min(100, (int)($_GET["per_page"] ?? 20));
    $sort    = in_array($_GET["sort"] ?? "", ["name", "email", "created_at"], true)
               ? $_GET["sort"] : "created_at";

    $offset = ($page - 1) * $perPage;

    $users = fetchFromDb("SELECT * FROM users ORDER BY $sort LIMIT ? OFFSET ?",
                         [$perPage, $offset]);
    $total = fetchScalar("SELECT COUNT(*) FROM users");

    http_response_code(200);
    echo json_encode([
        "data" => $users,
        "meta" => [
            "total"    => $total,
            "page"     => $page,
            "per_page" => $perPage,
            "pages"    => ceil($total / $perPage),
        ],
        "links" => [
            "next" => $page < ceil($total / $perPage) ? "/api/users?page=" . ($page + 1) : null,
            "prev" => $page > 1 ? "/api/users?page=" . ($page - 1) : null,
        ],
    ], JSON_UNESCAPED_UNICODE);
}

function createUser(): void {
    $body = json_decode(file_get_contents("php://input"), true);

    // Validate
    $errors = [];
    if (empty($body["name"]))  $errors["name"]  = "Name is required";
    if (empty($body["email"])) $errors["email"] = "Email is required";
    if (!filter_var($body["email"] ?? "", FILTER_VALIDATE_EMAIL)) {
        $errors["email"] = "Invalid email format";
    }

    if (!empty($errors)) {
        http_response_code(422);  // Unprocessable Entity — validation failed
        echo json_encode(["errors" => $errors]);
        return;
    }

    // Check duplicate
    if (emailExists($body["email"])) {
        http_response_code(409);  // Conflict
        echo json_encode(["error" => "Email already registered"]);
        return;
    }

    $id = insertUser($body["name"], $body["email"]);

    http_response_code(201);  // Created
    header("Location: /api/users/$id");  // Tell client where new resource lives
    echo json_encode(["id" => $id, "name" => $body["name"], "email" => $body["email"]]);
}

function notFound(): void {
    http_response_code(404);
    echo json_encode(["error" => "Resource not found"]);
}
?>
```

### Consistent API Response Structure

Always return a consistent structure — it makes the API predictable and easy to consume.

```json
// Success response
{
  "status": "success",
  "data": { "id": 42, "name": "Phyo", "email": "phyo@example.com" },
  "meta": { "request_id": "abc-123", "timestamp": 1719561600 }
}

// Error response
{
  "status": "error",
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The request data is invalid",
    "details": {
      "email": "Invalid email format",
      "name": "Name is required"
    }
  }
}

// Paginated list response
{
  "status": "success",
  "data": [...],
  "meta": {
    "total": 100,
    "page": 2,
    "per_page": 20,
    "pages": 5
  },
  "links": {
    "self": "/api/users?page=2",
    "next": "/api/users?page=3",
    "prev": "/api/users?page=1"
  }
}
```

---

## API Versioning

APIs must evolve. Versioning allows you to make breaking changes without breaking existing clients.

```
Strategy 1: URL Path versioning (most common, most visible)
GET /api/v1/users
GET /api/v2/users

Strategy 2: Header versioning
GET /api/users
API-Version: 2

Strategy 3: Query parameter versioning
GET /api/users?version=2
```

```php
<?php
// URL versioning in PHP
$path  = parse_url($_SERVER["REQUEST_URI"], PHP_URL_PATH);
$parts = explode("/", trim($path, "/"));
// /api/v2/users → ["api", "v2", "users"]

$version  = $parts[1] ?? "v1";  // Default to v1
$resource = $parts[2] ?? "";

match ($version) {
    "v1" => require __DIR__ . "/../api/v1/router.php",
    "v2" => require __DIR__ . "/../api/v2/router.php",
    default => sendJson(404, ["error" => "API version not found"]),
};
?>
```

> 💡 **Tip:** Never remove or change the behavior of existing API versions without a deprecation period — always maintain old versions for some time after introducing new ones. Announce deprecations in response headers: `Deprecation: version="v1", sunset="2027-01-01"`.

---

## WebSockets — Real-Time Communication

**HTTP** is a **request-response** model — the client must ask, then the server answers. This is inefficient for real-time use cases (chat, live scores, notifications) because you'd have to poll (ask repeatedly).

**WebSockets** provide a **persistent, full-duplex** connection — both client and server can send messages at any time.

```
HTTP (traditional):
Client: "Any new messages?"  →  Server: "No."   (repeated every second = wasteful)
Client: "Any new messages?"  →  Server: "No."
Client: "Any new messages?"  →  Server: "Yes! Here they are!"

WebSocket (real-time):
Client establishes WS connection → stays open
Server: "New message!"   (whenever something happens)
Server: "User joined!"
Client: "Send this chat message"
Server: "Message delivered!"
(One connection, used forever)
```

### WebSocket Handshake

```http
// Client sends HTTP upgrade request:
GET /chat HTTP/1.1
Host: chat.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

// Server responds:
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
// Connection is now a WebSocket — HTTP is done
```

```php
<?php
// PHP WebSocket example using Ratchet (popular PHP WebSocket library)
// composer require cboden/ratchet

use Ratchet\MessageComponentInterface;
use Ratchet\ConnectionInterface;

class ChatServer implements MessageComponentInterface {
    protected \SplObjectStorage $clients;

    public function __construct() {
        $this->clients = new \SplObjectStorage;
    }

    public function onOpen(ConnectionInterface $conn): void {
        $this->clients->attach($conn);
        echo "New connection: {$conn->resourceId}\n";
    }

    public function onMessage(ConnectionInterface $from, $msg): void {
        $data = json_decode($msg, true);

        // Broadcast to all OTHER connected clients
        foreach ($this->clients as $client) {
            if ($client !== $from) {
                $client->send(json_encode([
                    "type"    => "message",
                    "from"    => $from->resourceId,
                    "content" => htmlspecialchars($data["content"] ?? ""),
                    "time"    => date("H:i"),
                ]));
            }
        }
    }

    public function onClose(ConnectionInterface $conn): void {
        $this->clients->detach($conn);
        echo "Connection {$conn->resourceId} disconnected\n";
    }

    public function onError(ConnectionInterface $conn, \Exception $e): void {
        echo "Error: {$e->getMessage()}\n";
        $conn->close();
    }
}
?>
```

```javascript
// JavaScript client-side WebSocket
const ws = new WebSocket("wss://chat.example.com/ws");  // wss = WebSocket Secure

ws.onopen = () => {
    console.log("Connected!");
    ws.send(JSON.stringify({ content: "Hello everyone!" }));
};

ws.onmessage = (event) => {
    const msg = JSON.parse(event.data);
    console.log(`${msg.from}: ${msg.content}`);
};

ws.onclose = () => console.log("Disconnected");
ws.onerror = (error) => console.error("WebSocket error:", error);
```

**Use cases:** Chat apps, live notifications, multiplayer games, collaborative editing, live dashboards, trading platforms.

---

## Server-Sent Events (SSE)

**SSE** is a lighter alternative to WebSockets for **one-way** real-time communication: server → client only. Simpler to implement, works over regular HTTP.

```
HTTP: one request → one response → done
SSE:  one request → response streams forever → server sends events anytime
WebSocket: persistent bi-directional connection
```

```php
<?php
// Server-Sent Events in PHP
header("Content-Type: text/event-stream");
header("Cache-Control: no-cache");
header("X-Accel-Buffering: no");  // Disable Nginx buffering for SSE

// Keep the connection alive and stream events
while (true) {
    // Get latest data (e.g., new notifications)
    $notifications = getNewNotifications();

    foreach ($notifications as $notification) {
        echo "event: notification\n";  // Event type
        echo "data: " . json_encode($notification) . "\n";
        echo "id: " . $notification["id"] . "\n";  // Last event ID (for reconnect)
        echo "\n";  // Blank line = end of event
        ob_flush();
        flush();  // Send immediately
    }

    // Send a heartbeat every 30 seconds (keeps connection alive)
    echo ": heartbeat\n\n";
    ob_flush();
    flush();

    sleep(1);  // Check for new events every second
}
?>
```

```javascript
// JavaScript SSE client
const evtSource = new EventSource("/api/notifications");

evtSource.addEventListener("notification", (event) => {
    const data = JSON.parse(event.data);
    console.log("New notification:", data);
    showNotificationBadge(data);
});

evtSource.onerror = () => {
    // Browser automatically reconnects on error
    console.log("SSE connection lost — browser will retry...");
};
```

---

## HTTP vs WebSocket vs SSE

| Feature | HTTP (REST) | WebSocket | SSE |
|---|---|---|---|
| Direction | Request ↔ Response | Full-duplex (both ways) | Server → Client only |
| Connection | New per request | Persistent | Persistent |
| Protocol | HTTP/1.1, HTTP/2 | WebSocket (after upgrade) | HTTP |
| Use case | CRUD APIs, page loads | Chat, games, collaboration | Notifications, live feeds |
| Complexity | Simple | Moderate | Simple |
| Browser support | Universal | Very good | Very good |
| PHP ease | Native | Needs library (Ratchet) | Native |
| Best for | Most web communication | True real-time, bi-directional | One-way real-time updates |

---

## CDN — Content Delivery Networks

A **CDN** is a globally distributed network of servers that cache and serve content from the location **closest to the user** — dramatically reducing latency.

```
Without CDN:
User in Tokyo ──────────────────────────────► Server in London (200ms)

With CDN:
User in Tokyo ──────► CDN Edge in Tokyo (5ms) → Cache hit! Instant!
                        (or if miss) ──────────► Origin in London (fetch once, cache)
```

### What a CDN Caches

- Static files: images, CSS, JavaScript, fonts, videos — cached at edge nodes globally.
- API responses: cacheable responses (GET with appropriate Cache-Control headers).
- Entire HTML pages for static sites.

```php
<?php
// Setting cache headers for CDN caching

// Cache this response in CDN for 1 hour, in browser for 10 minutes
header("Cache-Control: public, s-maxage=3600, max-age=600");
//                      ↑       ↑               ↑
//                   shareable  CDN cache       browser cache

// Cache busting: when content changes, change the URL
// style.css?v=abc123  or  style.abc123.css
// CDN caches the new URL; old cache becomes irrelevant

// For private user data — never cache on CDN
header("Cache-Control: private, no-store");
?>
```

### Popular CDN Providers

| CDN | Best known for |
|---|---|
| **Cloudflare** | Free tier, DDoS protection, DNS, many extra features |
| **AWS CloudFront** | Tightly integrated with AWS ecosystem |
| **Fastly** | Highly programmable, used by GitHub, Twitter |
| **BunnyCDN** | Affordable, fast, simpler pricing |

---

## HTTP Caching Deep Dive

Understanding caching headers is essential — wrong cache settings cause stale content or wasted server resources.

### Cache-Control Directives

```http
Cache-Control: public, max-age=86400
```

| Directive | Meaning |
|---|---|
| `public` | Can be cached by browsers AND CDNs |
| `private` | Only browser cache — no CDN/shared caches |
| `no-store` | Never cache anywhere — always fetch fresh (sensitive data) |
| `no-cache` | Cache it, but revalidate with server before using |
| `max-age=N` | Cache for N seconds (browser) |
| `s-maxage=N` | Cache for N seconds (CDN/shared cache only) |
| `must-revalidate` | After max-age, must revalidate — no stale serving |
| `immutable` | Content will NEVER change — browser won't revalidate even after max-age |
| `stale-while-revalidate=N` | Serve stale while revalidating in background |

### ETag — Conditional Caching

```php
<?php
// ETag-based conditional caching
function serveWithETag(string $content): void {
    $etag = '"' . md5($content) . '"';

    // If client sends If-None-Match and it matches our ETag — not modified
    if (isset($_SERVER["HTTP_IF_NONE_MATCH"]) &&
        $_SERVER["HTTP_IF_NONE_MATCH"] === $etag) {
        http_response_code(304);  // 304 Not Modified — no body needed
        exit();
    }

    // Otherwise send fresh response with ETag
    header("ETag: $etag");
    header("Cache-Control: public, max-age=3600");
    echo $content;
}

$content = generateExpensiveReport();
serveWithETag($content);
// Client sends: If-None-Match: "abc123"
// Server: if content unchanged → 304 (saves bandwidth)
//         if content changed → 200 + new content + new ETag
?>
```

### Last-Modified — Time-Based Conditional Caching

```php
<?php
function serveWithLastModified(string $filePath): void {
    $modified = filemtime($filePath);
    $modStr   = gmdate("D, d M Y H:i:s", $modified) . " GMT";

    if (isset($_SERVER["HTTP_IF_MODIFIED_SINCE"]) &&
        strtotime($_SERVER["HTTP_IF_MODIFIED_SINCE"]) >= $modified) {
        http_response_code(304);
        exit();
    }

    header("Last-Modified: $modStr");
    header("Cache-Control: public, max-age=3600");
    readfile($filePath);
}
?>
```

---

## Performance Essentials

Things every full stack developer must know to build fast web applications.

### Response Time Budget

```
The "2-second rule": users expect pages to load in under 2 seconds.
The "100ms rule":    interactions should feel instant (under 100ms).

Time budget breakdown:
  DNS lookup:       5-50ms
  TCP connection:   10-100ms
  TLS handshake:    20-100ms
  Server processing: 50-500ms   ← your PHP code runs here
  Response transfer: 10-500ms   ← depends on content size + bandwidth
  Browser rendering: 100-500ms
  ─────────────────────────
  Total target:     < 1000ms
```

### Key Performance Metrics

| Metric | What it measures | Target |
|---|---|---|
| **TTFB** (Time to First Byte) | When server starts sending response | < 200ms |
| **FCP** (First Contentful Paint) | When user sees first content | < 1.8s |
| **LCP** (Largest Contentful Paint) | When main content is visible | < 2.5s |
| **CLS** (Cumulative Layout Shift) | How much the page shifts during load | < 0.1 |
| **INP** (Interaction to Next Paint) | Responsiveness to user input | < 200ms |

---

### PHP Performance Checklist

```php
<?php
// 1. Use OPcache (covered in Part 2 — eliminates PHP recompilation)

// 2. Profile slow queries — N+1 query problem
// ❌ N+1: 1 query for users + N queries (one per user) for their orders
$users = fetchAllUsers();  // 1 query
foreach ($users as $user) {
    $user["orders"] = fetchOrdersForUser($user["id"]);  // N more queries!
}

// ✅ Eager loading: fetch all related data in one query
$users = fetchAllUsersWithOrders();  // 1 query with JOIN

// 3. Use indexes on database columns used in WHERE, JOIN, ORDER BY
// CREATE INDEX idx_user_email ON users(email);
// CREATE INDEX idx_orders_user_created ON orders(user_id, created_at);

// 4. Paginate everything — never SELECT * without LIMIT
$stmt = $pdo->prepare("SELECT id, name, email FROM users
                        WHERE active = 1
                        ORDER BY created_at DESC
                        LIMIT ? OFFSET ?");
$stmt->execute([$perPage, $offset]);

// 5. Only SELECT columns you need — never SELECT *
// ❌ SELECT * FROM users      (fetches all 20 columns)
// ✅ SELECT id, name, email  (fetches only what you need)

// 6. Cache expensive operations in Redis
$result = $redis->get("expensive_report") ?? computeAndCache();

// 7. Use connection pooling (PgBouncer for PostgreSQL, ProxySQL for MySQL)
// Reuses DB connections instead of creating new ones per request

// 8. Defer non-critical work to background jobs
// ❌ Sending an email in the middle of a request (slow!)
sendWelcomeEmail($user);  // User waits for email to send

// ✅ Queue it for a background worker
$queue->push(new SendWelcomeEmailJob($user));  // Returns instantly
// Worker processes the email asynchronously
?>
```

---

## Developer Tools for Full Stack Debugging

### Browser DevTools (F12) — Network Tab

The single most important tool for understanding web communication.

```
What you see in DevTools Network tab:
┌─────────────────────────────────────────────────────────────────┐
│ Name           Status  Type      Size     Time    Initiator     │
├─────────────────────────────────────────────────────────────────┤
│ / (HTML)       200     document  12.4 kB  180ms   navigation    │
│ style.css      200     css       48.2 kB  22ms    index.html    │
│ app.js         200     script    215 kB   95ms    index.html    │
│ logo.png       200     image     8.4 kB   12ms    style.css     │
│ api/users      200     json      2.1 kB   45ms    app.js        │
│ tracking.js    200     script    87 kB    130ms   index.html    │
│ analytics.png  200     image     43 B     8ms     tracking.js   │
└─────────────────────────────────────────────────────────────────┘
```

**Key things to inspect in DevTools:**
- Click any request → see full Request/Response headers
- "Waterfall" view → see which requests are parallel vs sequential
- Check "Stalled" / "Waiting (TTFB)" → server-side bottlenecks
- Filter by type (XHR, JS, CSS, Img) to find specific requests
- "Disable cache" checkbox → force fresh requests
- "Throttle" dropdown → simulate slow 3G connections

---

### cURL — Testing APIs from Terminal

```bash
# Basic GET request
curl https://api.example.com/users

# GET with headers (auth token)
curl -H "Authorization: Bearer eyJhbGci..." https://api.example.com/users/42

# POST with JSON body
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGci..." \
  -d '{"name": "Phyo", "email": "phyo@example.com"}'

# See full request and response headers
curl -v https://api.example.com/users

# Follow redirects
curl -L https://example.com/old-page

# Save response to file
curl -o response.json https://api.example.com/report

# Test CORS preflight
curl -X OPTIONS https://api.example.com/users \
  -H "Origin: https://app.example.com" \
  -H "Access-Control-Request-Method: POST" \
  -v
```

---

### Postman / Insomnia — API Testing GUI

- Visual tool for sending HTTP requests — no command line needed.
- Save requests into collections, share with team.
- Set up environments (dev, staging, prod) with different base URLs and auth tokens.
- Write test scripts to verify response structure and values.
- Generate API documentation automatically from saved requests.

---

### PHP-Specific Debugging

```php
<?php
// 1. Log to server error log
error_log("Debug: " . json_encode($data));
// → Appears in /var/log/nginx/error.log or /var/log/apache2/error.log

// 2. Dump variable and die (quick debug)
function dd(mixed ...$vars): never {
    header("Content-Type: text/plain");
    foreach ($vars as $var) {
        var_dump($var);
        echo "\n";
    }
    exit();
}

dd($request, $response, $userData);

// 3. Check request in PHP
error_log("Method: " . $_SERVER["REQUEST_METHOD"]);
error_log("Headers: " . json_encode(getallheaders()));
error_log("Body: " . file_get_contents("php://input"));

// 4. Laravel/Symfony: use the Debug Toolbar
// Shows queries, routes, session, request/response, timeline

// 5. Xdebug — step debugger (connects to VS Code / PhpStorm)
// Step through code line by line, inspect variables at each step
// Configure in php.ini:
// zend_extension=xdebug
// xdebug.mode=debug
// xdebug.start_with_request=yes
?>
```

---

## The Complete Full Stack Communication Map

This is the **big picture** — every layer of the full stack and how they communicate.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FULL STACK COMMUNICATION MAP                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  USER                                                               │
│    │ keyboard/click                                                  │
│    ▼                                                                 │
│  BROWSER (Client)                                                   │
│    │ DOM Events → JavaScript                                         │
│    │ fetch() / XMLHttpRequest / WebSocket / SSE / form submit       │
│    │ Protocol: HTTP/1.1, HTTP/2, HTTP/3, WebSocket, SSE             │
│    │ Port: 80 (HTTP), 443 (HTTPS)                                   │
│    ▼                                                                 │
│  DNS (Domain Name System)                                           │
│    │ domain → IP address                                             │
│    │ Protocol: DNS (UDP port 53)                                     │
│    ▼                                                                 │
│  CDN (Cloudflare, CloudFront, etc.)                                 │
│    │ Cache hit → return cached response                              │
│    │ Cache miss → forward to origin                                  │
│    │ Also: DDoS protection, WAF, SSL termination                     │
│    ▼                                                                 │
│  LOAD BALANCER                                                      │
│    │ Distribute traffic across multiple app servers                  │
│    │ Algorithms: round robin, least connections, IP hash             │
│    ▼                                                                 │
│  REVERSE PROXY (Nginx)                                              │
│    │ SSL termination (if not done at CDN)                            │
│    │ Rate limiting, IP blocking                                      │
│    │ Serve static files directly (CSS, JS, images)                  │
│    │ Route dynamic requests to PHP-FPM via FastCGI                  │
│    │ Gzip/Brotli compression                                         │
│    │ Security headers                                                │
│    ▼                                                                 │
│  APPLICATION SERVER (PHP-FPM)                                       │
│    │ OPcache: compiled bytecode (no recompilation)                   │
│    │ Runs your PHP code                                              │
│    │ Framework routing (Laravel, Symfony, plain PHP)                 │
│    │ Business logic, validation, auth checks                         │
│    ▼                         ▼              ▼                       │
│  CACHE LAYER           MESSAGE QUEUE    FILE STORAGE                │
│  (Redis/Memcached)     (Redis/RabbitMQ) (S3/Local disk)             │
│  Session storage       Async jobs        User uploads               │
│  Query caching         Email sending     Generated files             │
│    ▼                         ▼                                       │
│  DATABASE SERVER (MySQL/PostgreSQL)                                 │
│    │ Tables, indexes, foreign keys                                   │
│    │ Protocol: MySQL protocol (port 3306)                            │
│    │ ACID transactions                                               │
│    ▼                                                                 │
│  DISK / SSD (Physical Storage)                                      │
│                                                                     │
│  RESPONSE FLOWS BACK UP THROUGH THE SAME LAYERS ▲                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Quick Revision

- **REST APIs** use HTTP methods as verbs (GET/POST/PUT/PATCH/DELETE) and URLs as nouns (resources). URLs should never contain action verbs.
- **API versioning** via URL paths (`/api/v1/`) is the most common approach — always maintain old versions during transitions.
- **WebSockets** provide persistent, full-duplex (bi-directional) connections — perfect for chat, games, collaboration. Upgrades from HTTP via the `101 Switching Protocols` response.
- **SSE (Server-Sent Events)** is one-way (server → client only) real-time over regular HTTP — simpler than WebSockets, great for notifications and live feeds.
- **CDNs** serve cached content from edge nodes close to users — dramatically reducing latency for static and cacheable content.
- **Cache-Control headers:** `public` (CDN cacheable), `private` (browser only), `no-store` (never cache), `max-age` (TTL in seconds), `s-maxage` (CDN-specific TTL), `immutable` (never revalidate).
- **ETags** and **Last-Modified** enable conditional requests — server returns `304 Not Modified` if content hasn't changed, saving bandwidth.
- **N+1 query problem** — the most common PHP/database performance killer. Fix with eager loading (JOINs) instead of looping queries.
- **PHP performance:** OPcache (no recompilation), database indexes, LIMIT everything, SELECT only needed columns, Redis caching, background queues for slow tasks.
- **Browser DevTools Network tab** — inspect every request and response, headers, timing, waterfall. Master this tool.
- **cURL** — test any API from the terminal. Essential for backend debugging.
- The **full stack request path:** User → Browser → DNS → CDN → Load Balancer → Nginx → PHP-FPM → Cache/Queue/Files → Database → response flows back up.
- Understanding ALL these layers makes you a full stack developer — you can trace any problem from the browser to the database and back.