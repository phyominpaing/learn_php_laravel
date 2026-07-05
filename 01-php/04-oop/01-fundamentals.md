# PHP Modern OOP — Part 1: Fundamentals

**Object-Oriented Programming (OOP)** is a programming paradigm that organizes code into **objects** — bundles of related data (properties) and behavior (methods) — making large applications easier to build, maintain, and scale.

---

## Table of Contents

1. [Why OOP?](#why-oop)
2. [Classes & Objects](#classes--objects)
3. [Properties](#properties)
4. [Methods](#methods)
5. [Constructor & Destructor](#constructor--destructor)
6. [Constructor Property Promotion (PHP 8+)](#constructor-property-promotion-php-8)
7. [Access Modifiers](#access-modifiers)
8. [Static Properties & Methods](#static-properties--methods)
9. [Constants in Classes](#constants-in-classes)
10. [The `$this` Keyword](#the-this-keyword)
11. [The `self::` and `static::` Keywords](#self-and-static-keywords)
12. [Quick Revision](#quick-revision)

---

## Why OOP?

Before OOP, PHP was written as long procedural scripts — functions and variables scattered across files. As applications grew, code became impossible to maintain.

```php
<?php
// Procedural style — everything is loose, unorganized
$userName    = "Phyo";
$userEmail   = "phyo@example.com";
$userBalance = 100.0;

function getUserName($name) { return $name; }
function deposit($balance, $amount) { return $balance + $amount; }
function withdraw($balance, $amount) { return $balance - $amount; }
?>
```

```php
<?php
// OOP style — data and behavior bundled together in one place
class BankAccount {
    private string $owner;
    private float  $balance;

    public function __construct(string $owner, float $balance) {
        $this->owner   = $owner;
        $this->balance = $balance;
    }

    public function deposit(float $amount): void  { $this->balance += $amount; }
    public function withdraw(float $amount): void { $this->balance -= $amount; }
    public function getBalance(): float           { return $this->balance; }
}

$account = new BankAccount("Phyo", 100.0);
$account->deposit(50);
echo $account->getBalance();  // 150
?>
```

### Benefits of OOP

| Benefit | What it means |
|---|---|
| **Encapsulation** | Data and logic are bundled together; internal details are hidden |
| **Reusability** | Write a class once, create multiple objects from it |
| **Inheritance** | Child classes reuse and extend parent class behavior |
| **Polymorphism** | Different classes can be used through a common interface |
| **Maintainability** | Changes in one class don't break unrelated code |

---

## Classes & Objects

- A **class** is a **blueprint** or template — it defines what properties and methods something has.
- An **object** is an **instance** of a class — a specific realization of the blueprint with its own data.

```php
<?php
// CLASS — the blueprint
class Car {
    public string $make;
    public string $model;
    public int    $year;

    public function describe(): string {
        return "{$this->year} {$this->make} {$this->model}";
    }
}

// OBJECTS — instances of the class (each has its own data)
$car1 = new Car();
$car1->make  = "Toyota";
$car1->model = "Corolla";
$car1->year  = 2022;

$car2 = new Car();
$car2->make  = "Honda";
$car2->model = "Civic";
$car2->year  = 2023;

echo $car1->describe();  // Output: 2022 Toyota Corolla
echo $car2->describe();  // Output: 2023 Honda Civic

// Each object is independent — changing one doesn't affect the other
$car1->year = 2024;
echo $car1->year;  // 2024
echo $car2->year;  // 2023 — unchanged
?>
```

> 💡 **Analogy:** A class is like an architectural blueprint for a house. The blueprint defines the rooms, walls, and features. Each actual house built from that blueprint is an object — they share the same design but have their own specific address, color, and contents.

---

## Properties

- **Properties** are variables that belong to a class — they hold the **state** (data) of an object.
- Declared inside the class body using visibility keywords (`public`, `protected`, `private`).
- Accessed on an object using the `->` arrow operator.

```php
<?php
class Product {
    // Typed properties (PHP 7.4+) — must be initialized before use
    public    string  $name;
    public    float   $price;
    protected int     $stock    = 0;      // Default value — initialized
    private   bool    $active   = true;
    public    ?string $category = null;   // Nullable — can hold null

    // Readonly property (PHP 8.1+) — can only be written ONCE
    public readonly int $id;
}

$p        = new Product();
$p->name  = "PHP Handbook";
$p->price = 29.99;

echo $p->name;   // PHP Handbook
echo $p->price;  // 29.99

// Accessing uninitialized typed property throws an Error
// echo $p->id;  // ❌ Error: must not be accessed before initialization
?>
```

### Readonly Properties (PHP 8.1+)

```php
<?php
class User {
    public readonly int    $id;
    public readonly string $email;

    public function __construct(int $id, string $email) {
        $this->id    = $id;      // ✅ First assignment — allowed
        $this->email = $email;   // ✅ First assignment — allowed
    }
}

$user = new User(1, "phyo@example.com");
echo $user->id;     // 1
echo $user->email;  // phyo@example.com

// $user->id = 2;   // ❌ Error: Cannot modify readonly property
?>
```

> 💡 **Readonly properties** are perfect for immutable data — things that should never change after creation, like a database record's ID, a user's email, or a transaction amount.

---

## Methods

- **Methods** are functions that belong to a class — they define the **behavior** of objects.
- Defined the same way as regular functions but inside the class body.
- Access the object's own properties and other methods using `$this`.

```php
<?php
class Temperature {
    private float $celsius;

    public function __construct(float $celsius) {
        $this->celsius = $celsius;
    }

    // Regular method
    public function getCelsius(): float {
        return $this->celsius;
    }

    // Method that computes and returns a derived value
    public function getFahrenheit(): float {
        return ($this->celsius * 9 / 5) + 32;
    }

    public function getKelvin(): float {
        return $this->celsius + 273.15;
    }

    // Method that modifies state
    public function setCelsius(float $celsius): void {
        $this->celsius = $celsius;
    }

    // Method that calls other methods
    public function describe(): string {
        return sprintf(
            "%.1f°C = %.1f°F = %.2fK",
            $this->getCelsius(),
            $this->getFahrenheit(),
            $this->getKelvin()
        );
    }
}

$temp = new Temperature(100);
echo $temp->getCelsius();     // 100
echo $temp->getFahrenheit();  // 212
echo $temp->getKelvin();      // 373.15
echo $temp->describe();       // 100.0°C = 212.0°F = 373.15K
?>
```

### Method Chaining (Fluent Interface)

```php
<?php
class QueryBuilder {
    private string $table  = "";
    private array  $wheres = [];
    private ?int   $limit  = null;

    public function from(string $table): static {
        $this->table = $table;
        return $this;  // Return the same object → enables chaining
    }

    public function where(string $condition): static {
        $this->wheres[] = $condition;
        return $this;
    }

    public function limit(int $n): static {
        $this->limit = $n;
        return $this;
    }

    public function build(): string {
        $sql = "SELECT * FROM {$this->table}";
        if (!empty($this->wheres)) {
            $sql .= " WHERE " . implode(" AND ", $this->wheres);
        }
        if ($this->limit !== null) {
            $sql .= " LIMIT {$this->limit}";
        }
        return $sql;
    }
}

$query = (new QueryBuilder())
    ->from("users")
    ->where("active = 1")
    ->where("age >= 18")
    ->limit(10)
    ->build();

echo $query;
// Output: SELECT * FROM users WHERE active = 1 AND age >= 18 LIMIT 10
?>
```

---

## Constructor & Destructor

### Constructor `__construct()`

- A **special method** called automatically when an object is created with `new`.
- Used to initialize properties and set up the object's initial state.
- Parameters defined in the constructor become arguments of `new ClassName(...)`.

```php
<?php
class Order {
    private string    $id;
    private array     $items    = [];
    private \DateTime $createdAt;

    public function __construct(string $id) {
        $this->id        = $id;
        $this->createdAt = new \DateTime();  // Set to now automatically

        echo "Order $id created at " . $this->createdAt->format("H:i:s") . "\n";
    }

    public function addItem(string $item, float $price): void {
        $this->items[] = ["item" => $item, "price" => $price];
    }

    public function getTotal(): float {
        return array_sum(array_column($this->items, "price"));
    }
}

$order = new Order("ORD-001");
// Output: Order ORD-001 created at 14:30:00

$order->addItem("PHP Book", 29.99);
$order->addItem("MySQL Guide", 24.99);
echo $order->getTotal();  // 54.98
?>
```

### Destructor `__destruct()`

- Called automatically when an object is **destroyed** — when the script ends, or when there are no more references to it.
- Used for cleanup — closing file handles, database connections, flushing buffers.

```php
<?php
class FileLogger {
    private       $handle;
    private string $path;

    public function __construct(string $path) {
        $this->path   = $path;
        $this->handle = fopen($path, "a");
        $this->log("Logger started");
    }

    public function log(string $message): void {
        fwrite($this->handle, date("[Y-m-d H:i:s] ") . $message . "\n");
    }

    public function __destruct() {
        $this->log("Logger stopped");
        fclose($this->handle);   // Always close the file
        echo "File handle closed\n";
    }
}

$logger = new FileLogger("/var/log/app.log");
$logger->log("Processing order #123");
$logger->log("Order complete");
// When $logger goes out of scope (script ends or unset($logger)):
// __destruct() runs → logs "Logger stopped" → closes file handle
?>
```

---

## Constructor Property Promotion (PHP 8+)

- A shorthand syntax that declares and assigns constructor parameters as class properties in one step.
- Eliminates the repetitive boilerplate of declaring properties then re-assigning them in the constructor.

```php
<?php
// ❌ The old verbose way (PHP 7 style)
class UserOld {
    public int    $id;
    public string $name;
    public string $email;
    private bool  $active;

    public function __construct(int $id, string $name, string $email, bool $active = true) {
        $this->id     = $id;
        $this->name   = $name;
        $this->email  = $email;
        $this->active = $active;
    }
}

// ✅ The modern way with constructor property promotion (PHP 8+)
class User {
    public function __construct(
        public readonly int    $id,
        public string          $name,
        public string          $email,
        private bool           $active = true,
    ) {} // Body is empty — PHP handles assignment automatically!
}

$user = new User(1, "Phyo", "phyo@example.com");
echo $user->id;     // 1
echo $user->name;   // Phyo
echo $user->email;  // phyo@example.com

// Can combine promoted and non-promoted in the same constructor
class Product {
    public float $tax;  // Non-promoted — declared separately

    public function __construct(
        public readonly int    $id,      // Promoted
        public string          $name,    // Promoted
        public float           $price,   // Promoted
        float $taxRate = 0.09,           // Regular parameter (not promoted)
    ) {
        $this->tax = $price * $taxRate;  // Used in body
    }
}

$product = new Product(1, "Shirt", 19.99);
echo $product->name;  // Shirt
echo $product->tax;   // 1.7991
?>
```

---

## Access Modifiers

- **Access modifiers** (also called **visibility keywords**) control where a property or method can be accessed from.
- PHP has three: `public`, `protected`, and `private`.

```php
<?php
class BankAccount {
    public    string $owner;      // Accessible from anywhere
    protected float  $balance;    // Accessible from this class AND subclasses
    private   string $pin;        // ONLY accessible from inside this class

    public function __construct(string $owner, float $balance, string $pin) {
        $this->owner   = $owner;
        $this->balance = $balance;
        $this->pin     = $pin;
    }

    public function getBalance(): float {
        return $this->balance;  // ✅ Can access private/protected from within
    }

    private function validatePin(string $input): bool {
        return $input === $this->pin;  // Private method — internal only
    }

    public function withdraw(float $amount, string $pin): bool {
        if (!$this->validatePin($pin)) {
            return false;
        }
        if ($amount > $this->balance) {
            return false;
        }
        $this->balance -= $amount;
        return true;
    }
}

$account = new BankAccount("Phyo", 500.0, "1234");

echo $account->owner;          // ✅ "Phyo" — public
echo $account->getBalance();   // ✅ 500 — public method accesses protected

// echo $account->balance;     // ❌ Fatal Error — protected
// echo $account->pin;         // ❌ Fatal Error — private
// $account->validatePin("1234"); // ❌ Fatal Error — private method
?>
```

### Access Modifier Comparison

| Modifier | Same Class | Subclass | Outside Class |
|---|---|---|---|
| `public` | ✅ Yes | ✅ Yes | ✅ Yes |
| `protected` | ✅ Yes | ✅ Yes | ❌ No |
| `private` | ✅ Yes | ❌ No | ❌ No |

> ⚠️ **Common Mistake:** Making everything `public` "to keep things simple." This breaks **encapsulation** — external code can modify internal state in unexpected ways, making bugs very hard to track. Start with `private`, loosen to `protected` or `public` only when needed.

---

## Static Properties & Methods

- **Static** properties and methods belong to the **class itself**, not to any specific object.
- Accessed using `ClassName::$property` or `ClassName::method()` — no object needed.
- Shared across ALL instances of the class.

```php
<?php
class Counter {
    private static int $count = 0;   // Shared across all instances

    public function __construct() {
        self::$count++;  // Increments the shared counter
    }

    public static function getCount(): int {
        return self::$count;
    }

    public static function reset(): void {
        self::$count = 0;
    }
}

$a = new Counter();
$b = new Counter();
$c = new Counter();

echo Counter::getCount();  // 3 — all three instances share the same counter
Counter::reset();
echo Counter::getCount();  // 0

// Static factory method pattern (very common)
class Database {
    private static ?Database $instance = null;
    private \PDO $pdo;

    private function __construct() {
        $this->pdo = new \PDO("mysql:host=localhost;dbname=myapp", "user", "pass");
    }

    public static function getInstance(): static {
        if (static::$instance === null) {
            static::$instance = new static();
        }
        return static::$instance;
    }

    public function getPdo(): \PDO {
        return $this->pdo;
    }
}

$db1 = Database::getInstance();
$db2 = Database::getInstance();
var_dump($db1 === $db2);  // bool(true) — same object!
?>
```

---

## Constants in Classes

- Class constants are defined with `const` inside a class.
- Belong to the class (not instances) — accessed with `ClassName::CONSTANT`.
- Cannot change after definition — immutable.
- Can have visibility: `public`, `protected`, `private` (PHP 7.1+).

```php
<?php
class HttpStatus {
    public const OK           = 200;
    public const CREATED      = 201;
    public const NO_CONTENT   = 204;
    public const BAD_REQUEST  = 400;
    public const UNAUTHORIZED = 401;
    public const NOT_FOUND    = 404;
    public const SERVER_ERROR = 500;

    public static function getMessage(int $code): string {
        return match($code) {
            self::OK           => "OK",
            self::CREATED      => "Created",
            self::NOT_FOUND    => "Not Found",
            self::SERVER_ERROR => "Internal Server Error",
            default            => "Unknown Status",
        };
    }
}

// Access without creating an object
echo HttpStatus::OK;                           // 200
echo HttpStatus::getMessage(404);              // Not Found

// Inside the class use self:: or static::
// Outside use ClassName::

// Constants in inheritance
class PaymentStatus extends HttpStatus {
    public const PENDING   = "pending";
    public const COMPLETED = "completed";
    public const FAILED    = "failed";
}

echo PaymentStatus::PENDING;  // pending
echo PaymentStatus::OK;       // 200 — inherited from HttpStatus
?>
```

---

## The `$this` Keyword

- `$this` refers to the **current object instance** — the specific object that the method was called on.
- Only available inside **non-static** methods.
- Used to access the object's own properties and call its own methods.

```php
<?php
class Rectangle {
    public function __construct(
        private float $width,
        private float $height,
    ) {}

    public function area(): float {
        return $this->width * $this->height;  // $this = this specific rectangle
    }

    public function perimeter(): float {
        return 2 * ($this->width + $this->height);
    }

    public function isSquare(): bool {
        return $this->width === $this->height;
    }

    public function scale(float $factor): static {
        $this->width  *= $factor;
        $this->height *= $factor;
        return $this;
    }

    public function describe(): string {
        return sprintf(
            "Rectangle %gx%g | Area: %g | Square: %s",
            $this->width,
            $this->height,
            $this->area(),
            $this->isSquare() ? "Yes" : "No"
        );
    }
}

$r1 = new Rectangle(5, 3);
echo $r1->describe();  // Rectangle 5x3 | Area: 15 | Square: No

$r2 = new Rectangle(4, 4);
echo $r2->describe();  // Rectangle 4x4 | Area: 16 | Square: Yes

// Each $this refers to its own object
$r1->scale(2);
echo $r1->describe();  // Rectangle 10x6 | Area: 60 | Square: No
echo $r2->describe();  // Rectangle 4x4 | Area: 16 | Square: Yes ← unchanged
?>
```

---

## `self::` and `static::` Keywords

- `self::` refers to the **class where the code is physically written** — resolved at compile time.
- `static::` refers to the **class that was actually called at runtime** — supports late static binding.
- The difference matters in **inheritance**.

```php
<?php
class ParentClass {
    protected static string $name = "Parent";

    public static function createSelf(): static {
        return new self();    // Always creates ParentClass, even from a child
    }

    public static function createStatic(): static {
        return new static();  // Creates whatever class was actually called
    }

    public static function getName(): string {
        return static::$name;  // Late static binding — reads child's $name
    }
}

class ChildClass extends ParentClass {
    protected static string $name = "Child";
}

$obj1 = ChildClass::createSelf();    // Returns ParentClass instance ← might surprise you
$obj2 = ChildClass::createStatic();  // Returns ChildClass instance  ← correct!

echo get_class($obj1);  // ParentClass
echo get_class($obj2);  // ChildClass

echo ParentClass::getName();  // Parent
echo ChildClass::getName();   // Child  ← static:: reads ChildClass::$name
?>
```

> 💡 **Tip:** In modern PHP, prefer `static::` over `self::` in most cases — it's more flexible and works correctly with inheritance. Use `self::` only when you specifically want to reference the exact defining class regardless of subclassing.

---

## Quick Revision

- A **class** is a blueprint; an **object** is an instance of that class created with `new`.
- **Properties** hold the object's data (state); **methods** define its behavior.
- The **constructor** (`__construct()`) runs automatically when an object is created; the **destructor** (`__destruct()`) runs when it's destroyed.
- **Constructor property promotion** (PHP 8+) — put visibility modifiers in constructor parameters to declare and initialize properties in one step: `public function __construct(public string $name) {}`.
- `public` — accessible everywhere. `protected` — accessible in the class and subclasses. `private` — accessible only inside the defining class.
- **Static** properties and methods belong to the class itself — shared across all instances — accessed with `ClassName::method()`.
- Class **constants** use `const NAME = value;` inside a class, accessed via `ClassName::NAME` or `self::NAME`.
- `$this` refers to the current object instance inside non-static methods.
- `self::` refers to the class where the code was written (compile-time). `static::` refers to the class actually called at runtime (late static binding — important in inheritance).
- **Encapsulation** — make properties `private` by default, expose them only through controlled public methods (getters/setters).
- **Method chaining** — return `$this` (or `static`) from methods to enable fluent builder-style APIs.
- **Readonly properties** (PHP 8.1+) — can be set only once (in the constructor), then are immutable.