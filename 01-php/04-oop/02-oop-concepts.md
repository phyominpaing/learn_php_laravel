# PHP Modern OOP — Part 2: Inheritance, Interfaces & Abstract Classes

**Inheritance, interfaces, and abstract classes** are the pillars of reusable, extensible PHP code — they define *how* classes relate to each other and enforce contracts that make large codebases predictable and consistent.

---

## Table of Contents

1. [Inheritance](#inheritance)
2. [Method Overriding](#method-overriding)
3. [The `parent::` Keyword](#the-parent-keyword)
4. [`final` — Preventing Inheritance & Overriding](#final--preventing-inheritance--overriding)
5. [Abstract Classes](#abstract-classes)
6. [Interfaces](#interfaces)
7. [Abstract Class vs Interface](#abstract-class-vs-interface)
8. [Traits](#traits)
9. [Multiple Trait Use & Conflict Resolution](#multiple-trait-use--conflict-resolution)
10. [Quick Revision](#quick-revision)

---

## Inheritance

- **Inheritance** lets a class (**child/subclass**) extend another class (**parent/superclass**), inheriting all its `public` and `protected` properties and methods.
- Use the `extends` keyword.
- A child class can add new properties/methods AND override inherited ones.
- PHP supports **single inheritance** only — a class can extend exactly one parent.

```php
<?php
// PARENT class
class Animal {
    public string $name;
    public string $sound;

    public function __construct(string $name, string $sound) {
        $this->name  = $name;
        $this->sound = $sound;
    }

    public function speak(): string {
        return "{$this->name} says: {$this->sound}";
    }

    public function eat(string $food): string {
        return "{$this->name} eats {$food}";
    }
}

// CHILD classes — inherit everything from Animal
class Dog extends Animal {
    public string $breed;

    public function __construct(string $name, string $breed) {
        parent::__construct($name, "Woof");  // Call parent constructor
        $this->breed = $breed;
    }

    public function fetch(): string {
        return "{$this->name} fetches the ball!";
    }
}

class Cat extends Animal {
    public function __construct(string $name) {
        parent::__construct($name, "Meow");
    }

    public function purr(): string {
        return "{$this->name} purrs...";
    }
}

$dog = new Dog("Rex", "Labrador");
echo $dog->speak();   // Rex says: Woof  (inherited from Animal)
echo $dog->eat("bone"); // Rex eats bone (inherited from Animal)
echo $dog->fetch();   // Rex fetches the ball! (Dog-specific)
echo $dog->breed;     // Labrador (Dog-specific property)

$cat = new Cat("Luna");
echo $cat->speak();   // Luna says: Meow
echo $cat->purr();    // Luna purrs...
?>
```

### Inheritance Chain

```php
<?php
class Vehicle {
    public function __construct(public string $brand) {}
    public function move(): string { return "{$this->brand} moves"; }
}

class Car extends Vehicle {
    public function __construct(string $brand, public int $doors) {
        parent::__construct($brand);
    }
    public function drive(): string { return "{$this->brand} drives on {$this->doors} wheels"; }
}

class ElectricCar extends Car {
    public function __construct(string $brand, int $doors, public int $batteryKwh) {
        parent::__construct($brand, $doors);
    }
    public function charge(): string { return "Charging {$this->batteryKwh}kWh battery"; }
}

$tesla = new ElectricCar("Tesla", 4, 100);
echo $tesla->move();    // Tesla moves       (from Vehicle)
echo $tesla->drive();   // Tesla drives on 4 wheels (from Car)
echo $tesla->charge();  // Charging 100kWh battery  (own method)
?>
```

---

## Method Overriding

- A child class can **override** (replace) an inherited method by defining a method with the **same name**.
- The child's version replaces the parent's for that specific object type.
- To still call the parent's version, use `parent::methodName()`.

```php
<?php
class Shape {
    public function area(): float {
        return 0.0;  // Base implementation — will be overridden
    }

    public function describe(): string {
        return "I am a shape with area: " . $this->area();
    }
}

class Circle extends Shape {
    public function __construct(private float $radius) {}

    // Override the parent's area() method
    public function area(): float {
        return M_PI * $this->radius ** 2;
    }
}

class Rectangle extends Shape {
    public function __construct(
        private float $width,
        private float $height
    ) {}

    public function area(): float {
        return $this->width * $this->height;
    }
}

$shapes = [new Circle(5), new Rectangle(4, 6), new Shape()];

foreach ($shapes as $shape) {
    echo $shape->describe() . "\n";
}
// Output:
// I am a shape with area: 78.539816339745
// I am a shape with area: 24
// I am a shape with area: 0
// Note: describe() calls $this->area() which calls the correct OVERRIDDEN version!
// This is POLYMORPHISM — same method call, different behavior per type
?>
```

---

## The `parent::` Keyword

- Used inside a child class to **explicitly call the parent class's** constructor or method.
- Essential when overriding a method but still wanting to run the parent's version too.

```php
<?php
class Logger {
    protected array $logs = [];

    public function log(string $message): void {
        $this->logs[] = $message;
        echo "BASE LOG: $message\n";
    }

    public function getLogs(): array {
        return $this->logs;
    }
}

class TimestampLogger extends Logger {
    // Override log() but also call parent's version
    public function log(string $message): void {
        $timestamped = "[" . date("H:i:s") . "] $message";

        parent::log($timestamped);  // Call parent's log() with the modified message

        echo "TIMESTAMP ADDED\n";
    }
}

class FileLogger extends TimestampLogger {
    public function __construct(private string $filepath) {}

    public function log(string $message): void {
        parent::log($message);  // Calls TimestampLogger::log()
        file_put_contents($this->filepath, $message . "\n", FILE_APPEND);
        echo "WRITTEN TO FILE\n";
    }
}

$logger = new FileLogger("/tmp/app.log");
$logger->log("User logged in");
// BASE LOG: [14:30:00] User logged in
// TIMESTAMP ADDED
// WRITTEN TO FILE
?>
```

---

## `final` — Preventing Inheritance & Overriding

- **`final` class** — cannot be extended (subclassed).
- **`final` method** — cannot be overridden in child classes.

```php
<?php
// Final CLASS — cannot be extended
final class Singleton {
    private static ?self $instance = null;

    private function __construct() {}

    public static function getInstance(): static {
        if (static::$instance === null) {
            static::$instance = new static();
        }
        return static::$instance;
    }
}

// class MySingleton extends Singleton {}  // ❌ Fatal Error: Cannot extend final class

// Final METHOD — cannot be overridden
class PaymentProcessor {
    final public function processPayment(float $amount): bool {
        // Critical payment logic that must never be changed by subclasses
        $this->validate($amount);
        $this->charge($amount);
        $this->log($amount);
        return true;
    }

    protected function validate(float $amount): void {}
    protected function charge(float $amount): void {}
    protected function log(float $amount): void {}
}

class StripeProcessor extends PaymentProcessor {
    // Can override non-final methods
    protected function charge(float $amount): void {
        echo "Charging $amount via Stripe";
    }

    // public function processPayment(float $amount): bool {}
    // ❌ Fatal Error: Cannot override final method
}
?>
```

---

## Abstract Classes

- An **abstract class** is a class that **cannot be instantiated directly** — it exists only to be extended.
- Can contain a mix of **abstract methods** (no body — must be implemented by child classes) and **concrete methods** (with a body).
- Declared with `abstract` keyword.

```php
<?php
abstract class Notification {
    // Concrete properties and methods — shared by all notifications
    protected string $recipient;
    protected string $message;

    public function __construct(string $recipient, string $message) {
        $this->recipient = $recipient;
        $this->message   = $message;
    }

    // ABSTRACT method — no body, MUST be implemented by every child class
    abstract public function send(): bool;

    // Concrete method — shared behavior, can be inherited or overridden
    public function format(): string {
        return "To: {$this->recipient}\nMessage: {$this->message}";
    }

    // Template method pattern — defines the algorithm, abstract step filled by child
    public function deliver(): void {
        echo "Preparing notification...\n";
        $formatted = $this->format();
        $success   = $this->send();       // ← This is abstract — called polymorphically
        echo $success ? "Sent!" : "Failed!\n";
    }
}

// Concrete child class — must implement send()
class EmailNotification extends Notification {
    public function __construct(
        string $recipient,
        string $message,
        private string $subject
    ) {
        parent::__construct($recipient, $message);
    }

    public function send(): bool {
        echo "Sending email to {$this->recipient}: {$this->subject}\n";
        // mail($this->recipient, $this->subject, $this->message);
        return true;
    }
}

class SMSNotification extends Notification {
    public function __construct(
        string $recipient,
        string $message,
        private string $phone
    ) {
        parent::__construct($recipient, $message);
    }

    public function send(): bool {
        echo "Sending SMS to {$this->phone}: {$this->message}\n";
        return true;
    }
}

class PushNotification extends Notification {
    public function send(): bool {
        echo "Sending push to {$this->recipient}: {$this->message}\n";
        return true;
    }
}

// Cannot instantiate abstract class directly
// $n = new Notification("x", "y");  // ❌ Fatal Error

$email = new EmailNotification("phyo@example.com", "Your order shipped!", "Order Update");
$email->deliver();
// Preparing notification...
// Sending email to phyo@example.com: Order Update
// Sent!

$sms = new SMSNotification("phyo@example.com", "Your OTP is 847291", "+959xxxxxxx");
$sms->deliver();
?>
```

---

## Interfaces

- An **interface** defines a **contract** — a list of method signatures that any implementing class MUST define.
- Interfaces contain **only** method signatures (no implementation) and constants — no properties.
- A class **implements** (not extends) an interface.
- A class can implement **multiple interfaces** — solving PHP's single-inheritance limitation.
- All interface methods are implicitly `public`.

```php
<?php
// Define contracts
interface Drawable {
    public function draw(): string;
    public function getColor(): string;
}

interface Resizable {
    public function resize(float $factor): static;
    public function getSize(): float;
}

interface Exportable {
    public function toArray(): array;
    public function toJSON(): string;
}

// A class can implement MULTIPLE interfaces
class Circle implements Drawable, Resizable, Exportable {
    public function __construct(
        private float  $radius,
        private string $color = "black"
    ) {}

    // Must implement ALL methods from Drawable
    public function draw(): string {
        return "Drawing a {$this->color} circle with radius {$this->radius}";
    }

    public function getColor(): string {
        return $this->color;
    }

    // Must implement ALL methods from Resizable
    public function resize(float $factor): static {
        $this->radius *= $factor;
        return $this;
    }

    public function getSize(): float {
        return $this->radius;
    }

    // Must implement ALL methods from Exportable
    public function toArray(): array {
        return ["type" => "circle", "radius" => $this->radius, "color" => $this->color];
    }

    public function toJSON(): string {
        return json_encode($this->toArray());
    }
}

$circle = new Circle(5, "red");
echo $circle->draw();      // Drawing a red circle with radius 5
echo $circle->toJSON();    // {"type":"circle","radius":5,"color":"red"}
$circle->resize(2);
echo $circle->getSize();   // 10

// Interface as a type hint — accepts ANY class implementing Drawable
function render(Drawable $shape): void {
    echo $shape->draw() . "\n";
}

render($circle);  // Works! Circle implements Drawable
?>
```

### Interface Extending Interface

```php
<?php
interface Readable {
    public function read(): string;
}

interface Writable {
    public function write(string $data): void;
}

// Interface can extend multiple interfaces
interface ReadWritable extends Readable, Writable {
    public function seek(int $position): void;
}

class FileStream implements ReadWritable {
    private int $position = 0;

    public function __construct(private string $content = "") {}

    public function read(): string {
        return substr($this->content, $this->position);
    }

    public function write(string $data): void {
        $this->content .= $data;
    }

    public function seek(int $position): void {
        $this->position = $position;
    }
}
?>
```

---

## Abstract Class vs Interface

| Feature | Abstract Class | Interface |
|---|---|---|
| Can have method implementations | ✅ Yes | ❌ No (only signatures) |
| Can have properties | ✅ Yes | ❌ No |
| Can have constructor | ✅ Yes | ❌ No |
| A class can extend/implement multiple | ❌ No (one parent only) | ✅ Yes (many interfaces) |
| Access modifiers on methods | Any (`public`, `protected`) | Always `public` |
| Constants allowed | ✅ Yes | ✅ Yes |
| Keyword | `extends` | `implements` |
| Purpose | Share partial implementation + contract | Pure contract only |

> 💡 **Decision guide:**
> - Use an **abstract class** when you want to share *some* common implementation between related classes (e.g., all Notifications share a `format()` method).
> - Use an **interface** when you want to define a *pure contract* that completely unrelated classes can all adhere to (e.g., `Loggable`, `Serializable`, `Drawable` — a Dog and a Circle can both be `Drawable`).

```php
<?php
// Both together — very common pattern
abstract class BaseRepository {
    abstract protected function findById(int $id): ?array;
    abstract protected function save(array $data): bool;

    public function exists(int $id): bool {
        return $this->findById($id) !== null;
    }
}

interface Cacheable {
    public function getCacheKey(): string;
    public function getTtl(): int;
}

class UserRepository extends BaseRepository implements Cacheable {
    public function findById(int $id): ?array {
        // Query database
        return null;
    }

    public function save(array $data): bool {
        // Save to database
        return true;
    }

    public function getCacheKey(): string { return "users"; }
    public function getTtl(): int         { return 3600; }
}
?>
```

---

## Traits

- A **trait** is a reusable collection of methods that can be "mixed into" any class.
- Solves the problem of sharing code between classes that don't share a common parent (PHP's single inheritance limitation).
- A class can `use` multiple traits.
- Think of traits as "copy-paste at the compiler level" — the trait's methods are injected directly into the class.

```php
<?php
// Define reusable traits
trait Timestamps {
    private ?DateTime $createdAt = null;
    private ?DateTime $updatedAt = null;

    public function setCreatedAt(): void {
        $this->createdAt = new DateTime();
    }

    public function setUpdatedAt(): void {
        $this->updatedAt = new DateTime();
    }

    public function getCreatedAt(): ?DateTime { return $this->createdAt; }
    public function getUpdatedAt(): ?DateTime { return $this->updatedAt; }
}

trait SoftDelete {
    private bool      $deleted   = false;
    private ?DateTime $deletedAt = null;

    public function softDelete(): void {
        $this->deleted   = true;
        $this->deletedAt = new DateTime();
    }

    public function restore(): void {
        $this->deleted   = false;
        $this->deletedAt = null;
    }

    public function isDeleted(): bool { return $this->deleted; }
}

trait Serializable2 {
    public function toJson(): string {
        return json_encode(get_object_vars($this));
    }

    public function toArray(): array {
        return get_object_vars($this);
    }
}

// Mix in traits — any class, regardless of hierarchy
class User {
    use Timestamps, SoftDelete, Serializable2;

    public function __construct(
        public readonly int    $id,
        public string          $name,
        public string          $email,
    ) {
        $this->setCreatedAt();
    }
}

class Post {
    use Timestamps, SoftDelete;  // No need to re-implement these

    public function __construct(
        public readonly int    $id,
        public string          $title,
        public string          $body,
    ) {
        $this->setCreatedAt();
    }
}

$user = new User(1, "Phyo", "phyo@example.com");
echo $user->getCreatedAt()->format("Y-m-d") . "\n";  // Today's date

$user->softDelete();
var_dump($user->isDeleted());  // bool(true)

$user->restore();
var_dump($user->isDeleted());  // bool(false)

echo $user->toJson();
// {"id":1,"name":"Phyo","email":"phyo@example.com","deleted":false,...}
?>
```

### Trait with Abstract Method

```php
<?php
trait Validatable {
    // Trait requires using class to implement this
    abstract protected function rules(): array;

    public function validate(array $data): bool {
        foreach ($this->rules() as $field => $rule) {
            if ($rule === "required" && empty($data[$field])) {
                echo "Validation failed: $field is required\n";
                return false;
            }
        }
        return true;
    }
}

class RegistrationForm {
    use Validatable;

    protected function rules(): array {
        return [
            "username" => "required",
            "email"    => "required",
            "password" => "required",
        ];
    }
}

$form = new RegistrationForm();
var_dump($form->validate(["username" => "Phyo", "email" => "", "password" => ""]));
// Validation failed: email is required → bool(false)
?>
```

---

## Multiple Trait Use & Conflict Resolution

```php
<?php
trait Hello {
    public function greet(): string {
        return "Hello!";
    }

    public function doSomething(): string {
        return "Hello trait does something";
    }
}

trait World {
    public function greet(): string {
        return "World!";
    }

    public function doSomething(): string {
        return "World trait does something";
    }
}

class MyClass {
    use Hello, World {
        // Resolve naming conflicts:
        Hello::greet       insteadof World;  // Use Hello's greet, ignore World's
        World::doSomething insteadof Hello;  // Use World's doSomething, ignore Hello's

        // Give an alias to the ignored method so it's still accessible
        World::greet       as greetWorld;    // World's greet accessible as greetWorld()
        Hello::doSomething as doHello;       // Hello's doSomething accessible as doHello()
    }
}

$obj = new MyClass();
echo $obj->greet();       // Hello!  (Hello wins)
echo $obj->greetWorld();  // World!  (alias for World's greet)
echo $obj->doSomething(); // World trait does something (World wins)
echo $obj->doHello();     // Hello trait does something (alias)
?>
```

---

## Quick Revision

- **Inheritance** (`extends`) — a child class inherits all `public` and `protected` members from its parent. PHP allows only one parent (single inheritance).
- **Method overriding** — child class redefines a parent method with the same name. `parent::method()` calls the parent's version.
- **`final` class** cannot be extended. **`final` method** cannot be overridden. Use to protect critical logic.
- **Abstract class** — cannot be instantiated; can mix abstract methods (no body) with concrete methods. Child classes MUST implement all abstract methods. Use `extends`.
- **Interface** — pure contract, methods have no body, all `public`. A class can `implements` multiple interfaces. No properties allowed.
- **Abstract class vs Interface:** abstract class = partial implementation + contract for related classes. Interface = pure contract for unrelated classes.
- **Trait** — `use` to inject reusable methods into any class regardless of hierarchy. Solves PHP's single-inheritance limitation. A class can use multiple traits.
- **Trait conflicts** — when two traits have the same method name, resolve with `TraitA::method insteadof TraitB` and optionally alias the losing one with `TraitB::method as alias`.
- **`parent::`** — calls the parent class's version of a method from inside a child class.
- **Polymorphism in action** — `foreach ($shapes as $shape) { $shape->area(); }` calls the correct `area()` for each type automatically because each class overrides it. This is the power of combining inheritance + method overriding.
- A class can both `extend` a parent AND `implement` multiple interfaces AND `use` multiple traits simultaneously: `class Child extends Parent implements A, B { use T1, T2; }`.