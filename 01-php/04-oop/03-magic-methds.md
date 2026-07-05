# PHP Modern OOP — Part 3: Magic Methods, Advanced OOP & Design Patterns

**Magic methods and design patterns** are what separate junior developers from senior ones — magic methods let you control how objects behave in special situations, while design patterns are proven solutions to recurring architectural problems.

---

## Table of Contents

1. [Magic Methods](#magic-methods)
   - [`__toString()`](#__tostring)
   - [`__get()` / `__set()` / `__isset()` / `__unset()`](#__get--__set--__isset--__unset)
   - [`__call()` / `__callStatic()`](#__call--__callstatic)
   - [`__clone()`](#__clone)
   - [`__invoke()`](#__invoke)
   - [`__debugInfo()`](#__debuginfo)
2. [Named Arguments with OOP](#named-arguments-with-oop)
3. [Enums (PHP 8.1+)](#enums-php-81)
4. [Fibers (PHP 8.1+)](#fibers-php-81)
5. [Design Patterns](#design-patterns)
   - [Singleton](#singleton-pattern)
   - [Factory & Factory Method](#factory--factory-method-pattern)
   - [Builder](#builder-pattern)
   - [Repository](#repository-pattern)
   - [Observer](#observer-pattern)
   - [Strategy](#strategy-pattern)
   - [Dependency Injection](#dependency-injection-pattern)
6. [Quick Revision](#quick-revision)

---

## Magic Methods

**Magic methods** are special methods with double-underscore prefixes (`__`) that PHP calls automatically in specific situations — when an object is converted to string, when you access a non-existent property, when you call an object like a function, etc.

---

### `__toString()`

- Called automatically when an object is used **as a string** — in `echo`, string concatenation, `printf`, etc.
- Must return a `string`.

```php
<?php
class Money {
    public function __construct(
        private float  $amount,
        private string $currency = "USD"
    ) {}

    public function __toString(): string {
        return number_format($this->amount, 2) . " " . $this->currency;
    }

    public function add(Money $other): static {
        return new static($this->amount + $other->amount, $this->currency);
    }
}

$price    = new Money(19.99);
$shipping = new Money(5.00);
$total    = $price->add($shipping);

echo $price;    // 19.99 USD  ← __toString() called automatically
echo $shipping; // 5.00 USD
echo $total;    // 24.99 USD

echo "Total: $total";   // Total: 24.99 USD ← works in double-quoted strings too
echo "Price is " . $price . " today";  // works in concatenation
?>
```

---

### `__get()` / `__set()` / `__isset()` / `__unset()`

- **`__get($name)`** — called when accessing a property that doesn't exist or is inaccessible.
- **`__set($name, $value)`** — called when setting a property that doesn't exist or is inaccessible.
- **`__isset($name)`** — called when `isset()` or `empty()` is used on an inaccessible/non-existent property.
- **`__unset($name)`** — called when `unset()` is used on an inaccessible property.

```php
<?php
// Dynamic property bag — flexible storage with validation
class Config {
    private array $data = [];
    private array $protected = ["app_key", "db_password"];

    public function __get(string $name): mixed {
        if (!array_key_exists($name, $this->data)) {
            throw new \RuntimeException("Config key '$name' does not exist");
        }
        return $this->data[$name];
    }

    public function __set(string $name, mixed $value): void {
        if (in_array($name, $this->protected, true)) {
            throw new \RuntimeException("Cannot set protected key '$name'");
        }
        $this->data[$name] = $value;
    }

    public function __isset(string $name): bool {
        return isset($this->data[$name]);
    }

    public function __unset(string $name): void {
        unset($this->data[$name]);
    }
}

$config = new Config();
$config->debug    = true;       // __set() called
$config->app_name = "MyApp";    // __set() called

echo $config->app_name;         // MyApp — __get() called
var_dump(isset($config->debug)); // bool(true) — __isset() called

unset($config->debug);          // __unset() called
var_dump(isset($config->debug)); // bool(false)

// $config->app_key = "secret"; // ❌ Throws RuntimeException — protected key
?>
```

---

### `__call()` / `__callStatic()`

- **`__call($name, $args)`** — called when an **inaccessible or undefined method** is called on an object.
- **`__callStatic($name, $args)`** — same, but for static method calls.
- Powerful for building **proxy objects**, **dynamic APIs**, and **magic method routing**.

```php
<?php
class APIClient {
    private string $baseUrl;
    private array  $defaultHeaders;

    public function __construct(string $baseUrl) {
        $this->baseUrl = rtrim($baseUrl, "/");
        $this->defaultHeaders = ["Content-Type" => "application/json"];
    }

    // Catches calls like $client->getUsers(), $client->postLogin(), etc.
    public function __call(string $method, array $args): mixed {
        // Parse method name: getUsers → GET /users, postLogin → POST /login
        preg_match('/^(get|post|put|patch|delete)(.+)$/i', $method, $matches);

        if (!$matches) {
            throw new \BadMethodCallException("Unknown method: $method");
        }

        $httpMethod = strtoupper($matches[1]);
        $endpoint   = strtolower(preg_replace('/([A-Z])/', '/$1', lcfirst($matches[2])));
        $body       = $args[0] ?? null;

        echo "Calling: $httpMethod {$this->baseUrl}/$endpoint\n";
        if ($body) echo "Body: " . json_encode($body) . "\n";

        // Here you'd actually make the HTTP request with cURL or Guzzle
        return ["status" => "success", "method" => $httpMethod, "endpoint" => $endpoint];
    }
}

$api = new APIClient("https://api.example.com");
$api->getUsers();                        // Calling: GET https://api.example.com/users
$api->postLogin(["email" => "phyo@example.com"]);  // Calling: POST .../login
$api->deleteProduct(["id" => 42]);       // Calling: DELETE .../product
?>
```

```php
<?php
// __callStatic — for fluent query builder style
class DB {
    private static array $query = [];

    public static function __callStatic(string $method, array $args): static {
        $obj = new static();

        if ($method === "table") {
            $obj::$query["table"] = $args[0];
        }
        return $obj;
    }

    public function where(string $condition): static {
        static::$query["where"][] = $condition;
        return $this;
    }
}
?>
```

---

### `__clone()`

- Called automatically when an object is **cloned** with the `clone` keyword.
- By default, PHP does a **shallow clone** — nested objects inside are still shared.
- Use `__clone()` to perform a **deep clone** — creating independent copies of nested objects.

```php
<?php
class Address {
    public function __construct(
        public string $street,
        public string $city
    ) {}
}

class Person {
    public Address $address;
    public array   $hobbies = [];

    public function __construct(
        public string $name,
        Address $address
    ) {
        $this->address = $address;
    }

    // Deep clone — create independent copies of contained objects
    public function __clone() {
        // Without this, $clone->address would point to the SAME Address object
        $this->address = clone $this->address;  // Create independent copy
        // Arrays are automatically deep-copied in PHP — no need to clone them
    }
}

$original = new Person("Phyo", new Address("Main St", "Yangon"));
$original->hobbies = ["coding", "reading"];

$clone = clone $original;

// Modify the clone
$clone->name            = "Alice";
$clone->address->city   = "Mandalay";
$clone->hobbies[]       = "gaming";

// Original is unaffected
echo $original->name;          // Phyo    ← unchanged
echo $original->address->city; // Yangon  ← unchanged (deep clone!)
print_r($original->hobbies);   // ["coding", "reading"] ← unchanged
?>
```

---

### `__invoke()`

- Called when an object is **used as a function** — called with `()`.
- Makes objects **callable** — they can be passed to `array_map`, `usort`, etc.

```php
<?php
class TaxCalculator {
    public function __construct(private float $rate) {}

    // Makes the object callable: $calc($price)
    public function __invoke(float $price): float {
        return $price * (1 + $this->rate);
    }
}

$calc = new TaxCalculator(0.09);  // 9% tax

echo $calc(100);   // 109  ← called as a function!
echo $calc(200);   // 218

// Works as a callable in higher-order functions
$prices    = [10.0, 25.0, 50.0, 100.0];
$withTax   = array_map($calc, $prices);  // Passes $calc as callback
print_r($withTax);  // [10.9, 27.25, 54.5, 109]

var_dump(is_callable($calc));  // bool(true) — TaxCalculator is callable!

// Another common use: middleware/pipeline handlers
class LogMiddleware {
    public function __invoke(array $request, callable $next): array {
        echo "Before: " . $request["path"] . "\n";
        $response = $next($request);
        echo "After: status {$response["status"]}\n";
        return $response;
    }
}
?>
```

---

### `__debugInfo()`

- Controls what `var_dump()` shows for an object — useful to hide sensitive data.

```php
<?php
class DatabaseConnection {
    public function __construct(
        private string $host,
        private string $username,
        private string $password,  // Should never appear in debug output!
        private string $database,
    ) {}

    public function __debugInfo(): array {
        return [
            "host"     => $this->host,
            "username" => $this->username,
            "password" => "***HIDDEN***",  // Mask sensitive data
            "database" => $this->database,
            "status"   => "connected",
        ];
    }
}

$db = new DatabaseConnection("localhost", "root", "super_secret_pass", "myapp");
var_dump($db);
// object(DatabaseConnection)#1 (4) {
//   ["host"]     => string(9) "localhost"
//   ["username"] => string(4) "root"
//   ["password"] => string(10) "***HIDDEN***"
//   ["database"] => string(5) "myapp"
//   ["status"]   => string(9) "connected"
// }
// Note: "super_secret_pass" never appears in debug output!
?>
```

---

## Named Arguments with OOP

```php
<?php
class Pagination {
    public function __construct(
        public int    $page     = 1,
        public int    $perPage  = 20,
        public string $sortBy   = "id",
        public string $sortDir  = "asc",
        public bool   $cache    = true,
    ) {}
}

// Named arguments — skip parameters you don't need to change
$pagination = new Pagination(page: 3, sortBy: "created_at", sortDir: "desc");
echo $pagination->page;    // 3
echo $pagination->perPage; // 20 (default)
echo $pagination->sortBy;  // created_at
?>
```

---

## Enums (PHP 8.1+)

- **Enums** (enumerations) are a type that represents a **fixed set of possible values**.
- Much safer than using string constants — PHP enforces that only valid values are used.
- Two kinds: **pure enums** (just labels) and **backed enums** (each case has a string or int value).

```php
<?php
// Pure enum — just named cases
enum Status {
    case Active;
    case Inactive;
    case Pending;
    case Banned;
}

// Backed enum — each case has a value (string or int)
enum Color: string {
    case Red   = "red";
    case Green = "green";
    case Blue  = "blue";
}

enum Priority: int {
    case Low    = 1;
    case Medium = 2;
    case High   = 3;
}
?>
```

```php
<?php
enum OrderStatus: string {
    case Pending   = "pending";
    case Confirmed = "confirmed";
    case Shipped   = "shipped";
    case Delivered = "delivered";
    case Cancelled = "cancelled";

    // Methods can be added to enums!
    public function label(): string {
        return match($this) {
            OrderStatus::Pending   => "⏳ Pending",
            OrderStatus::Confirmed => "✅ Confirmed",
            OrderStatus::Shipped   => "🚚 Shipped",
            OrderStatus::Delivered => "📦 Delivered",
            OrderStatus::Cancelled => "❌ Cancelled",
        };
    }

    public function canTransitionTo(OrderStatus $new): bool {
        return match($this) {
            OrderStatus::Pending   => $new === OrderStatus::Confirmed || $new === OrderStatus::Cancelled,
            OrderStatus::Confirmed => $new === OrderStatus::Shipped   || $new === OrderStatus::Cancelled,
            OrderStatus::Shipped   => $new === OrderStatus::Delivered,
            default                => false,
        };
    }
}

class Order {
    private OrderStatus $status = OrderStatus::Pending;

    public function transitionTo(OrderStatus $newStatus): void {
        if (!$this->status->canTransitionTo($newStatus)) {
            throw new \LogicException(
                "Cannot transition from {$this->status->value} to {$newStatus->value}"
            );
        }
        $this->status = $newStatus;
    }
}

// Usage
$order = new Order();

// Access cases
echo OrderStatus::Pending->value;   // pending
echo OrderStatus::Pending->label(); // ⏳ Pending
echo OrderStatus::Pending->name;    // Pending (the case name as string)

// From a value (backed enum)
$status = OrderStatus::from("shipped");  // OrderStatus::Shipped
echo $status->label();  // 🚚 Shipped

$status = OrderStatus::tryFrom("unknown");  // null — safe, no exception
var_dump($status);  // NULL

// All cases
print_r(OrderStatus::cases());
// Array of all OrderStatus cases
?>
```

---

## Design Patterns

Design patterns are **proven, reusable solutions** to commonly occurring software design problems. They are not code — they are templates you apply to your specific situation.

---

### Singleton Pattern

- Ensures a class has **only one instance** throughout the application.
- Provides a global access point to that instance.
- **Use cases:** Database connections, application config, loggers, caches.

```php
<?php
class AppConfig {
    private static ?self $instance = null;
    private array $settings = [];

    // Private constructor — no one can create an instance with `new`
    private function __construct() {
        // Load config from file/env
        $this->settings = [
            "debug"    => getenv("APP_DEBUG") === "true",
            "version"  => "1.0.0",
            "timezone" => "Asia/Yangon",
        ];
    }

    // Prevent cloning
    private function __clone() {}

    // The only way to get the instance
    public static function getInstance(): static {
        if (static::$instance === null) {
            static::$instance = new static();
        }
        return static::$instance;
    }

    public function get(string $key, mixed $default = null): mixed {
        return $this->settings[$key] ?? $default;
    }

    public function set(string $key, mixed $value): void {
        $this->settings[$key] = $value;
    }
}

$config1 = AppConfig::getInstance();
$config2 = AppConfig::getInstance();

var_dump($config1 === $config2);  // bool(true) — same object!

$config1->set("debug", true);
echo $config2->get("debug");  // true — both point to the same object

echo AppConfig::getInstance()->get("timezone");  // Asia/Yangon
?>
```

---

### Factory & Factory Method Pattern

- **Factory** — a class whose job is to create objects, deciding which class to instantiate based on input.
- Decouples the code that uses objects from the code that creates them.

```php
<?php
// Abstract product
interface Logger {
    public function log(string $level, string $message): void;
}

// Concrete products
class FileLogger implements Logger {
    public function __construct(private string $path) {}
    public function log(string $level, string $message): void {
        file_put_contents($this->path, "[$level] $message\n", FILE_APPEND);
    }
}

class DatabaseLogger implements Logger {
    public function log(string $level, string $message): void {
        // INSERT INTO logs (level, message, created_at) VALUES (...)
        echo "DB LOG: [$level] $message\n";
    }
}

class NullLogger implements Logger {
    public function log(string $level, string $message): void {
        // Do nothing — useful for testing or disabled logging
    }
}

// FACTORY — decides which Logger to create
class LoggerFactory {
    public static function create(string $type, array $options = []): Logger {
        return match ($type) {
            "file"     => new FileLogger($options["path"] ?? "/tmp/app.log"),
            "database" => new DatabaseLogger(),
            "null"     => new NullLogger(),
            default    => throw new \InvalidArgumentException("Unknown logger type: $type"),
        };
    }
}

// Usage — caller doesn't know or care which Logger class is created
$logger = LoggerFactory::create("file", ["path" => "/var/log/app.log"]);
$logger->log("INFO",  "User logged in");
$logger->log("ERROR", "Database connection failed");

$testLogger = LoggerFactory::create("null");
$testLogger->log("DEBUG", "This goes nowhere — silent in tests");

// Swap implementations with one config change — no other code changes!
$env    = getenv("LOG_DRIVER") ?: "file";
$logger = LoggerFactory::create($env);
?>
```

---

### Builder Pattern

- **Separates the construction of a complex object** from its representation.
- Allows building complex objects **step by step**, only specifying what you need.
- Already seen in the `QueryBuilder` earlier — here's a fuller example.

```php
<?php
class Email {
    // Private constructor — must use the Builder
    private function __construct(
        public readonly string  $from,
        public readonly string  $to,
        public readonly string  $subject,
        public readonly string  $body,
        public readonly array   $cc      = [],
        public readonly array   $bcc     = [],
        public readonly array   $attachments = [],
        public readonly bool    $isHtml  = false,
    ) {}

    public static function builder(): EmailBuilder {
        return new EmailBuilder();
    }
}

class EmailBuilder {
    private string $from    = "";
    private string $to      = "";
    private string $subject = "";
    private string $body    = "";
    private array  $cc      = [];
    private array  $bcc     = [];
    private array  $attachments = [];
    private bool   $isHtml  = false;

    public function from(string $from): static {
        $this->from = $from;
        return $this;
    }

    public function to(string $to): static {
        $this->to = $to;
        return $this;
    }

    public function subject(string $subject): static {
        $this->subject = $subject;
        return $this;
    }

    public function body(string $body, bool $html = false): static {
        $this->body   = $body;
        $this->isHtml = $html;
        return $this;
    }

    public function cc(string ...$emails): static {
        $this->cc = array_merge($this->cc, $emails);
        return $this;
    }

    public function attach(string $path): static {
        $this->attachments[] = $path;
        return $this;
    }

    public function build(): Email {
        // Validate required fields
        if (empty($this->from) || empty($this->to) || empty($this->subject)) {
            throw new \InvalidArgumentException("from, to, and subject are required");
        }

        return new Email(
            from:        $this->from,
            to:          $this->to,
            subject:     $this->subject,
            body:        $this->body,
            cc:          $this->cc,
            attachments: $this->attachments,
            isHtml:      $this->isHtml,
        );
    }
}

// Clean, readable construction — only set what you need
$email = Email::builder()
    ->from("noreply@myapp.com")
    ->to("phyo@example.com")
    ->subject("Your order has shipped!")
    ->body("<h1>Your order #1234 is on its way.</h1>", html: true)
    ->cc("manager@myapp.com")
    ->attach("/tmp/invoice-1234.pdf")
    ->build();

echo $email->subject;         // Your order has shipped!
echo count($email->attachments); // 1
?>
```

---

### Repository Pattern

- **Abstracts data access** — your business logic doesn't know (or care) if data comes from MySQL, a file, an API, or a cache.
- Makes switching storage backends easy and makes testing possible without a real database.

```php
<?php
// The data model
class User {
    public function __construct(
        public readonly int    $id,
        public string          $name,
        public string          $email,
    ) {}
}

// The contract — interface that all user repositories must follow
interface UserRepositoryInterface {
    public function findById(int $id): ?User;
    public function findByEmail(string $email): ?User;
    public function findAll(): array;
    public function save(User $user): void;
    public function delete(int $id): void;
}

// MySQL implementation
class MySQLUserRepository implements UserRepositoryInterface {
    public function __construct(private \PDO $pdo) {}

    public function findById(int $id): ?User {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE id = ?");
        $stmt->execute([$id]);
        $row = $stmt->fetch(\PDO::FETCH_ASSOC);
        return $row ? new User($row["id"], $row["name"], $row["email"]) : null;
    }

    public function findByEmail(string $email): ?User {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE email = ?");
        $stmt->execute([$email]);
        $row = $stmt->fetch(\PDO::FETCH_ASSOC);
        return $row ? new User($row["id"], $row["name"], $row["email"]) : null;
    }

    public function findAll(): array {
        $rows  = $this->pdo->query("SELECT * FROM users")->fetchAll(\PDO::FETCH_ASSOC);
        return array_map(fn($r) => new User($r["id"], $r["name"], $r["email"]), $rows);
    }

    public function save(User $user): void {
        $stmt = $this->pdo->prepare("INSERT INTO users (name, email) VALUES (?, ?)
                                     ON DUPLICATE KEY UPDATE name = ?, email = ?");
        $stmt->execute([$user->name, $user->email, $user->name, $user->email]);
    }

    public function delete(int $id): void {
        $this->pdo->prepare("DELETE FROM users WHERE id = ?")->execute([$id]);
    }
}

// In-memory implementation — perfect for testing, no real DB needed
class InMemoryUserRepository implements UserRepositoryInterface {
    private array $users    = [];
    private int   $nextId   = 1;

    public function findById(int $id): ?User {
        return $this->users[$id] ?? null;
    }

    public function findByEmail(string $email): ?User {
        foreach ($this->users as $user) {
            if ($user->email === $email) return $user;
        }
        return null;
    }

    public function findAll(): array { return array_values($this->users); }

    public function save(User $user): void {
        $this->users[$user->id] = $user;
    }

    public function delete(int $id): void {
        unset($this->users[$id]);
    }
}

// Business logic — depends on the INTERFACE, not the implementation
class UserService {
    public function __construct(private UserRepositoryInterface $users) {}

    public function register(string $name, string $email): User {
        if ($this->users->findByEmail($email)) {
            throw new \DomainException("Email already registered");
        }
        $user = new User(0, $name, $email);
        $this->users->save($user);
        return $user;
    }

    public function getUser(int $id): User {
        return $this->users->findById($id)
            ?? throw new \RuntimeException("User #$id not found");
    }
}

// Swap implementations without changing UserService!
$memRepo  = new InMemoryUserRepository();
$service  = new UserService($memRepo);

// Or with real MySQL:
// $pdo     = new PDO("mysql:host=localhost;dbname=myapp", "user", "pass");
// $service = new UserService(new MySQLUserRepository($pdo));
?>
```

---

### Observer Pattern

- Defines a **one-to-many dependency** — when one object (subject/publisher) changes state, all its observers (subscribers) are notified automatically.
- **Use cases:** Event systems, UI updates, logging, cache invalidation.

```php
<?php
// The Observer interface — anything that wants to be notified
interface Observer {
    public function update(string $event, mixed $data): void;
}

// The Subject interface — anything that can be observed
interface Subject {
    public function subscribe(string $event, Observer $observer): void;
    public function unsubscribe(string $event, Observer $observer): void;
    public function notify(string $event, mixed $data = null): void;
}

// EventEmitter trait — reusable Observer logic for any class
trait EventEmitter {
    private array $observers = [];

    public function subscribe(string $event, Observer $observer): void {
        $this->observers[$event][] = $observer;
    }

    public function unsubscribe(string $event, Observer $observer): void {
        $this->observers[$event] = array_filter(
            $this->observers[$event] ?? [],
            fn($obs) => $obs !== $observer
        );
    }

    public function notify(string $event, mixed $data = null): void {
        foreach ($this->observers[$event] ?? [] as $observer) {
            $observer->update($event, $data);
        }
    }
}

// Concrete subject
class UserRegistration {
    use EventEmitter;

    public function register(string $name, string $email): array {
        // Do the actual registration
        $user = ["id" => rand(1, 1000), "name" => $name, "email" => $email];

        // Notify all subscribers
        $this->notify("user.registered", $user);

        return $user;
    }
}

// Concrete observers
class WelcomeEmailObserver implements Observer {
    public function update(string $event, mixed $data): void {
        if ($event === "user.registered") {
            echo "📧 Sending welcome email to {$data["email"]}\n";
        }
    }
}

class AuditLogObserver implements Observer {
    public function update(string $event, mixed $data): void {
        echo "📝 Audit: $event — User #{$data["id"]} created\n";
    }
}

class SlackNotificationObserver implements Observer {
    public function update(string $event, mixed $data): void {
        if ($event === "user.registered") {
            echo "💬 Slack: New user registered: {$data["name"]}\n";
        }
    }
}

// Wire it up
$registration = new UserRegistration();
$registration->subscribe("user.registered", new WelcomeEmailObserver());
$registration->subscribe("user.registered", new AuditLogObserver());
$registration->subscribe("user.registered", new SlackNotificationObserver());

$registration->register("Phyo", "phyo@example.com");
// 📧 Sending welcome email to phyo@example.com
// 📝 Audit: user.registered — User #847 created
// 💬 Slack: New user registered: Phyo
?>
```

---

### Strategy Pattern

- Defines a **family of algorithms**, encapsulates each one, and makes them interchangeable.
- Lets you change the algorithm (strategy) a class uses at runtime without changing the class itself.
- **Use cases:** Sorting algorithms, payment methods, shipping calculators, discount strategies.

```php
<?php
// Strategy interface
interface DiscountStrategy {
    public function apply(float $price): float;
    public function describe(): string;
}

// Concrete strategies
class NoDiscount implements DiscountStrategy {
    public function apply(float $price): float    { return $price; }
    public function describe(): string             { return "No discount"; }
}

class PercentageDiscount implements DiscountStrategy {
    public function __construct(private float $percent) {}
    public function apply(float $price): float    { return $price * (1 - $this->percent / 100); }
    public function describe(): string             { return "{$this->percent}% off"; }
}

class FixedDiscount implements DiscountStrategy {
    public function __construct(private float $amount) {}
    public function apply(float $price): float    { return max(0, $price - $this->amount); }
    public function describe(): string             { return "\${$this->amount} off"; }
}

class BuyOneGetOneFree implements DiscountStrategy {
    public function apply(float $price): float    { return $price / 2; }
    public function describe(): string             { return "Buy 1 get 1 free"; }
}

// Context class — uses a strategy
class ShoppingCart {
    private DiscountStrategy $discount;

    public function __construct(private float $subtotal) {
        $this->discount = new NoDiscount();  // Default: no discount
    }

    public function setDiscount(DiscountStrategy $strategy): void {
        $this->discount = $strategy;
    }

    public function getTotal(): float {
        return $this->discount->apply($this->subtotal);
    }

    public function describe(): string {
        return sprintf(
            "Subtotal: $%.2f | %s | Total: $%.2f",
            $this->subtotal,
            $this->discount->describe(),
            $this->getTotal()
        );
    }
}

$cart = new ShoppingCart(100.0);
echo $cart->describe();  // Subtotal: $100.00 | No discount | Total: $100.00

$cart->setDiscount(new PercentageDiscount(20));
echo $cart->describe();  // Subtotal: $100.00 | 20% off | Total: $80.00

$cart->setDiscount(new FixedDiscount(15));
echo $cart->describe();  // Subtotal: $100.00 | $15 off | Total: $85.00

$cart->setDiscount(new BuyOneGetOneFree());
echo $cart->describe();  // Subtotal: $100.00 | Buy 1 get 1 free | Total: $50.00
?>
```

---

### Dependency Injection Pattern

- **Dependency Injection (DI)** means providing a class's dependencies (the objects it needs) from outside, rather than creating them internally.
- Makes classes loosely coupled, reusable, and testable.
- This is the most important pattern for building maintainable PHP applications.

```php
<?php
// ❌ BAD — tight coupling, hard to test, hard to swap implementations
class OrderService {
    private MySQLUserRepository $userRepo;
    private FileLogger          $logger;
    private StripePayment       $payment;

    public function __construct() {
        $this->userRepo = new MySQLUserRepository();  // Hardcoded!
        $this->logger   = new FileLogger("/var/log/orders.log");  // Hardcoded!
        $this->payment  = new StripePayment(getenv("STRIPE_KEY")); // Hardcoded!
    }
}

// ✅ GOOD — Dependency Injection, loose coupling
class OrderService {
    public function __construct(
        private UserRepositoryInterface $userRepo,  // Depends on abstraction
        private Logger                  $logger,    // Depends on abstraction
        private PaymentProcessor        $payment,   // Depends on abstraction
    ) {}

    public function placeOrder(int $userId, array $items): string {
        $user  = $this->userRepo->findById($userId);
        if (!$user) throw new \RuntimeException("User not found");

        $total = array_sum(array_column($items, "price"));

        $this->logger->log("INFO", "Processing order for {$user->name}");

        $orderId = $this->payment->charge($total, $user->email);

        $this->logger->log("INFO", "Order $orderId completed — \${$total}");
        return $orderId;
    }
}

// Production — real dependencies
$orderService = new OrderService(
    new MySQLUserRepository($pdo),
    new FileLogger("/var/log/orders.log"),
    new StripePayment(getenv("STRIPE_KEY")),
);

// Testing — fake/mock dependencies (no real DB, no real emails, no real charges!)
$orderService = new OrderService(
    new InMemoryUserRepository(),
    new NullLogger(),
    new FakePaymentProcessor(),
);

// DI Container (how frameworks like Laravel do it automatically)
class Container {
    private array $bindings = [];

    public function bind(string $abstract, callable $factory): void {
        $this->bindings[$abstract] = $factory;
    }

    public function make(string $abstract): mixed {
        $factory = $this->bindings[$abstract]
            ?? throw new \RuntimeException("No binding for $abstract");
        return $factory($this);
    }
}

$container = new Container();

$container->bind(UserRepositoryInterface::class, fn($c) => new MySQLUserRepository($pdo));
$container->bind(Logger::class,                  fn($c) => new FileLogger("/var/log/app.log"));
$container->bind(OrderService::class, fn($c) => new OrderService(
    $c->make(UserRepositoryInterface::class),
    $c->make(Logger::class),
    new StripePayment(getenv("STRIPE_KEY")),
));

$service = $container->make(OrderService::class);
?>
```

---

## Quick Revision

- **`__toString()`** — called when object is used as a string. Must return a string.
- **`__get()`/`__set()`/`__isset()`/`__unset()`** — intercept property access for non-existent/inaccessible properties. Used for dynamic objects and proxy patterns.
- **`__call()`/`__callStatic()`** — intercept calls to undefined methods. Great for dynamic API clients and method routing.
- **`__clone()`** — called after `clone`. Override for deep cloning of nested objects (shallow clone is default).
- **`__invoke()`** — called when object is used as `$obj()`. Makes objects callable — works with `array_map`, `usort`, etc.
- **`__debugInfo()`** — controls what `var_dump()` shows. Use to hide sensitive data (passwords, tokens).
- **Enums** (PHP 8.1+) — represent fixed sets of values. Backed enums have values (`string` or `int`). Can have methods. Use `::from()` (throws) or `::tryFrom()` (returns null) to create from value.
- **Singleton** — one instance, global access. Private constructor + static `getInstance()`.
- **Factory** — centralizes object creation. Decouples creation logic from usage code.
- **Builder** — constructs complex objects step by step with fluent chaining. Prevents huge constructors with 10+ params.
- **Repository** — abstracts data access behind an interface. Swap MySQL for InMemory for testing with zero business logic changes.
- **Observer** — publish/subscribe. Subject notifies all registered observers when events happen. Decouples event producers from consumers.
- **Strategy** — encapsulates interchangeable algorithms. Swap behavior at runtime (different discounts, payment methods, etc.) without `if/else` chains.
- **Dependency Injection** — provide dependencies from outside rather than creating them inside. The most important pattern for maintainable, testable PHP. Depend on abstractions (interfaces), not concrete implementations.
- The **golden OOP principle** underlying all patterns: **depend on abstractions, not concretions** (Dependency Inversion Principle — the D in SOLID).