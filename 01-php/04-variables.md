# PHP Variables

A **variable** is a named container that stores a value in memory. In PHP, variables are one of the most fundamental building blocks — almost everything you write will involve them. PHP variables are flexible, dynamic, and easy to work with once you understand the rules.

---

## Table of Contents

1. [What is a Variable?](#what-is-a-variable)
2. [Declaring & Assigning Variables](#declaring--assigning-variables)
3. [Naming Rules](#naming-rules)
4. [Variable Types are Dynamic](#variable-types-are-dynamic)
5. [Variable Variables](#variable-variables)
6. [Assigning by Value vs by Reference](#assigning-by-value-vs-by-reference)
7. [Variable Scope](#variable-scope)
   - [Local Scope](#local-scope)
   - [Global Scope & the `global` Keyword](#global-scope--the-global-keyword)
   - [$GLOBALS Superglobal](#globals-superglobal)
   - [Static Variables](#static-variables)
8. [Constants](#constants)
   - [define()](#define)
   - [const](#const)
   - [Magic Constants](#magic-constants)
9. [Superglobals](#superglobals)
10. [Checking Variables](#checking-variables)
11. [Destroying Variables](#destroying-variables)
12. [Quick Revision](#quick-revision)

---

## What is a Variable?

- A **variable** holds a value that can change during the execution of a script.
- In PHP, variables always start with a **dollar sign** `$`.
- PHP is **loosely typed** — you don't declare the type, PHP figures it out from the value.

```php
<?php
$message = "Hello, World!";  // Stores a string
$score   = 100;               // Stores an integer
$price   = 9.99;              // Stores a float
$isReady = true;              // Stores a boolean
?>
```

> 💡 **Think of a variable like a labeled box.** The label is the variable name, and what's inside the box is the value. You can change what's inside the box any time.

---

## Declaring & Assigning Variables

- In PHP, you **declare and assign** a variable in one step using `=` (the assignment operator).
- There is no separate declaration step like in Java or C.
- A variable comes into existence the moment you assign a value to it.

```php
<?php
// Declaration + Assignment in one step
$name = "Phyo";
$age  = 25;

// You can assign a new value any time — the old value is replaced
$name = "Min";   // Now $name is "Min", not "Phyo"

// You can assign the value of one variable to another
$a = 10;
$b = $a;   // $b gets a copy of $a's value (10)

echo $b;   // Output: 10
?>
```

> ⚠️ **Warning:** Using a variable before assigning it produces a **Notice** (PHP 8: warning/error). Always assign a value before using a variable.

```php
<?php
echo $undefined;  // Notice: Undefined variable $undefined
?>
```

---

## Naming Rules

PHP variable names must follow strict rules — breaking them causes a **parse error**.

| Rule | Example |
|---|---|
| Must start with `$` | `$name` ✅ |
| Must start with a letter or underscore after `$` | `$_name`, `$name` ✅ |
| Cannot start with a number after `$` | `$1name` ❌ |
| Can contain letters, numbers, underscores | `$user_name`, `$item2` ✅ |
| Cannot contain spaces or special characters | `$my name`, `$my-name` ❌ |
| **Case-sensitive** | `$name` ≠ `$Name` ≠ `$NAME` |

```php
<?php
// ✅ Valid variable names
$name       = "Phyo";
$_private   = "hidden";
$user2      = "Alice";
$firstName  = "Bob";        // camelCase — common in PHP
$first_name = "Charlie";    // snake_case — also common in PHP

// ❌ Invalid variable names (these cause parse errors)
// $2fast     = "nope";
// $my-var    = "nope";
// $my var    = "nope";
?>
```

> 💡 **Convention:** PHP developers typically use **snake_case** (`$first_name`) for variables, following the PSR coding standards. Both camelCase and snake_case are widely used.

---

## Variable Types are Dynamic

- The **type** of a PHP variable can change simply by assigning a different value.
- PHP automatically updates the type — no extra work needed.

```php
<?php
$variable = "Hello";      // String
var_dump($variable);      // string(5) "Hello"

$variable = 42;           // Now it's an Integer
var_dump($variable);      // int(42)

$variable = 3.14;         // Now it's a Float
var_dump($variable);      // float(3.14)

$variable = true;         // Now it's a Boolean
var_dump($variable);      // bool(true)

$variable = null;         // Now it's NULL
var_dump($variable);      // NULL
?>
```

> ⚠️ **Warning:** While dynamic typing is convenient, accidentally reassigning the wrong type to a variable is a common source of bugs. Be intentional about your variable values.

---

## Variable Variables

- A **variable variable** uses the **value** of one variable as the **name** of another variable.
- Written with a double dollar sign `$$`.
- Useful in advanced scenarios but can make code hard to read — use sparingly.

```php
<?php
$varName = "city";   // $varName holds the string "city"
$$varName = "Yangon"; // This creates a variable called $city with value "Yangon"

echo $city;          // Output: Yangon
echo $$varName;      // Output: Yangon  (same thing, accessed differently)

// Another example
$key = "color";
$$key = "blue";

echo $color;         // Output: blue
?>
```

> ⚠️ **Warning:** Variable variables make code hard to read and debug. Avoid them in everyday code — use arrays instead when you need dynamic key-value storage.

---

## Assigning by Value vs by Reference

### By Value (Default)

- When you assign a variable normally, PHP copies the **value**.
- Changing one variable does **not** affect the other.

```php
<?php
$a = 10;
$b = $a;   // $b gets a COPY of $a

$b = 99;   // Only $b changes

echo $a;   // Output: 10  ($a is unchanged)
echo $b;   // Output: 99
?>
```

---

### By Reference (`&`)

- When you assign by **reference**, both variables point to the **same value in memory**.
- Changing one **also changes the other**.
- Use `&` before the variable name to assign by reference.

```php
<?php
$a = 10;
$b = &$a;  // $b is a REFERENCE to $a — they share the same memory

$b = 99;   // Changing $b also changes $a

echo $a;   // Output: 99  ($a changed!)
echo $b;   // Output: 99

// Unsetting a reference
unset($b);  // Destroys $b, but $a still exists with its value
echo $a;    // Output: 99
?>
```

> 💡 **When to use references:** References are useful when passing large data structures to functions and you want to avoid copying them, or when you need a function to modify the original variable.

---

### By Value vs By Reference — Summary

| Feature | By Value (`=`) | By Reference (`&=`) |
|---|---|---|
| What is copied | The value | A pointer to the same memory |
| Changing one affects other? | ❌ No | ✅ Yes |
| Default behavior | ✅ Yes | ❌ No — explicit `&` required |

---

## Variable Scope

- **Scope** defines where a variable can be accessed in your code.
- PHP has three types of scope: **local**, **global**, and **static**.

---

### Local Scope

- A variable declared **inside a function** is **local** to that function.
- It cannot be accessed outside the function.
- Variables with the same name inside and outside a function are completely separate.

```php
<?php
$message = "I am global";   // Global variable

function greet() {
    $message = "I am local"; // Local variable — separate from the global one
    echo $message;           // Output: I am local
}

greet();
echo $message;  // Output: I am global  (global variable is unchanged)
?>
```

> 💡 **PHP is different from JavaScript here.** In JS, functions can access outer variables. In PHP, functions are completely isolated — you must explicitly bring outer variables in.

---

### Global Scope & the `global` Keyword

- To access a **global variable inside a function**, you must declare it with the `global` keyword.
- Without `global`, PHP treats it as a new local variable.

```php
<?php
$count = 0;     // Global variable

function increment() {
    global $count;  // Tell PHP to use the global $count
    $count++;       // Now modifies the global variable
}

increment();
increment();
increment();

echo $count;   // Output: 3
?>
```

> ⚠️ **Warning:** Using `global` makes functions depend on external state, which makes code harder to test and debug. Prefer **passing variables as parameters** and **returning values** instead.

```php
<?php
// ✅ Better approach — pass and return
function increment($count) {
    return $count + 1;
}

$count = 0;
$count = increment($count);  // 1
$count = increment($count);  // 2
$count = increment($count);  // 3

echo $count;  // Output: 3
?>
```

---

### $GLOBALS Superglobal

- `$GLOBALS` is a special built-in array that holds **all global variables**.
- You can access and modify global variables inside functions using `$GLOBALS["varname"]` — no `global` keyword needed.

```php
<?php
$username = "Phyo";
$score    = 100;

function showUser() {
    echo $GLOBALS["username"];  // Access global variable via $GLOBALS
    $GLOBALS["score"] += 50;    // Modify global variable
}

showUser();         // Output: Phyo
echo $score;        // Output: 150
?>
```

---

### Static Variables

- Normally, a local variable is **destroyed** when a function finishes.
- A **static variable** keeps its value between function calls.
- Declared with the `static` keyword.

```php
<?php
function counter() {
    static $count = 0;  // Initialized only ONCE, then remembered between calls
    $count++;
    echo $count . "\n";
}

counter();  // Output: 1
counter();  // Output: 2
counter();  // Output: 3

// Without static:
function normalCounter() {
    $count = 0;  // Reset to 0 on every call
    $count++;
    echo $count . "\n";
}

normalCounter();  // Output: 1
normalCounter();  // Output: 1  ← always 1, resets every time
normalCounter();  // Output: 1
?>
```

> 💡 **Use case:** Static variables are great for counters, memoization, or tracking how many times a function has been called — without needing a global variable.

---

### Variable Scope — Summary

| Scope | Where Declared | Accessible Where |
|---|---|---|
| **Local** | Inside a function | Only inside that function |
| **Global** | Outside all functions | Outside functions; inside with `global` |
| **Static** | Inside a function (with `static`) | Only inside that function, but persists between calls |

---

## Constants

- A **constant** is like a variable, but its value **cannot change** once defined.
- Constants are **global by default** — accessible anywhere in the script.
- By convention, constant names are written in **ALL_CAPS**.
- Constants do **not** use the `$` prefix.

---

### `define()`

- The classic way to define constants — works anywhere in a script.

```php
<?php
define("SITE_NAME", "My PHP App");
define("MAX_USERS", 100);
define("PI", 3.14159);
define("DEBUG", true);

echo SITE_NAME;   // Output: My PHP App
echo MAX_USERS;   // Output: 100

// ❌ Cannot reassign
// SITE_NAME = "Other";   // Error!
// define("SITE_NAME", "Other"); // Silently ignored or warning

// Constants are global — accessible inside functions too
function showSite() {
    echo SITE_NAME;  // Works! No 'global' keyword needed
}
showSite();  // Output: My PHP App
?>
```

---

### `const`

- The modern way to define constants — must be used at the **top level** or inside a class.
- Cannot be used inside functions or conditional blocks.
- Slightly faster than `define()`.

```php
<?php
const VERSION = "1.0.0";
const DB_HOST = "localhost";

echo VERSION;   // Output: 1.0.0
echo DB_HOST;   // Output: localhost
?>
```

---

### `define()` vs `const`

| Feature | `define()` | `const` |
|---|---|---|
| Use inside functions | ✅ Yes | ❌ No |
| Use in conditionals | ✅ Yes | ❌ No |
| Use inside classes | ❌ No | ✅ Yes |
| Dynamic name (variable) | ✅ Yes | ❌ No |
| Performance | Slightly slower | Slightly faster |

```php
<?php
// ✅ define() with dynamic name
$name = "APP_ENV";
define($name, "production");
echo APP_ENV;   // production

// ✅ const inside a class
class Config {
    const VERSION = "2.0";
}
echo Config::VERSION;  // 2.0
?>
```

---

### Magic Constants

- **Magic constants** are built-in constants whose values change depending on where they are used.
- They are always available — no need to define them.

```php
<?php
echo __LINE__;     // Current line number in the file
echo __FILE__;     // Full path and filename of the current file
echo __DIR__;      // Directory of the current file
echo __FUNCTION__; // Name of the current function (empty if outside function)
echo __CLASS__;    // Name of the current class (empty if outside class)
echo __METHOD__;   // Name of the current class method
echo __NAMESPACE__; // Current namespace name
?>
```

```php
<?php
function myFunc() {
    echo __FUNCTION__;  // Output: myFunc
}

class MyClass {
    public function myMethod() {
        echo __CLASS__;   // Output: MyClass
        echo __METHOD__;  // Output: MyClass::myMethod
    }
}
?>
```

> 💡 **Useful for:** Debugging (`__FILE__`, `__LINE__`), autoloading files (`__DIR__`), and framework-level code.

---

## Superglobals

- **Superglobals** are built-in PHP variables that are **always available everywhere** — inside functions, classes, and files — without using the `global` keyword.
- They are all arrays prefixed with `$_`.

| Superglobal | Contains |
|---|---|
| `$_GET` | Data from URL query string `?name=Phyo` |
| `$_POST` | Data from HTML form (POST method) |
| `$_REQUEST` | Combined `$_GET`, `$_POST`, and `$_COOKIE` data |
| `$_SERVER` | Server and request info (IP, headers, file path, etc.) |
| `$_SESSION` | Session data for the current user |
| `$_COOKIE` | Cookie data sent by the browser |
| `$_FILES` | Data about uploaded files |
| `$_ENV` | Environment variables |
| `$GLOBALS` | All global variables |

```php
<?php
// URL: example.com/page.php?username=Phyo&age=25

echo $_GET["username"];  // Output: Phyo
echo $_GET["age"];       // Output: 25

// Server info
echo $_SERVER["SERVER_NAME"];    // Output: example.com
echo $_SERVER["REQUEST_METHOD"]; // Output: GET
echo $_SERVER["PHP_SELF"];       // Output: /page.php
echo $_SERVER["REMOTE_ADDR"];    // Output: user's IP address
?>
```

> ⚠️ **Warning:** **Never trust superglobal input directly.** Data from `$_GET`, `$_POST`, and `$_COOKIE` comes from users and can be malicious. Always **validate and sanitize** input before using it.

```php
<?php
// ❌ Dangerous — directly using user input
echo $_GET["name"];

// ✅ Safer — sanitize first
echo htmlspecialchars($_GET["name"], ENT_QUOTES, "UTF-8");
?>
```

---

## Checking Variables

- PHP provides several functions to inspect and validate variables before using them.

```php
<?php
$name  = "Phyo";
$empty = "";
$zero  = 0;
$null  = null;

// isset() — true if variable exists AND is not NULL
var_dump(isset($name));    // bool(true)
var_dump(isset($null));    // bool(false) — NULL counts as "not set"
var_dump(isset($unknown)); // bool(false) — undefined variable

// empty() — true if variable is "empty" (falsy values)
var_dump(empty($empty));   // bool(true)  — empty string
var_dump(empty($zero));    // bool(true)  — 0 is considered empty
var_dump(empty($name));    // bool(false) — "Phyo" is not empty
var_dump(empty($null));    // bool(true)  — null is empty

// is_null() — true only if value is exactly NULL
var_dump(is_null($null));  // bool(true)
var_dump(is_null($zero));  // bool(false) — 0 is not NULL

// isset() vs empty() vs is_null()
?>
```

### `isset()` vs `empty()` vs `is_null()`

| Function | Returns `true` when |
|---|---|
| `isset($var)` | Variable exists AND is not `null` |
| `empty($var)` | Variable is falsy: `""`, `0`, `"0"`, `[]`, `null`, `false` |
| `is_null($var)` | Variable is exactly `null` |

> 💡 **Tip:** Use `isset()` before accessing variables that may not exist (like `$_GET` values) to avoid undefined variable warnings.

```php
<?php
// ✅ Safe pattern for reading optional inputs
$username = isset($_GET["username"]) ? $_GET["username"] : "Guest";

// ✅ Even cleaner with PHP 7+ null coalescing operator ??
$username = $_GET["username"] ?? "Guest";

echo $username;  // "Phyo" if ?username=Phyo in URL, otherwise "Guest"
?>
```

---

## Destroying Variables

- Use `unset()` to **destroy** a variable — free its memory and remove it from scope.
- After `unset()`, the variable no longer exists (`isset()` returns `false`).

```php
<?php
$name = "Phyo";
echo $name;        // Output: Phyo

unset($name);
echo isset($name); // Output: (nothing — false)
// echo $name;     // Notice: Undefined variable

// Unset multiple variables at once
$a = 1;
$b = 2;
$c = 3;
unset($a, $b, $c);

// Unset an array element (doesn't destroy the whole array)
$colors = ["red", "green", "blue"];
unset($colors[1]);       // Removes "green"
print_r($colors);        // Array ( [0] => red [2] => blue )
// Note: Keys are NOT re-indexed automatically after unset
?>
```

> 💡 **Tip:** After `unset()` on an array element, keys are not re-indexed. Use `array_values()` to reset the keys if needed.

```php
<?php
$colors = ["red", "green", "blue"];
unset($colors[1]);
$colors = array_values($colors);  // Re-index: [0] => red, [1] => blue
print_r($colors);
?>
```

---

## Quick Revision

- A **variable** starts with `$` and stores a value. No type declaration needed — PHP is loosely typed.
- Variable names are **case-sensitive**: `$name` ≠ `$Name`.
- Name rules: start with letter or `_` after `$`, only letters/numbers/underscores, no spaces or special characters.
- **Dynamic typing** — the type of a variable changes automatically when you assign a different value.
- **Variable variables** (`$$var`) use a variable's value as another variable's name — use sparingly.
- **By value** (`=`) copies the value; **by reference** (`&=`) makes two variables share the same memory.
- **Local scope** — variables inside functions are isolated. PHP functions cannot see outer variables without `global`.
- **`global $var`** — brings a global variable into a function's scope. Prefer passing parameters instead.
- **`$GLOBALS["var"]`** — alternative to `global`; access any global variable from anywhere.
- **Static variables** (`static $var`) remember their value between function calls without being global.
- **Constants** — defined with `define()` or `const`, written in `ALL_CAPS`, no `$`, cannot be reassigned.
- Use `define()` for constants inside functions/conditions; use `const` at the top level or inside classes.
- **Magic constants** (`__FILE__`, `__LINE__`, `__DIR__`, etc.) change value based on where they appear.
- **Superglobals** (`$_GET`, `$_POST`, `$_SERVER`, etc.) are available everywhere — always sanitize user input from them.
- Use `isset()` before reading variables that might not exist; use `??` (null coalescing) for clean defaults.
- `unset()` destroys a variable or removes an array element — use `array_values()` to re-index after.