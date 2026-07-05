# Modern Web Communication — Part 2: How Web Servers Work

Understanding what happens *inside* a web server — from the moment a request arrives to the moment a response leaves — is essential knowledge for every full stack developer. This is where your code meets the real world.

---

## Table of Contents

1. [What Happens Inside a Web Server](#what-happens-inside-a-web-server)
2. [Static vs Dynamic Content](#static-vs-dynamic-content)
3. [Web Server Software Deep Dive](#web-server-software-deep-dive)
   - [Apache](#apache)
   - [Nginx](#nginx)
   - [Apache vs Nginx](#apache-vs-nginx)
4. [PHP and the Web Server — How They Connect](#php-and-the-web-server--how-they-connect)
   - [mod_php (Apache)](#mod_php-apache)
   - [PHP-FPM + Nginx](#php-fpm--nginx)
   - [CGI vs FastCGI vs PHP-FPM](#cgi-vs-fastcgi-vs-php-fpm)
5. [Virtual Hosts — One Server, Many Websites](#virtual-hosts--one-server-many-websites)
6. [Reverse Proxy](#reverse-proxy)
7. [Load Balancing](#load-balancing)
8. [Caching Layers](#caching-layers)
9. [Web Server Configuration Essentials](#web-server-configuration-essentials)
10. [How a Request Travels Through the Full Stack](#how-a-request-travels-through-the-full-stack)
11. [Quick Revision](#quick-revision)

---

## What Happens Inside a Web Server

When a request arrives, a web server performs a series of steps before sending a response. Understanding this pipeline is key to debugging performance, security, and routing issues.

```
┌─────────────────────────────────────────────────────────────────┐
│                  WEB SERVER REQUEST PIPELINE                    │
│                                                                 │
│  Incoming Request                                               │
│       │                                                         │
│       ▼                                                         │
│  1. Accept TCP Connection (port 80 or 443)                      │
│       │                                                         │
│       ▼                                                         │
│  2. TLS Decryption (HTTPS only)                                 │
│       │                                                         │
│       ▼                                                         │
│  3. Parse HTTP Request (method, path, headers, body)            │
│       │                                                         │
│       ▼                                                         │
│  4. Apply Server Rules                                          │
│     ├── Virtual host matching (which site?)                     │
│     ├── URL rewriting (.htaccess / nginx rewrite)               │
│     ├── Access control (IP blocking, auth)                      │
│     └── Rate limiting                                           │
│       │                                                         │
│       ▼                                                         │
│  5. Route the Request                                           │
│     ├── Static file? → Serve directly from disk                 │
│     └── Dynamic? → Pass to PHP-FPM / app server                │
│       │                                                         │
│       ▼                                                         │
│  6. Build HTTP Response                                         │
│     ├── Set status code                                         │
│     ├── Set response headers                                    │
│     └── Attach body (HTML, JSON, file, etc.)                    │
│       │                                                         │
│       ▼                                                         │
│  7. Compress Response (gzip/brotli if client supports it)       │
│       │                                                         │
│       ▼                                                         │
│  8. Send Response + Log the Request                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Static vs Dynamic Content

One of the most important distinctions in web serving is whether the content being served is **static** or **dynamic**.

### Static Content

- Files that exist exactly as-is on disk — the server reads the file and sends it directly.
- No code is executed — the server just reads and sends bytes.
- **Fast** — the server doesn't need to think, just fetch and serve.
- Examples: `.html`, `.css`, `.js`, `.jpg`, `.png`, `.pdf`, font files, `.ico`.

```
Browser → GET /logo.png
           ↓
Web Server → Reads /var/www/html/logo.png from disk
           ↓
Browser ← 200 OK + [binary image bytes]
```

---

### Dynamic Content

- Content **generated on-the-fly** by server-side code (PHP, Python, Node.js, etc.).
- The server executes code, often querying a database, and builds the response.
- **Slower** than static — but much more powerful.
- Examples: product pages, user dashboards, API responses, search results.

```
Browser → GET /products?category=shoes
           ↓
Web Server → Detects .php extension (or configured route)
           ↓
PHP executes → Queries MySQL → Builds HTML
           ↓
Web Server ← HTML response from PHP
           ↓
Browser ← 200 OK + [generated HTML]
```

> 💡 **Performance tip:** Serve as much as possible as **static** content. Use caching to turn dynamic responses into temporarily static ones. A modern stack serves static files directly from Nginx (extremely fast) and only hits PHP for truly dynamic requests.

---

## Web Server Software Deep Dive

### Apache

- The **oldest** and most widely deployed web server — been around since 1995.
- Extremely flexible — almost every behavior can be configured via `.htaccess` files in any directory.
- Uses a **process/thread-based** model — creates a new process or thread for each connection.
- Native PHP integration via the **mod_php** module — PHP runs inside Apache's process.

**Key Apache concepts:**

```apache
# .htaccess — Apache's per-directory config file
# This is what makes WordPress, Laravel, etc. work on Apache

# Enable URL rewriting
RewriteEngine On

# Route all requests through index.php (Front Controller pattern)
RewriteCond %{REQUEST_FILENAME} !-f   # Not a real file
RewriteCond %{REQUEST_FILENAME} !-d   # Not a real directory
RewriteRule ^(.*)$ index.php [QSA,L]  # Send to index.php

# Force HTTPS
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

# Block access to sensitive files
<Files ".env">
    Order allow,deny
    Deny from all
</Files>

# Set CORS headers
Header set Access-Control-Allow-Origin "https://app.example.com"

# Enable gzip compression
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/css application/javascript
</IfModule>
```

---

### Nginx

- Pronounced **"engine-x"** — created in 2004 specifically to handle high concurrency.
- Uses an **event-driven, non-blocking** model — one worker process handles thousands of connections simultaneously without creating new threads.
- Does **not** run PHP natively — always delegates to **PHP-FPM** via FastCGI.
- The go-to choice for high-traffic production servers and modern deployments.

**Key Nginx concepts:**

```nginx
# /etc/nginx/sites-available/myapp.conf

server {
    listen 80;
    server_name myapp.example.com;

    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name myapp.example.com;

    # SSL certificate (from Let's Encrypt)
    ssl_certificate     /etc/letsencrypt/live/myapp.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myapp.example.com/privkey.pem;

    # Web root
    root /var/www/myapp/public;
    index index.php index.html;

    # Serve static files directly (very fast — Nginx handles it alone)
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff2)$ {
        expires 1y;                  # Cache static assets for 1 year
        add_header Cache-Control "public, immutable";
        try_files $uri =404;
    }

    # Front Controller — route everything through index.php
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # Pass PHP requests to PHP-FPM
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # Block access to hidden files (.env, .git, etc.)
    location ~ /\. {
        deny all;
    }

    # Enable gzip compression
    gzip on;
    gzip_types text/html text/css application/json application/javascript;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
}
```

---

### Apache vs Nginx

| Feature | Apache | Nginx |
|---|---|---|
| Architecture | Process/thread per connection | Event-driven, async (1 process → 1000s of connections) |
| Performance under load | Good | Excellent — handles high concurrency much better |
| Static file serving | Good | Excellent — faster than Apache |
| PHP integration | mod_php (runs PHP inside Apache) | PHP-FPM via FastCGI (separate process) |
| Config files | Per-directory `.htaccess` (flexible) | Central config only (faster, no per-request config read) |
| URL rewriting | `.htaccess` RewriteRule | `try_files` in server block |
| Learning curve | Moderate (`.htaccess` is forgiving) | Steeper (all config centralized) |
| Best for | Shared hosting, WordPress, legacy apps | High-traffic production, microservices, API servers |

> 💡 **In the real world:** Many production stacks use Nginx as the front-facing server (handling SSL, static files, compression) and Apache behind it for PHP — combining both strengths. Or simply Nginx + PHP-FPM directly — the modern standard.

---

## PHP and the Web Server — How They Connect

PHP does not run as a standalone web server. It needs to be connected to a web server. There are three main ways this connection works.

### mod_php (Apache)

- PHP is loaded as a **module inside Apache**.
- Every Apache worker process has PHP embedded in it.
- Simple to set up — PHP files are handled automatically.
- **Downside:** Even requests for static files (images, CSS) go through a PHP-aware Apache process, wasting memory.

```
Browser Request
      ↓
Apache (with mod_php embedded)
      ↓ (if .php file)
PHP executes inside Apache's process
      ↓
Response sent
```

---

### PHP-FPM + Nginx

- **PHP-FPM** (FastCGI Process Manager) is a completely **separate service** from the web server.
- Nginx handles HTTP — serves static files directly, and passes PHP requests via FastCGI protocol to PHP-FPM.
- PHP-FPM manages a **pool of PHP worker processes** — handles multiple PHP requests in parallel.
- This is the **modern standard** for production PHP deployments.

```
Browser Request
      ↓
Nginx (web server)
      ├── Static file (.jpg, .css, .js) → Serves directly ✅ (fast!)
      └── PHP file (.php) → FastCGI → PHP-FPM
                                           ↓
                                    PHP worker process executes
                                           ↓
                                    Response → Nginx → Browser
```

```bash
# Check PHP-FPM status
systemctl status php8.3-fpm

# PHP-FPM pool configuration (/etc/php/8.3/fpm/pool.d/www.conf)
# pm = dynamic                   — manage workers dynamically
# pm.max_children = 50           — max simultaneous PHP processes
# pm.start_servers = 5           — start with 5 workers
# pm.min_spare_servers = 5       — keep at least 5 idle workers
# pm.max_spare_servers = 35      — keep at most 35 idle workers
# pm.max_requests = 500          — restart each worker after 500 requests (prevents memory leaks)
```

---

### CGI vs FastCGI vs PHP-FPM

| Method | How it works | Performance | Modern? |
|---|---|---|---|
| **CGI** | New process spawned for every single request | Very slow (process start overhead per request) | ❌ Obsolete |
| **FastCGI** | Long-running process handles many requests | Fast | ✅ Yes |
| **PHP-FPM** | Pool of FastCGI PHP processes, managed efficiently | Best | ✅ Modern standard |
| **mod_php** | PHP embedded in Apache process | Fast but inflexible | 🟡 Legacy |

---

## Virtual Hosts — One Server, Many Websites

A **virtual host** lets one physical server (one IP address) host **multiple completely separate websites**.

The web server reads the `Host` header from the request to determine which website to serve.

```
One server at IP: 203.0.113.1
Hosts 3 websites:
  - blog.example.com   → /var/www/blog
  - shop.example.com   → /var/www/shop
  - api.example.com    → /var/www/api
```

```nginx
# Nginx virtual hosts — 3 separate server blocks

server {
    server_name blog.example.com;
    root /var/www/blog/public;
    # ... blog-specific config
}

server {
    server_name shop.example.com;
    root /var/www/shop/public;
    # ... shop-specific config
}

server {
    server_name api.example.com;
    root /var/www/api/public;
    # ... API-specific config
}
```

```apache
# Apache virtual hosts

<VirtualHost *:80>
    ServerName blog.example.com
    DocumentRoot /var/www/blog
</VirtualHost>

<VirtualHost *:80>
    ServerName shop.example.com
    DocumentRoot /var/www/shop
</VirtualHost>
```

> 💡 **This is how shared hosting works.** One server running Apache/Nginx hosts hundreds or thousands of websites by reading the `Host` header and routing to the right document root.

---

## Reverse Proxy

A **reverse proxy** sits in front of your actual application server and acts as an intermediary — clients talk to the proxy, not directly to your app.

```
Internet
   │
   ▼
┌─────────────────┐
│  Reverse Proxy  │  (Nginx, Cloudflare, AWS ALB)
│  Port 80/443    │
└────────┬────────┘
         │
    Distributes to:
    ├── App Server 1 (PHP-FPM on port 9000)
    ├── App Server 2 (Node.js on port 3000)
    └── App Server 3 (Python on port 5000)
```

### What a Reverse Proxy Does

- **SSL Termination** — handles HTTPS, decrypts traffic, forwards plain HTTP to app servers (so app servers don't need to deal with SSL).
- **Load Balancing** — distributes requests across multiple app server instances.
- **Caching** — caches responses, reducing load on app servers.
- **Compression** — compresses responses with gzip/brotli before sending to clients.
- **Security** — hides the real app server, can block malicious requests, rate-limit.
- **Routing** — routes different paths to different services (`/api/` → Node.js, `/app/` → PHP).

```nginx
# Nginx as reverse proxy for a Node.js app
server {
    listen 443 ssl;
    server_name app.example.com;

    # Proxy all requests to Node.js running on port 3000
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## Load Balancing

When traffic grows beyond what one server can handle, you scale **horizontally** — multiple servers share the load. A **load balancer** distributes incoming requests among them.

```
                    ┌─────────────────┐
                    │  Load Balancer  │
                    │  (Nginx/AWS ALB)│
                    └────────┬────────┘
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │  App     │  │  App     │  │  App     │
        │ Server 1 │  │ Server 2 │  │ Server 3 │
        └──────────┘  └──────────┘  └──────────┘
              │              │              │
              └──────────────┼──────────────┘
                             ▼
                    ┌─────────────────┐
                    │ Shared Database │
                    │ (MySQL/Redis)   │
                    └─────────────────┘
```

### Load Balancing Algorithms

| Algorithm | How it works | Best for |
|---|---|---|
| **Round Robin** | Each server gets requests in turn (1→2→3→1→2→3) | Servers with equal capacity |
| **Least Connections** | Send to server with fewest active connections | Requests of varying duration |
| **IP Hash** | Same IP always goes to same server | Session-based apps (sticky sessions) |
| **Weighted** | Servers get traffic proportional to their weight | Servers with different capacities |

```nginx
# Nginx load balancing configuration
upstream app_servers {
    least_conn;  # Use least-connections algorithm

    server 10.0.0.1:9000 weight=3;  # Gets 3x traffic (powerful server)
    server 10.0.0.2:9000 weight=1;  # Gets 1x traffic
    server 10.0.0.3:9000 backup;    # Only used if others are down
}

server {
    location / {
        proxy_pass http://app_servers;
    }
}
```

> ⚠️ **Sessions and Load Balancing:** If your app uses server-side sessions (`$_SESSION`), a user's requests must hit the **same** server every time — or session data will be lost. Solutions: **sticky sessions** (IP hash), or store sessions in a **shared store** like Redis that all servers can access.

---

## Caching Layers

Caching is one of the most powerful tools for performance. The web has multiple caching layers, each serving a different purpose.

```
┌─────────────────────────────────────────────────────────────────┐
│                        CACHING LAYERS                           │
│                                                                 │
│  Browser Cache         → Stores CSS, JS, images locally        │
│       ↓ (miss)                                                  │
│  CDN Cache             → Globally distributed edge cache       │
│       ↓ (miss)                                                  │
│  Reverse Proxy Cache   → Nginx FastCGI cache / Varnish         │
│       ↓ (miss)                                                  │
│  Application Cache     → Redis/Memcached (PHP-level caching)   │
│       ↓ (miss)                                                  │
│  Database Query Cache  → MySQL query cache / OPcache           │
│       ↓ (miss)                                                  │
│  Database Disk         → The actual data                       │
└─────────────────────────────────────────────────────────────────┘
```

### Browser Cache

Controlled by `Cache-Control` and `Expires` response headers. The browser stores files locally and reuses them without making a network request.

```php
<?php
// Cache a response for 1 hour in the browser
header("Cache-Control: public, max-age=3600");

// Cache static assets forever (use content hashing for cache busting)
header("Cache-Control: public, max-age=31536000, immutable");

// Never cache (for dynamic, private content)
header("Cache-Control: no-store, no-cache, must-revalidate");
header("Pragma: no-cache");
?>
```

---

### OPcache — PHP's Built-in Performance Cache

- PHP normally compiles `.php` source code to bytecode on **every single request** — very slow.
- **OPcache** compiles each PHP file once and stores the compiled bytecode in memory.
- Subsequent requests use the cached bytecode — **no recompilation needed**.
- This is one of the most impactful PHP performance improvements — often 5-10x faster.

```ini
; php.ini — Enable OPcache
opcache.enable=1
opcache.memory_consumption=256         ; MB of RAM for bytecode cache
opcache.max_accelerated_files=20000    ; How many files to cache
opcache.validate_timestamps=0          ; In production: disable timestamp checks for speed
                                       ; (requires manual cache clear after deploy)
opcache.revalidate_freq=0              ; Revalidation frequency in seconds (0 = never in prod)
```

```php
<?php
// Check OPcache status
print_r(opcache_get_status());

// Clear OPcache after deployment
opcache_reset();  // Or: opcache_invalidate('/path/to/changed/file.php');
?>
```

---

### Application-Level Cache (Redis)

```php
<?php
// Caching expensive database queries in Redis

$redis = new Redis();
$redis->connect("127.0.0.1", 6379);

function getProductList(int $page): array {
    global $redis;

    $cacheKey = "products:page:$page";

    // Try cache first
    $cached = $redis->get($cacheKey);
    if ($cached !== false) {
        return json_decode($cached, true);  // Cache hit — return instantly
    }

    // Cache miss — query the database (slow)
    $products = queryDatabase("SELECT * FROM products LIMIT 20 OFFSET ?", [$page * 20]);

    // Store in cache for 5 minutes (300 seconds)
    $redis->setex($cacheKey, 300, json_encode($products));

    return $products;
}
?>
```

---

## Web Server Configuration Essentials

Things every full stack developer must know how to configure.

### Directory Structure for a PHP App

```
/var/www/myapp/
├── public/            ← Web root (only this directory is publicly accessible!)
│   ├── index.php      ← Front controller — all requests go through here
│   ├── assets/
│   │   ├── css/
│   │   ├── js/
│   │   └── images/
│   └── uploads/       ← User uploads (never execute PHP here!)
├── app/               ← Application code (not publicly accessible)
├── config/            ← Config files (never publicly accessible)
├── vendor/            ← Composer dependencies (never publicly accessible)
├── .env               ← Environment variables (NEVER publicly accessible)
└── composer.json
```

> ⚠️ **Critical Security Rule:** Your web server's document root should point to the `public/` folder — **never** to the project root. This ensures `.env`, database credentials, and application source code are **never directly accessible** via a URL.

---

### The Front Controller Pattern

Modern PHP frameworks (Laravel, Symfony) use a single `index.php` as the **front controller** — every request goes through it.

```php
<?php
// public/index.php — the front controller

// Bootstrap the application
require_once __DIR__ . "/../vendor/autoload.php";

// Load environment variables
$dotenv = Dotenv\Dotenv::createImmutable(__DIR__ . "/../");
$dotenv->load();

// Create the application
$app = new Application();

// Route the request
$request = Request::fromGlobals();
$response = $app->handle($request);
$response->send();
```

```nginx
# Nginx config that makes the front controller work
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
# → Try to serve the file directly ($uri)
# → If not found, try as directory ($uri/)
# → If still not found, send to index.php (front controller)
```

---

### Environment-Specific Configuration

```bash
# .env file (NEVER commit to version control!)
APP_ENV=production
APP_DEBUG=false
APP_URL=https://myapp.example.com

DB_HOST=db.internal.example.com
DB_PORT=3306
DB_DATABASE=myapp_prod
DB_USERNAME=myapp_user
DB_PASSWORD=super_secret_password_here

REDIS_HOST=redis.internal.example.com
REDIS_PORT=6379

MAIL_HOST=smtp.mailgun.org
MAIL_USERNAME=noreply@myapp.com
MAIL_PASSWORD=mail_api_key_here
```

```php
<?php
// Accessing environment variables safely
$dbHost = getenv("DB_HOST") ?: "localhost";
$debug  = getenv("APP_DEBUG") === "true";

// Never hardcode credentials in PHP files!
// ❌ $conn = new mysqli("production-db.example.com", "root", "hardcoded_password");
// ✅ $conn = new mysqli(getenv("DB_HOST"), getenv("DB_USERNAME"), getenv("DB_PASSWORD"));
?>
```

---

## How a Request Travels Through the Full Stack

Let's put it all together with a complete real-world example — a user viewing their order history on an e-commerce site.

```
USER ACTION: Clicks "My Orders" on the website

─────────────────────────────────────────────────────────────
STEP 1 — BROWSER (Client)
─────────────────────────────────────────────────────────────
  Browser checks its cache → no cached version
  Browser looks up DNS for shop.example.com → 203.0.113.1
  Browser opens HTTPS connection (TLS handshake)
  Browser sends:
    GET /orders HTTP/2
    Host: shop.example.com
    Authorization: Bearer eyJhbGci...
    Cookie: session_id=abc123
    Accept: text/html

─────────────────────────────────────────────────────────────
STEP 2 — CDN EDGE NODE (e.g., Cloudflare)
─────────────────────────────────────────────────────────────
  CDN receives request
  Checks its cache → not cached (it's user-specific data)
  Forwards request to origin server: 10.0.0.1

─────────────────────────────────────────────────────────────
STEP 3 — LOAD BALANCER / REVERSE PROXY (Nginx)
─────────────────────────────────────────────────────────────
  Nginx receives request on port 443
  SSL already terminated by Cloudflare (or Nginx terminates it)
  Applies rate limiting → user is within limit ✅
  Checks URL: /orders → matches location block
  Checks file: /var/www/shop/public/orders doesn't exist
  Routes to: index.php (front controller)
  Passes via FastCGI to PHP-FPM

─────────────────────────────────────────────────────────────
STEP 4 — PHP-FPM (Application Server)
─────────────────────────────────────────────────────────────
  PHP-FPM worker picks up the request
  OPcache serves compiled bytecode (no recompilation)
  index.php bootstraps the framework
  Router matches /orders → OrderController::index()

─────────────────────────────────────────────────────────────
STEP 5 — APPLICATION CODE (PHP / Controller)
─────────────────────────────────────────────────────────────
  Authenticates request:
    Reads JWT token from Authorization header
    Validates token signature → user_id = 42 ✅
  Checks Redis cache:
    Key: "orders:user:42:page:1"
    → MISS (not cached)
  Queries MySQL Database:
    SELECT * FROM orders
    WHERE user_id = 42
    ORDER BY created_at DESC
    LIMIT 10 OFFSET 0
  Caches result in Redis for 5 minutes

─────────────────────────────────────────────────────────────
STEP 6 — DATABASE SERVER (MySQL)
─────────────────────────────────────────────────────────────
  Executes query using indexes on user_id and created_at
  Returns 10 rows of order data to PHP

─────────────────────────────────────────────────────────────
STEP 7 — BACK IN PHP (Response Building)
─────────────────────────────────────────────────────────────
  Controller receives order data
  Passes to view template → generates HTML
  Sets response headers:
    Content-Type: text/html; charset=UTF-8
    Cache-Control: private, no-store
    X-Request-ID: a1b2c3d4

─────────────────────────────────────────────────────────────
STEP 8 — RESPONSE TRAVELS BACK
─────────────────────────────────────────────────────────────
  PHP-FPM → Nginx (FastCGI response)
  Nginx compresses with gzip
  Nginx adds security headers
  Nginx → Cloudflare → User's Browser

─────────────────────────────────────────────────────────────
STEP 9 — BROWSER RENDERS
─────────────────────────────────────────────────────────────
  Browser receives HTML
  Parses — discovers CSS/JS/image URLs
  Browser checks cache for each asset:
    style.css  → CACHE HIT → no request needed
    app.js     → CACHE HIT → no request needed
    logo.png   → CACHE HIT → no request needed
  Renders the order history page
  Total time: ~180ms

─────────────────────────────────────────────────────────────
```

---

## Quick Revision

- A web server processes requests through a pipeline: accept connection → TLS decrypt → parse HTTP → apply rules → route (static or dynamic) → build response → compress → send.
- **Static content** (files) is served directly by the web server — fast, no code execution. **Dynamic content** is generated by application code (PHP) — slower but powerful.
- **Apache** uses a process/thread model, supports `.htaccess` per-directory config, integrates PHP via `mod_php`. Best for shared hosting and legacy apps.
- **Nginx** uses an event-driven async model, handles thousands of connections per worker, connects to PHP via PHP-FPM (FastCGI). The modern standard for high-traffic production.
- **PHP-FPM** is a separate pool of PHP worker processes that Nginx delegates dynamic requests to via FastCGI. This is the modern PHP execution model.
- **Virtual hosts** let one server host multiple websites — the web server reads the `Host` header to determine which site to serve.
- A **reverse proxy** (usually Nginx) sits in front of app servers — handling SSL termination, compression, caching, routing, and security before passing requests to application servers.
- **Load balancing** distributes requests across multiple server instances. Algorithms: round robin, least connections, IP hash, weighted. Use Redis for shared sessions when load balancing PHP.
- **Caching layers** (browser cache → CDN → proxy cache → Redis → OPcache → DB query cache) reduce load at every level. OPcache is essential — it eliminates PHP recompilation.
- **Front Controller pattern** — all requests route through `index.php`. Nginx's `try_files` directive enables this.
- **Web root = `public/` only** — application code, `.env`, and `vendor/` must never be directly accessible.
- The full stack request journey: Browser → DNS → CDN → Load Balancer → Nginx → PHP-FPM → App Code → Redis → MySQL → back up the chain → Browser renders.