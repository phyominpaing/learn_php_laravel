# Backend Fundamentals — Web Services, Servers & How the Web Works

Before diving deeper into PHP backend development, it's essential to understand the **infrastructure** your code runs on. This note covers the foundational concepts every backend developer needs: what servers actually are, how clients and servers talk to each other, what ports do, and the critical difference between HTTP and HTTPS.

---

## Table of Contents

1. [What is a Web Service?](#what-is-a-web-service)
2. [What is a Web Server?](#what-is-a-web-server)
3. [How the Web Works — The Big Picture](#how-the-web-works--the-big-picture)
4. [Client, Server & Database — How They Relate](#client-server--database--how-they-relate)
5. [Types of Servers](#types-of-servers)
   - [Web Server](#web-server-detail)
   - [Database Server](#database-server-detail)
   - [File Server](#file-server-detail)
   - [Application Server](#application-server-detail)
6. [What is a Port?](#what-is-a-port)
7. [HTTP vs HTTPS](#http-vs-https)
8. [How to Get HTTPS (SSL/TLS Certificates)](#how-to-get-https-ssltls-certificates)
9. [Domain Names & DNS](#domain-names--dns)
10. [Other Essential Backend Concepts](#other-essential-backend-concepts)
11. [Quick Revision](#quick-revision)

---

## What is a Web Service?

- A **web service** is a way for two applications (often on different machines, possibly written in different languages) to **communicate over a network**, usually the internet.
- It exposes specific functionality through a well-defined interface — most commonly using **HTTP** as the transport and **JSON** or **XML** as the data format.
- Think of it as a "function call" that happens over the internet instead of within a single program.

```
Your App  →  HTTP Request  →  Web Service  →  Processes & responds  →  Your App receives data
```

### Real-World Example

```
Weather App on your phone
        ↓ sends request
Weather Web Service (e.g., OpenWeatherMap API)
        ↓ looks up data, responds with JSON
{
  "city": "Yangon",
  "temperature": 32,
  "condition": "Sunny"
}
        ↓
Weather App displays the result to you
```

### Common Types of Web Services

| Type | Description |
|---|---|
| **REST API** | The most common style today — uses standard HTTP methods (GET, POST, PUT, DELETE) and typically returns JSON |
| **SOAP** | An older, stricter, XML-based protocol — still used in some enterprise/banking systems |
| **GraphQL** | A flexible query language that lets clients request exactly the data they need |
| **gRPC** | A high-performance protocol using binary data, often used for service-to-service communication |

> 💡 **Tip:** When people say "I'm calling an API," they are almost always describing the act of consuming a **web service**. "API" (Application Programming Interface) is the broader term; a **web API** is simply an API exposed over the web.

---

## What is a Web Server?

- A **web server** is **software** (sometimes the term loosely also refers to the physical machine running that software) that:
  1. **Listens** for incoming requests from clients (usually browsers).
  2. **Processes** those requests.
  3. **Sends back a response** — typically an HTML page, JSON data, an image, or a file.

```
Browser sends request: "GET /index.html please"
        ↓
Web Server receives it, finds/generates the file
        ↓
Web Server sends response: the HTML content
        ↓
Browser renders the page
```

### Popular Web Server Software

| Software | Notes |
|---|---|
| **Apache (httpd)** | One of the oldest and most widely used; highly configurable via `.htaccess` |
| **Nginx** | Known for speed and efficiency, especially under heavy traffic; popular as a reverse proxy too |
| **LiteSpeed** | Commercial alternative, drop-in replacement for Apache, known for performance |
| **IIS** | Microsoft's web server, built into Windows Server |
| **Caddy** | Modern, simple to configure, automatic HTTPS out of the box |

```bash
# Example: starting PHP's own built-in dev web server (covered in installation notes)
php -S localhost:8000
```

> 💡 **Important distinction:** PHP itself is **not** a web server — it's a scripting language. In production, PHP code is usually executed by **Apache** or **Nginx** (with PHP-FPM), which act as the actual web server handling the HTTP traffic.

---

## How the Web Works — The Big Picture

Here's the complete journey of what happens when you type a URL and hit Enter:

```
1. You type: https://example.com
        ↓
2. Browser asks DNS: "What's the IP address for example.com?"
        ↓
3. DNS responds: "93.184.216.34"
        ↓
4. Browser sends an HTTP(S) request to that IP address, on a specific port (443 for HTTPS)
        ↓
5. Request arrives at the Web Server (e.g., Nginx)
        ↓
6. Web Server hands the request to the Application (e.g., PHP via PHP-FPM)
        ↓
7. PHP code runs — may query a Database Server for data
        ↓
8. Database Server returns the requested data to PHP
        ↓
9. PHP generates an HTML/JSON response
        ↓
10. Web Server sends the response back through the same path to your Browser
        ↓
11. Browser renders the page (or processes the JSON)
```

> 💡 **Mental model:** Think of the web server as a **receptionist**. It receives every visitor (request), checks what they need, routes them to the right department (PHP, database, files), and delivers the response back to them.

---

## Client, Server & Database — How They Relate

These three pieces form the backbone of almost every web application — commonly called the **three-tier architecture**.

```
┌─────────────┐         HTTP Request          ┌─────────────┐         SQL Query        ┌─────────────┐
│   CLIENT    │  ─────────────────────────►   │   SERVER    │  ──────────────────────► │  DATABASE   │
│  (Browser)  │                                │ (Web + App) │                           │   SERVER    │
│             │  ◄─────────────────────────   │             │  ◄────────────────────── │             │
└─────────────┘         HTTP Response          └─────────────┘         Query Result      └─────────────┘
```

### The Three Tiers

| Tier | Role | Example |
|---|---|---|
| **Client** | What the user interacts with — requests data, displays results | Chrome browser, mobile app |
| **Server** | Processes logic, handles requests, talks to the database | Apache/Nginx + PHP application |
| **Database** | Stores and retrieves persistent data | MySQL, PostgreSQL |

### Step-by-Step Example: Logging In

```
1. CLIENT (Browser)
   You type your username/password and click "Login"
   → Browser sends a POST request with your credentials

2. SERVER (PHP application)
   Receives the request
   → Validates the input format
   → Builds a SQL query: "SELECT * FROM users WHERE username = ?"
   → Sends that query to the Database Server

3. DATABASE SERVER (MySQL)
   Searches the `users` table
   → Finds a matching row (or doesn't)
   → Sends the result back to the PHP Server

4. SERVER (PHP application)
   Receives the result
   → Checks if the password matches (using password_verify())
   → Creates a session if successful
   → Sends an HTML/JSON response back

5. CLIENT (Browser)
   Receives the response
   → Redirects to the dashboard, or shows an error message
```

> ⚠️ **Important:** The **client never talks directly to the database**. The server always sits in between, acting as a gatekeeper. This is critical for **security** — if a browser could query the database directly, anyone could read or destroy your data.

---

## Types of Servers

The word "server" gets used for many different roles. Here's how the main ones differ.

| Server Type | Primary Job | Example Software |
|---|---|---|
| **Web Server** | Serves web pages and handles HTTP(S) requests | Apache, Nginx |
| **Application Server** | Runs your backend application logic | PHP-FPM, Node.js runtime |
| **Database Server** | Stores, retrieves, and manages structured data | MySQL, PostgreSQL, MongoDB |
| **File Server** | Stores and serves files (not necessarily over HTTP) | FTP server, Samba, NAS |
| **Mail Server** | Sends and receives email | Postfix, Exchange |
| **DNS Server** | Translates domain names into IP addresses | BIND, Cloudflare DNS |

---

### Web Server Detail

- Handles **HTTP/HTTPS requests** specifically.
- Serves static files directly (images, CSS, JS) and forwards dynamic requests (like `.php` files) to an application layer.

```php
<?php
// When your browser requests: https://example.com/products.php
// 1. Nginx/Apache (web server) receives the request
// 2. Sees it's a .php file → forwards it to PHP-FPM (application server)
// 3. PHP runs products.php, possibly queries the database
// 4. PHP sends the generated HTML back to the web server
// 5. Web server sends it to the browser
?>
```

---

### Database Server Detail

- A dedicated system (often a separate physical/virtual machine in production) that runs **database software** and manages all data storage, retrieval, and integrity.
- Your PHP application connects to it using credentials (host, username, password, database name).

```php
<?php
// Example: PHP connecting to a MySQL database server
$conn = new mysqli(
    "localhost",      // Database server host (could be a different machine in production)
    "db_user",         // Username
    "db_password",     // Password
    "my_database"      // Database name
);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

echo "Connected successfully to the database server!";
?>
```

> 💡 **In production:** Large applications often run the database server on a **completely separate machine** from the web server — for security isolation, independent scaling, and so a single server crash doesn't take down everything.

---

### File Server Detail

- Specializes in **storing and delivering files** — could be user uploads (profile photos, documents), static assets, backups.
- Can be part of the same machine as your web server, or a **separate dedicated service** like Amazon S3, Google Cloud Storage, or a NAS (Network Attached Storage).

```php
<?php
// Example: a PHP app storing an uploaded file
move_uploaded_file($_FILES["photo"]["tmp_name"], "/var/www/uploads/" . $_FILES["photo"]["name"]);

// In larger systems, this would instead be:
// uploadToS3($_FILES["photo"], "my-bucket-name");
?>
```

> 💡 **Why separate file storage?** As an app grows, storing files directly on the web server becomes risky (disk space limits, server crashes losing files, scaling issues). Cloud file storage services (S3, etc.) solve this by separating "where files live" from "what server is running your code."

---

### Application Server Detail

- Sits between the web server and your code — actually **executes your backend logic** (PHP, Python, Node.js, etc.).
- For PHP specifically, this role is filled by **PHP-FPM** (FastCGI Process Manager) when using Nginx, or the built-in **mod_php** module when using Apache.

```
Nginx (Web Server)  →  PHP-FPM (Application Server)  →  Runs your .php files  →  Returns HTML/JSON
```

---

## What is a Port?

- A **port** is a **numbered "doorway"** on a server that a specific type of traffic uses to enter or exit.
- A single server (one IP address) can run **many services simultaneously** — ports let the operating system know which service should handle which incoming traffic.
- Port numbers range from `0` to `65535`.

```
Think of an IP address like a building's street address.
A port is like a specific apartment/suite number within that building.

192.168.1.1:80   → "Building 192.168.1.1, Suite 80"  (the web server)
192.168.1.1:3306 → "Building 192.168.1.1, Suite 3306" (the MySQL database)
192.168.1.1:22   → "Building 192.168.1.1, Suite 22"   (SSH remote access)
```

### Common Well-Known Ports

| Port | Service | Purpose |
|---|---|---|
| **80** | HTTP | Unencrypted web traffic |
| **443** | HTTPS | Encrypted web traffic |
| **21** | FTP | File transfer |
| **22** | SSH | Secure remote terminal access |
| **25** | SMTP | Sending email |
| **53** | DNS | Domain name resolution |
| **3306** | MySQL | MySQL database connections |
| **5432** | PostgreSQL | PostgreSQL database connections |
| **6379** | Redis | Redis cache/database connections |
| **27017** | MongoDB | MongoDB database connections |

```bash
# Practical example: connecting to MySQL on its default port
mysql -h localhost -P 3306 -u root -p

# Running PHP's built-in server on a custom port
php -S localhost:8080
```

> 💡 **Tip:** Ports `80` and `443` are so standard for web traffic that browsers **assume them automatically** — you never need to type `https://example.com:443`, the browser already knows to use port 443 for `https://`.

> ⚠️ **Security Note:** Database ports (like `3306` for MySQL) should generally **never** be exposed directly to the public internet. They should only be accessible from your application server, often through a firewall rule restricting access to specific IPs.

---

## HTTP vs HTTPS

### HTTP (Port 80)

- **HTTP** stands for **HyperText Transfer Protocol** — the foundational protocol used to transfer web pages and data between client and server.
- Data sent over HTTP is **plain text** — anyone intercepting the traffic (on public Wi-Fi, for example) can read it.

```
Browser  ──────  "username=phyo&password=12345"  ──────  Server
                  (visible to anyone listening!)
```

### HTTPS (Port 443)

- **HTTPS** stands for **HyperText Transfer Protocol Secure** — it's HTTP, but **encrypted** using **TLS** (Transport Layer Security, the modern successor to SSL).
- Data is **scrambled** during transit — even if intercepted, it's unreadable without the decryption key.

```
Browser  ──────  "x7$kP9#mZ@qR2...(encrypted gibberish)"  ──────  Server
                  (unreadable even if intercepted)
```

### HTTP vs HTTPS — Comparison Table

| Feature | HTTP | HTTPS |
|---|---|---|
| Port | 80 | 443 |
| Encryption | ❌ None — plain text | ✅ Encrypted (TLS/SSL) |
| Data integrity | ❌ Can be tampered with in transit | ✅ Tamper-evident |
| Server identity verification | ❌ No way to confirm you're talking to the real server | ✅ Certificate proves server identity |
| Browser indicator | ⚠️ "Not Secure" warning | 🔒 Padlock icon |
| SEO impact | Lower ranking preference | Google favors HTTPS sites |
| Required for | Old/legacy sites, local development | Login forms, payments, modern websites (basically everything today) |

> ⚠️ **Critical Warning:** **Never** send sensitive data (passwords, credit card numbers, personal info) over plain HTTP. Modern browsers actively warn users with a "Not Secure" label on HTTP sites, and many features (like camera/location access, service workers) are **disabled entirely** on non-HTTPS sites.

---

### How HTTPS Actually Works (Simplified)

```
1. Browser connects to server, requests a secure connection
        ↓
2. Server presents its SSL/TLS Certificate (proves its identity)
        ↓
3. Browser verifies the certificate is valid and trusted
        ↓
4. Browser and Server perform a "handshake" — agree on encryption keys
        ↓
5. All further data is encrypted using those keys
        ↓
6. Browser shows the padlock 🔒 — connection is now secure
```

> 💡 **What is a "certificate," really?** It's a digital file issued by a trusted **Certificate Authority (CA)** that cryptographically proves "this server really is example.com" — preventing impersonation attacks.

---

## How to Get HTTPS (SSL/TLS Certificates)

To enable HTTPS on your website, you need an **SSL/TLS certificate**. Here are your main options:

### Option 1: Let's Encrypt (Free, Most Common)

- A **free**, automated Certificate Authority trusted by all major browsers.
- Certificates are valid for 90 days but **auto-renew** with the right tooling.
- Most hosting providers and control panels (cPanel, Plesk) offer one-click Let's Encrypt setup.

```bash
# Example: using Certbot (the standard tool for Let's Encrypt) on Ubuntu with Nginx
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d example.com -d www.example.com

# Certbot automatically configures Nginx and sets up auto-renewal
```

> 💡 **Tip:** Let's Encrypt is the standard choice for most personal projects, startups, and even many production businesses — it's genuinely free and just as secure as paid certificates.

---

### Option 2: Hosting Provider's Free SSL

- Many hosting platforms (Cloudflare, Vercel, Netlify, Hostinger, etc.) provide **free, automatic HTTPS** the moment you connect a domain — no manual setup required.

```
Example with Cloudflare:
1. Point your domain's nameservers to Cloudflare
2. Cloudflare automatically provisions an SSL certificate
3. HTTPS works immediately — zero configuration on your server
```

---

### Option 3: Paid SSL Certificates

- Purchased from Certificate Authorities like **DigiCert**, **Sectigo**, **GlobalSign**.
- Offer additional features like **Extended Validation (EV)** certificates (shows the company name in some browser UIs) — mainly relevant for large enterprises, banks, and e-commerce needing extra trust signals.
- For most projects, this level of certificate is **not necessary** — Let's Encrypt provides the same encryption strength.

---

### SSL Certificate Types — Quick Comparison

| Type | Cost | Validates | Best For |
|---|---|---|---|
| **DV** (Domain Validated) | Free–Low | Just confirms you own the domain | Personal sites, blogs, most apps |
| **OV** (Organization Validated) | Paid | Confirms the organization exists | Business websites |
| **EV** (Extended Validation) | Paid (higher) | Deep verification of the legal entity | Banks, large e-commerce, high-trust needs |

> 💡 **For almost every project you'll build:** A free **DV certificate from Let's Encrypt** is all you need. The encryption strength is identical across all certificate types — the differences are about identity verification level, not security strength.

---

### Local Development & HTTPS

- During local development (`localhost`), you typically **don't need HTTPS** — browsers treat `localhost` as a "secure context" automatically for testing purposes.
- If you need to test HTTPS locally (e.g., for features that require it), tools like **mkcert** can generate locally-trusted certificates.

```bash
# Example: mkcert for local HTTPS development
mkcert -install
mkcert localhost
```

---

## Domain Names & DNS

- A **domain name** (like `example.com`) is a human-friendly name that maps to a server's **IP address**.
- **DNS** (Domain Name System) is the internet's "phone book" — it translates domain names into IP addresses.

```
example.com  →  DNS lookup  →  93.184.216.34  (the actual server IP)
```

### Common DNS Record Types

| Record | Purpose |
|---|---|
| **A** | Maps a domain to an IPv4 address |
| **AAAA** | Maps a domain to an IPv6 address |
| **CNAME** | Maps a domain/subdomain to another domain name |
| **MX** | Specifies the mail server for the domain |
| **TXT** | Stores arbitrary text — often used for verification (e.g., proving domain ownership) |

> 💡 **Tip:** When you "point your domain" to a hosting provider, you're typically editing the domain's **A record** (or nameservers) to tell DNS which server's IP address to direct traffic to.

---

## Other Essential Backend Concepts

A few more terms you'll run into constantly as a backend developer:

| Term | Simple Explanation |
|---|---|
| **Localhost** | Your own computer, used for local development — always resolves to `127.0.0.1` |
| **IP Address** | A unique numerical address identifying a device on a network (e.g., `192.168.1.1`) |
| **Firewall** | A security system that controls which traffic/ports are allowed in or out of a server |
| **Load Balancer** | Distributes incoming traffic across multiple servers to handle high load |
| **CDN** (Content Delivery Network) | A network of distributed servers that cache and serve content closer to users geographically, for speed |
| **Reverse Proxy** | A server (like Nginx) that sits in front of your application server, forwarding requests and adding security/caching |
| **Environment Variables** | Configuration values (API keys, database passwords) stored outside your code for security and flexibility |
| **API Endpoint** | A specific URL where a web service/API can be accessed (e.g., `/api/users`) |
| **Status Code** | A 3-digit HTTP response code indicating the result (`200 OK`, `404 Not Found`, `500 Server Error`) |
| **Latency** | The time delay between a request being sent and a response being received |

```php
<?php
// Example: common HTTP status codes in a PHP context
http_response_code(200);  // OK — success
http_response_code(201);  // Created — new resource created successfully
http_response_code(400);  // Bad Request — client sent invalid data
http_response_code(401);  // Unauthorized — authentication required
http_response_code(403);  // Forbidden — authenticated but not allowed
http_response_code(404);  // Not Found — resource doesn't exist
http_response_code(500);  // Internal Server Error — something broke on the server
?>
```

> 💡 **Tip:** Understanding HTTP status codes is essential for backend work — your PHP scripts should return the **correct status code** for every response, not just `200` for everything. This is especially important when building APIs.

---

## Quick Revision

- A **web service** lets applications communicate over a network (usually via HTTP + JSON) — REST APIs are the most common type today.
- A **web server** is software (Apache, Nginx) that listens for HTTP(S) requests and sends back responses. PHP itself is not a web server — it needs one (or PHP-FPM) to run in production.
- The journey of a request: **Browser → DNS lookup → Web Server → Application (PHP) → Database → back up the chain**.
- **Three-tier architecture**: Client (browser) → Server (web + app logic) → Database. The client **never** talks directly to the database — the server is the gatekeeper.
- **Server types** differ by job: Web Server (HTTP), Application Server (runs your code), Database Server (stores data), File Server (stores files), each can live on separate machines in production.
- A **port** is a numbered doorway on a server letting multiple services run on one IP address. Port `80` = HTTP, `443` = HTTPS, `3306` = MySQL, `22` = SSH.
- **HTTP** (port 80) sends data in plain text — anyone can read it if intercepted. **HTTPS** (port 443) encrypts data using TLS/SSL — essential for any site handling logins, payments, or personal data.
- HTTPS works via an **SSL/TLS certificate** issued by a trusted Certificate Authority, proving the server's identity and enabling encryption.
- Get HTTPS for free via **Let's Encrypt** (most common, auto-renewing) or your **hosting provider's built-in SSL** (Cloudflare, Vercel, etc.). Paid certificates (OV/EV) are mainly for large enterprises needing extra identity verification — not stronger encryption.
- **DNS** translates human-readable domain names into IP addresses, primarily via **A records**.
- Know your **HTTP status codes**: `200` success, `400` bad request, `401`/`403` auth issues, `404` not found, `500` server error — always return the correct one in your backend code.