# PHP Constants

A **constant** is an identifier (name) for a simple value that **cannot be changed** during the execution of a script. Once defined, its value stays the same from that point forward — no matter what happens in your code. Think of it as a variable that gets locked the moment you create it.

---

## Table of Contents

1. [Constants vs Variables](#constants-vs-variables)
2. [Defining Constants with `define()`](#defining-constants-with-define)
3. [Defining Constants with `const`](#defining-constants-with-const)
4. [`define()` vs `const`](#define-vs-const)
5. [Constant Naming Conventions](#constant-naming-conventions)
6. [Constants are Global](#constants-are-global)
7. [Checking if a Constant is Defined](#checking-if-a-constant-is-defined)
8. [Constants in Arrays (PHP 7+)](#constants-in-arrays-php-7)
9. [Class Constants](#class-constants)
10. [Built-in PHP Constants](#built-in-php-constants)
11. [Magic Constants](#magic-constants)
12. [Quick Revision](#quick-revision)

---

## Constants vs Variables

Before diving in, here's a clear picture of how constants differ from variables:

| Feature | Variable | Constant |
|---|---|---|
| Syntax | `$name = "value"` | `define("NAME", "value")` |
| `$` prefix | ✅ Required | ❌ Not used |
| Can be changed | ✅ Yes | ❌ No — fixed forever |
| Scope | Local / Global | **Always global** |
| Type declared | ❌ Auto-detected | ❌ Auto-detected |
| Case sensitive | ✅ Yes | ✅ Yes (by default) |

```php
<?php
$siteName  = "My App";   // Variable — can change
define("SITE_NAME", "My App");  // Constant — locked

$siteName  = "New Name"; // ✅ Works fine
// SITE_NAME = "New Name"; // ❌ Fatal Error: Cannot reassign a constant
?>
```

> 💡 **Rule of thumb:** Use constants for values that should **never change** — app name, database host, API keys, file paths, version numbers, tax rates, etc.

---

## Defining Constants with `define()`

- `define()` is a **function** that creates a constant at **runtime**.
- Can be used anywhere in the script — inside functions, inside `if` blocks, anywhere.
- Syntax: `define("CONSTANT_NAME", value);`

```php
<?php
define("SITE_NAME",  "My PHP App");
define("MAX_LOGIN_ATTEMPTS", 5);
define("TAX_RATE",   0.07);
define("IS_DEBUG",   true);
define("COLORS",     ["red", "green", "blue"]);  // Array constant (PHP 7+)

echo SITE_NAME;           // Output: My PHP App
echo MAX_LOGIN_ATTEMPTS;  // Output: 5
echo TAX_RATE;            // Output: 0.07
echo IS_DEBUG;            // Output: 1  (true prints as 1)
echo COLORS[0];           // Output: red
?>
```

---

### `define()` Inside Conditionals & Functions

- This is one of the key strengths of `define()` — it works in dynamic contexts.

```php
<?php
$env = "production";

// Defined conditionally at runtime
if ($env === "production") {
    define("DB_HOST", "prod.db.example.com");
} else {
    define("DB_HOST", "localhost");
}

echo DB_HOST;  // Output: prod.db.example.com


// Defined inside a function
function setupApp() {
    define("APP_VERSION", "2.1.0");
}

setupApp();
echo APP_VERSION;  // Output: 2.1.0
?>
```

> ⚠️ **Warning:** If you try to define the same constant twice, PHP will throw a notice/warning and ignore the second definition.

```php
<?php
define("COLOR", "red");
define("COLOR", "blue");  // ⚠️ Notice: Constant COLOR already defined

echo COLOR;  // Output: red  (first definition wins)
?>
```

---

## Defining Constants with `const`

- `const` is a **language keyword** that creates a constant at **compile time**.
- Must be used at the **top level** of a script or inside a **class/interface**.
- Cannot be used inside functions, loops, or `if` statements.
- Slightly faster than `define()` because it is resolved at compile time.

```php
<?php
const APP_NAME    = "My PHP App";
const DB_PORT     = 3306;
const GRAVITY     = 9.81;
const MAINTENANCE = false;

echo APP_NAME;  // Output: My PHP App
echo DB_PORT;   // Output: 3306
echo GRAVITY;   // Output: 9.81
?>
```

---

### `const` Cannot Be Used in Dynamic Contexts

```php
<?php
// ❌ const inside a function — Fatal Error
function setup() {
    const VERSION = "1.0"; // Fatal error: const is not allowed here
}

// ❌ const inside an if block — Fatal Error
if (true) {
    const MODE = "debug"; // Fatal error
}

// ✅ Use define() for dynamic contexts instead
function setup() {
    define("VERSION", "1.0");  // ✅ This is fine
}
?>
```

---

## `define()` vs `const`

| Feature | `define()` | `const` |
|---|---|---|
| Type | Function | Language keyword |
| Resolved at | Runtime | Compile time |
| Inside functions | ✅ Yes | ❌ No |
| Inside conditionals | ✅ Yes | ❌ No |
| Inside classes | ❌ No | ✅ Yes |
| Dynamic constant name | ✅ Yes | ❌ No |
| Array values (PHP 7+) | ✅ Yes | ✅ Yes |
| Performance | Slightly slower | Slightly faster |

```php
<?php
// ✅ define() with a dynamic constant name
$prefix = "APP_";
define($prefix . "NAME", "MyApp");  // Creates APP_NAME
echo APP_NAME;  // Output: MyApp

// ❌ const cannot use dynamic names
// const $prefix . "NAME" = "MyApp";  // Syntax error
?>
```

> 💡 **When in doubt:** Use `const` at the top of your file or inside classes. Use `define()` when you need the constant to be defined conditionally or dynamically.

---

## Constant Naming Conventions

- Constants are **case-sensitive** by default.
- By strong convention, always write constant names in **ALL_CAPS with underscores**.
- This makes them instantly recognizable and distinguishable from variables.

```php
<?php
// ✅ Correct — ALL_CAPS convention
define("MAX_FILE_SIZE",  5242880);  // 5MB in bytes
define("API_BASE_URL",   "https://api.example.com");
define("DEFAULT_LANG",   "en");

// ❌ Avoid — hard to distinguish from variables
define("maxFileSize",    5242880);  // Looks like a variable
define("apiBaseUrl",     "https://api.example.com");

// Accessing
echo MAX_FILE_SIZE;   // 5242880
echo API_BASE_URL;    // https://api.example.com
?>
```

---

## Constants are Global

- Unlike variables, constants are **automatically global**.
- You can access a constant from **anywhere** in your script — inside functions, classes, or included files — without using the `global` keyword.

```php
<?php
define("TAX_RATE", 0.09);

function calculateTax($price) {
    // No 'global' keyword needed — constants are global by default
    return $price * TAX_RATE;
}

class Invoice {
    public function getTotal($price) {
        return $price + ($price * TAX_RATE);  // Also works inside a class
    }
}

echo calculateTax(100);  // Output: 9

$invoice = new Invoice();
echo $invoice->getTotal(100);  // Output: 109
?>
```

---

## Checking if a Constant is Defined

- Use `defined()` to check whether a constant exists before using it.
- Returns `true` if the constant is defined, `false` if not.
- Useful for preventing "constant already defined" notices.

```php
<?php
define("APP_NAME", "MyApp");

// Check before using
if (defined("APP_NAME")) {
    echo APP_NAME;  // Output: MyApp
}

// Check before defining (safe pattern)
if (!defined("DB_HOST")) {
    define("DB_HOST", "localhost");  // Only define if not already set
}

// Get value of a constant by name (string) using constant()
$name = "APP_NAME";
echo constant($name);  // Output: MyApp  (useful when name is dynamic)
?>
```

---

## Constants in Arrays (PHP 7+)

- Starting from PHP 7.0, constants can hold **array values**.
- Works with both `define()` and `const`.

```php
<?php
// Array constant with define()
define("WEEKDAYS", ["Mon", "Tue", "Wed", "Thu", "Fri"]);

echo WEEKDAYS[0];  // Output: Mon
echo WEEKDAYS[4];  // Output: Fri

// Array constant with const
const ALLOWED_ROLES = ["admin", "editor", "viewer"];

echo ALLOWED_ROLES[1];  // Output: editor

// Looping over a constant array
foreach (WEEKDAYS as $day) {
    echo $day . "\n";
}
// Output:
// Mon
// Tue
// Wed
// Thu
// Fri
?>
```

> ⚠️ **Note:** Before PHP 7.0, `define()` only supported scalar values (string, int, float, bool). Always use PHP 7+ for array constants.

---

## Class Constants

- Constants defined **inside a class** using `const`.
- Accessed using `ClassName::CONSTANT_NAME` (double colon — called the **scope resolution operator**).
- Class constants are shared across all instances of the class — they belong to the class itself, not to any object.
- Can be `public`, `protected`, or `private` (visibility added in PHP 7.1).

```php
<?php
class MathHelper {
    const PI            = 3.14159265358979;
    const EULER         = 2.71828182845905;
    public    const GRAVITY  = 9.81;
    protected const SECRET   = "hidden";
    private   const INTERNAL = "private";

    public function circleArea($radius) {
        return self::PI * $radius * $radius;  // Use self:: inside the class
    }
}

// Accessing from outside the class
echo MathHelper::PI;       // Output: 3.14159265358979
echo MathHelper::GRAVITY;  // Output: 9.81
// echo MathHelper::SECRET;   // ❌ Fatal Error — protected
// echo MathHelper::INTERNAL; // ❌ Fatal Error — private

// Accessing from inside the class
$math = new MathHelper();
echo $math->circleArea(5);  // Output: 78.539816...
?>
```

---

### Class Constants in Inheritance

- Child classes **inherit** parent class constants.
- Use `parent::CONSTANT` to access a parent constant from a child class.
- Child classes can **override** parent constants.

```php
<?php
class Animal {
    const TYPE = "Animal";

    public function describe() {
        return "I am a " . static::TYPE;  // static:: for late static binding
    }
}

class Dog extends Animal {
    const TYPE = "Dog";  // Overrides parent constant
}

$animal = new Animal();
$dog    = new Dog();

echo $animal->describe();  // Output: I am a Animal
echo $dog->describe();     // Output: I am a Dog

// Directly accessing parent constant
echo Animal::TYPE;  // Output: Animal
echo Dog::TYPE;     // Output: Dog
?>
```

---

### Interface Constants

- Interfaces can also define constants that implementing classes must not override.

```php
<?php
interface Colorable {
    const DEFAULT_COLOR = "white";
}

class Wall implements Colorable {
    public function getColor() {
        return self::DEFAULT_COLOR;  // Access via self::
    }
}

$wall = new Wall();
echo $wall->getColor();             // Output: white
echo Colorable::DEFAULT_COLOR;      // Output: white
echo Wall::DEFAULT_COLOR;           // Output: white
?>
```

---

## Built-in PHP Constants

- PHP comes with many predefined constants you can use anywhere.

```php
<?php
// PHP version info
echo PHP_VERSION;         // e.g. 8.3.2
echo PHP_MAJOR_VERSION;   // e.g. 8
echo PHP_MINOR_VERSION;   // e.g. 3
echo PHP_VERSION_ID;      // e.g. 80302 (useful for version comparisons)

// Integer limits
echo PHP_INT_MAX;         // 9223372036854775807
echo PHP_INT_MIN;         // -9223372036854775808
echo PHP_INT_SIZE;        // 8 (bytes)

// Float limits
echo PHP_FLOAT_MAX;       // 1.7976931348623E+308
echo PHP_FLOAT_MIN;       // 2.2250738585072E-308
echo PHP_FLOAT_EPSILON;   // 2.2204460492503E-16

// System info
echo PHP_OS;              // e.g. Linux, WINNT, Darwin
echo PHP_OS_FAMILY;       // e.g. Linux, Windows, Darwin (PHP 7.2+)
echo PHP_SAPI;            // e.g. cli, apache2handler, fpm-fcgi
echo PHP_MAXPATHLEN;      // Maximum length of a file path

// Special values
echo PHP_EOL;             // Line ending for current OS (\n on Linux, \r\n on Windows)
echo PHP_BINARY;          // Path to current PHP executable
echo DIRECTORY_SEPARATOR; // / on Linux, \ on Windows
echo PATH_SEPARATOR;      // : on Linux, ; on Windows

// Math
echo M_PI;                // 3.1415926535898
echo M_E;                 // 2.718281828459
echo M_SQRT2;             // 1.4142135623731 (square root of 2)

// Boolean & null (always available)
var_dump(TRUE);   // bool(true)
var_dump(FALSE);  // bool(false)
var_dump(NULL);   // NULL
?>
```

---

## Magic Constants

- **Magic constants** are special built-in constants whose value **changes depending on where they are used** in your code.
- They start and end with **double underscores** `__`.
- Always available — no need to define them.

```php
<?php
echo __LINE__;      // Current line number in this file
echo __FILE__;      // Full path to the current file: /var/www/html/index.php
echo __DIR__;       // Directory of the current file: /var/www/html
echo __FUNCTION__;  // Name of the current function ("" if outside a function)
echo __CLASS__;     // Name of the current class ("" if outside a class)
echo __TRAIT__;     // Name of the current trait ("" if outside a trait)
echo __METHOD__;    // Name of the current class + method: "ClassName::methodName"
echo __NAMESPACE__; // Current namespace ("" if no namespace)
?>
```

---

### Magic Constants in Practice

```php
<?php
// __FILE__ and __DIR__ for reliable file paths
$configPath = __DIR__ . "/config/database.php";
require_once $configPath;  // Always finds the right path, regardless of where you include this file from

// __LINE__ for debugging
echo "Error on line: " . __LINE__;

// __FUNCTION__ for debugging inside functions
function processPayment() {
    echo "Running: " . __FUNCTION__;  // Output: Running: processPayment
}

// __CLASS__ and __METHOD__ inside classes
class Logger {
    public function log($message) {
        echo "[" . __CLASS__ . "::" . __FUNCTION__ . "] " . $message;
        // Output: [Logger::log] your message here
    }
}

// __METHOD__ (includes class name, unlike __FUNCTION__)
class Auth {
    public function login() {
        echo __METHOD__;    // Output: Auth::login
        echo __FUNCTION__;  // Output: login  (just the method name)
    }
}
?>
```

---

### All Magic Constants at a Glance

| Magic Constant | Returns |
|---|---|
| `__LINE__` | Current line number |
| `__FILE__` | Full path + filename of current file |
| `__DIR__` | Directory of current file (no trailing slash) |
| `__FUNCTION__` | Current function name |
| `__CLASS__` | Current class name (including namespace) |
| `__TRAIT__` | Current trait name |
| `__METHOD__` | Current class + method: `Class::method` |
| `__NAMESPACE__` | Current namespace |

> 💡 **Most useful:** `__DIR__` is incredibly handy for building file paths that work regardless of where a script is called from. Use it instead of hardcoded paths.

---

## Quick Revision

- A **constant** is a named value that **cannot change** after it is defined. No `$` prefix.
- Define with `define("NAME", value)` or `const NAME = value;`. Both result in the same thing.
- `define()` works **anywhere** — functions, conditionals, dynamically. Use it for runtime constants.
- `const` works only at the **top level** or inside **classes**. Faster; resolved at compile time.
- By convention, constant names are **ALL_CAPS_WITH_UNDERSCORES**.
- Constants are **automatically global** — no `global` keyword needed anywhere.
- Use `defined("NAME")` to check if a constant exists before using or redefining it.
- Constants can hold **arrays** since PHP 7.0.
- **Class constants** use `const` inside a class, accessed via `ClassName::CONSTANT`. Can be `public`, `protected`, or `private`.
- Use `self::CONSTANT` inside the same class and `static::CONSTANT` for late static binding in inheritance.
- PHP has many **built-in constants**: `PHP_VERSION`, `PHP_INT_MAX`, `PHP_EOL`, `DIRECTORY_SEPARATOR`, `M_PI`, etc.
- **Magic constants** (`__FILE__`, `__DIR__`, `__LINE__`, `__CLASS__`, `__METHOD__`, etc.) change value based on context — especially useful for debugging and file paths.
- `__DIR__` is the most practically useful magic constant — always use it for reliable file paths.