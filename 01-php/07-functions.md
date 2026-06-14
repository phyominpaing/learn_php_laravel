# PHP Functions

A **function** is a reusable block of code that performs a specific task. Instead of writing the same code over and over, you wrap it in a function, give it a name, and call it whenever you need it. Functions are one of the most important tools for writing clean, organized, and maintainable PHP code.

---

## Table of Contents

1. [What is a Function?](#what-is-a-function)
2. [User-Defined Functions](#user-defined-functions)
3. [Function Arguments](#function-arguments)
   - [Default Argument Values](#default-argument-values)
   - [Named Arguments (PHP 8+)](#named-arguments-php-8)
   - [Variadic Functions (`...`)](#variadic-functions-)
   - [Pass by Value vs Pass by Reference](#pass-by-value-vs-pass-by-reference)
   - [Type Declarations (Type Hints)](#type-declarations-type-hints)
4. [Returning Values](#returning-values)
   - [Return Type Declarations](#return-type-declarations)
   - [Returning Multiple Values](#returning-multiple-values)
5. [Variable Functions](#variable-functions)
6. [Anonymous Functions (Closures)](#anonymous-functions-closures)
7. [Arrow Functions (PHP 7.4+)](#arrow-functions-php-74)
8. [Recursive Functions](#recursive-functions)
9. [Built-in (Internal) Functions](#built-in-internal-functions)
10. [Variable Scope in Functions](#variable-scope-in-functions)
11. [Quick Revision](#quick-revision)

---

## What is a Function?

- A **function** groups a set of instructions under a single name.
- You **define** it once and **call** (invoke) it as many times as needed.
- Functions help with **DRY** (Don't Repeat Yourself) — a core programming principle.

```php
<?php
// Defining a function
function sayHello() {
    echo "Hello, World!";
}

// Calling the function
sayHello();  // Output: Hello, World!
sayHello();  // Output: Hello, World!  (can call it as many times as you want)
?>
```

> 💡 **Naming convention:** Function names in PHP are typically written in **camelCase** (`getUserData`) or **snake_case** (`get_user_data`). Built-in PHP functions use snake_case (`str_replace`, `array_map`).

---

## User-Defined Functions

- A **user-defined function** is one **you** create, as opposed to PHP's **built-in functions**.
- Defined using the `function` keyword.
- Function names are **case-insensitive** (unlike variables).

```php
<?php
function greet() {
    echo "Welcome to PHP!";
}

greet();  // Output: Welcome to PHP!
GREET();  // Output: Welcome to PHP!  ← works! function names are case-insensitive
Greet();  // Output: Welcome to PHP!
?>
```

### Function Naming Rules

| Rule | Example |
|---|---|
| Must start with a letter or underscore | `function getName()` ✅ |
| Cannot start with a number | `function 2getName()` ❌ |
| Can contain letters, numbers, underscores | `function get_user_2()` ✅ |
| Cannot contain spaces or hyphens | `function get-user()` ❌ |
| Case-insensitive | `myFunc()` = `MYFUNC()` = `MyFunc()` |

> ⚠️ **Warning:** Even though function names are case-insensitive, **always call functions using the exact case you defined them** — it keeps your code consistent and readable.

---

## Function Arguments

- **Arguments** (also called **parameters**) are values passed INTO a function so it can use them.
- Defined inside the parentheses `()` when declaring the function.

```php
<?php
// One parameter
function greet($name) {
    echo "Hello, $name!";
}
greet("Phyo");  // Output: Hello, Phyo!
greet("Alice"); // Output: Hello, Alice!

// Multiple parameters
function addNumbers($a, $b) {
    echo $a + $b;
}
addNumbers(5, 10);  // Output: 15
?>
```

> 💡 **Terminology:** "Parameter" refers to the variable in the function definition (`$name`). "Argument" refers to the actual value passed when calling (`"Phyo"`). In practice, people use both terms interchangeably.

---

### Default Argument Values

- You can give a parameter a **default value** — used automatically if no argument is passed for it.
- Default-valued parameters should come **after** required parameters.

```php
<?php
function greet($name, $greeting = "Hello") {
    echo "$greeting, $name!";
}

greet("Phyo");              // Output: Hello, Phyo!  (uses default greeting)
greet("Phyo", "Welcome");   // Output: Welcome, Phyo! (overrides default)

// Multiple defaults
function createUser($name, $role = "member", $active = true) {
    echo "$name | $role | " . ($active ? "active" : "inactive");
}

createUser("Alice");                    // Alice | member | active
createUser("Bob", "admin");             // Bob | admin | active
createUser("Charlie", "editor", false); // Charlie | editor | inactive
?>
```

> ⚠️ **Common Mistake:** Required parameters **cannot** come after parameters with default values.

```php
<?php
// ❌ Fatal Error — required parameter after optional one
// function greet($greeting = "Hello", $name) { ... }

// ✅ Correct order — required first, optional last
function greet($name, $greeting = "Hello") { ... }
?>
```

---

### Named Arguments (PHP 8+)

- **Named arguments** let you pass values by **specifying the parameter name**, regardless of order.
- Useful for skipping optional parameters or improving readability.

```php
<?php
function createUser($name, $role = "member", $active = true, $country = "Unknown") {
    echo "$name | $role | " . ($active ? "active" : "inactive") . " | $country";
}

// Without named arguments — must provide all in order to set $country
createUser("Phyo", "member", true, "Myanmar");

// ✅ With named arguments — skip the ones you don't need to change
createUser("Phyo", country: "Myanmar");
// Output: Phyo | member | active | Myanmar

// Order doesn't matter with named arguments
createUser(name: "Alice", country: "USA", role: "admin");
// Output: Alice | admin | active | USA
?>
```

> 💡 **Tip:** Named arguments make function calls **self-documenting** — you can immediately see what each value represents without checking the function definition.

---

### Variadic Functions (`...`)

- A **variadic function** accepts an **unlimited number of arguments** using the `...` (spread/splat) operator.
- All extra arguments are collected into an **array**.

```php
<?php
function sum(...$numbers) {
    $total = 0;
    foreach ($numbers as $num) {
        $total += $num;
    }
    return $total;
}

echo sum(1, 2, 3);        // Output: 6
echo sum(1, 2, 3, 4, 5);  // Output: 15
echo sum(10);             // Output: 10
echo sum();               // Output: 0

// Combining regular and variadic parameters
function greetAll($greeting, ...$names) {
    foreach ($names as $name) {
        echo "$greeting, $name!\n";
    }
}

greetAll("Hello", "Alice", "Bob", "Charlie");
// Output:
// Hello, Alice!
// Hello, Bob!
// Hello, Charlie!
?>
```

### Spread Operator — Unpacking Arrays into Arguments

- The `...` operator also works in **reverse** — unpacking an array into individual arguments when calling a function.

```php
<?php
function addThree($a, $b, $c) {
    return $a + $b + $c;
}

$numbers = [1, 2, 3];

echo addThree(...$numbers);  // Output: 6
// Equivalent to: addThree(1, 2, 3)
?>
```

---

### Pass by Value vs Pass by Reference

#### Pass by Value (Default)

- By default, arguments are passed **by value** — the function receives a **copy**.
- Changes inside the function **do not** affect the original variable.

```php
<?php
function addOne($num) {
    $num++;
    echo "Inside function: $num\n";
}

$x = 5;
addOne($x);
echo "Outside function: $x\n";

// Output:
// Inside function: 6
// Outside function: 5   ← unchanged!
?>
```

#### Pass by Reference (`&`)

- Adding `&` before the parameter name passes the **actual variable** — changes inside the function **affect the original**.

```php
<?php
function addOne(&$num) {
    $num++;
    echo "Inside function: $num\n";
}

$x = 5;
addOne($x);
echo "Outside function: $x\n";

// Output:
// Inside function: 6
// Outside function: 6   ← changed!
?>
```

> ⚠️ **Warning:** Pass by reference can make code harder to follow because a function can silently modify variables from outside its scope. Use it intentionally and document it clearly — e.g., when a function needs to modify a large array without copying it.

```php
<?php
// Real-world example: a function that modifies an array in place
function addTax(array &$prices, $taxRate) {
    foreach ($prices as &$price) {
        $price += $price * $taxRate;
    }
}

$prices = [100, 200, 300];
addTax($prices, 0.1);
print_r($prices);  // [110, 220, 330]  ← original array modified
?>
```

---

### Type Declarations (Type Hints)

- PHP allows you to **declare the expected type** of a parameter.
- If a wrong type is passed, PHP throws a `TypeError` (in strict mode) or tries to convert it (in non-strict/coercive mode, the default).

```php
<?php
function multiply(int $a, int $b) {
    return $a * $b;
}

echo multiply(4, 5);     // Output: 20
echo multiply("4", "5"); // Output: 20  — numeric strings are coerced to int

// Supported types: int, float, string, bool, array, callable, iterable, object, mixed, and class names

function greet(string $name): string {
    return "Hello, $name!";
}

// Nullable types — allow the type OR null
function findUser(?int $id) {
    if ($id === null) {
        return "No ID provided";
    }
    return "Looking up user #$id";
}

echo findUser(5);     // Looking up user #5
echo findUser(null);  // No ID provided
?>
```

### Strict Types

- By default, PHP uses **coercive typing** — it tries to convert values to match the declared type.
- Add `declare(strict_types=1);` at the **top of the file** to enforce **strict typing** — no automatic conversion.

```php
<?php
declare(strict_types=1);  // Must be the FIRST statement in the file

function multiply(int $a, int $b): int {
    return $a * $b;
}

echo multiply(4, 5);     // ✅ Output: 20
// echo multiply("4", 5);   // ❌ TypeError — "4" is a string, not an int (with strict_types)
?>
```

> 💡 **Tip:** Use `declare(strict_types=1);` in larger projects to catch type-related bugs early. It forces you to be explicit and intentional about types.

---

## Returning Values

- The `return` keyword sends a value **back** to wherever the function was called.
- A function **stops executing immediately** when `return` is reached.
- If no `return` is used, the function returns `NULL` by default.

```php
<?php
function add($a, $b) {
    return $a + $b;   // Sends the result back
    echo "This never runs"; // ❌ Unreachable code — function already exited
}

$result = add(5, 10);
echo $result;  // Output: 15

// Using the return value directly
echo add(2, 3);          // Output: 5
echo add(2, 3) * 2;       // Output: 10  (can use it in further expressions)

// No return statement
function logMessage($msg) {
    echo "Log: $msg";
    // No return — function returns NULL
}

$result = logMessage("Test"); // Output: Log: Test
var_dump($result);            // NULL
?>
```

---

### Return Type Declarations

- Just like parameters, you can declare the **return type** of a function using `: type` after the parentheses.

```php
<?php
function add(int $a, int $b): int {
    return $a + $b;
}

function getName(): string {
    return "Phyo";
}

function isAdult(int $age): bool {
    return $age >= 18;
}

// void — function returns nothing
function logMessage(string $msg): void {
    echo "Log: $msg";
    // Cannot use "return $something;" — only "return;" or nothing
}

// Nullable return type
function findUser(int $id): ?string {
    if ($id === 1) {
        return "Phyo";
    }
    return null;  // Allowed because of the ? before string
}
?>
```

> 💡 **Tip:** Adding return types makes functions **self-documenting** and helps catch bugs — if you accidentally `return "text";` from a function declared `: int`, PHP will warn or error.

---

### Returning Multiple Values

- PHP functions can only `return` **one value**, but you can return an **array** (or use **list assignment / destructuring**) to simulate multiple return values.

```php
<?php
function getMinMax(array $numbers) {
    return [min($numbers), max($numbers)];
}

$result = getMinMax([3, 7, 1, 9, 4]);
echo $result[0];  // 1  (min)
echo $result[1];  // 9  (max)

// ✅ Cleaner — destructure directly into variables
[$min, $max] = getMinMax([3, 7, 1, 9, 4]);
echo $min;  // 1
echo $max;  // 9

// Named destructuring with associative arrays (PHP 7.1+)
function getUser() {
    return ["name" => "Phyo", "age" => 25];
}

["name" => $name, "age" => $age] = getUser();
echo $name;  // Phyo
echo $age;   // 25
?>
```

---

## Variable Functions

- If a variable holds the **name of a function as a string**, you can call it by appending `()` to the variable.
- Useful for dynamic function calls, callback systems, and routing.

```php
<?php
function sayHello() {
    echo "Hello!";
}

function sayGoodbye() {
    echo "Goodbye!";
}

$action = "sayHello";
$action();   // Output: Hello!  — calls sayHello()

$action = "sayGoodbye";
$action();   // Output: Goodbye! — calls sayGoodbye()

// Practical example: simple router
$page = "home";

function home()    { echo "Welcome to the Home page"; }
function about()   { echo "About us"; }
function contact() { echo "Contact us"; }

if (function_exists($page)) {
    $page();  // Calls home(), about(), or contact() based on $page
}
// Output: Welcome to the Home page
?>
```

> ⚠️ **Warning:** Always use `function_exists()` before calling a variable function with user-provided input — calling an undefined function causes a `Fatal Error`. Never use raw user input directly as a function name (security risk).

---

## Anonymous Functions (Closures)

- An **anonymous function** is a function **without a name** — often used as a value, passed around, or assigned to a variable.
- Also called a **closure** because it can "close over" (capture) variables from its surrounding scope.

```php
<?php
// Basic anonymous function
$greet = function ($name) {
    return "Hello, $name!";
};

echo $greet("Phyo");  // Output: Hello, Phyo!

// Passing an anonymous function as a callback
$numbers = [1, 2, 3, 4, 5];
$doubled = array_map(function ($n) {
    return $n * 2;
}, $numbers);

print_r($doubled);  // [2, 4, 6, 8, 10]
?>
```

### Capturing Outer Variables with `use`

- By default, anonymous functions **cannot access** variables from the surrounding scope.
- Use the `use` keyword to **import** outer variables into the closure.

```php
<?php
$tax = 0.07;

// ❌ Without 'use' — $tax is not accessible inside
$calculateTotal = function ($price) {
    // return $price + ($price * $tax); // Error: undefined variable $tax
};

// ✅ With 'use' — captures $tax by VALUE (a copy)
$calculateTotal = function ($price) use ($tax) {
    return $price + ($price * $tax);
};

echo $calculateTotal(100);  // Output: 107

// 'use (&$var)' — captures by REFERENCE (changes affect outer variable)
$counter = 0;

$increment = function () use (&$counter) {
    $counter++;
};

$increment();
$increment();
$increment();

echo $counter;  // Output: 3  ← outer variable changed!
?>
```

> 💡 **`use ($var)` vs `use (&$var)`:** Without `&`, the closure gets a **snapshot** (copy) of the variable at the time the closure is created. With `&`, the closure shares the **same variable** — changes are reflected outside.

```php
<?php
// Demonstrating the difference clearly
$x = 10;

$byValue     = function () use ($x)  { return $x; };
$byReference = function () use (&$x) { return $x; };

$x = 99;  // Change AFTER closures are created

echo $byValue();      // Output: 10  (captured the OLD value)
echo $byReference();  // Output: 99  (sees the CURRENT value)
?>
```

---

## Arrow Functions (PHP 7.4+)

- **Arrow functions** are a **shorthand syntax** for anonymous functions — shorter and cleaner, especially for simple one-line operations.
- Syntax: `fn (parameters) => expression`
- The biggest difference from regular closures: **arrow functions automatically capture outer variables by value** — no `use` keyword needed!

```php
<?php
// Regular anonymous function
$square = function ($n) {
    return $n * $n;
};

// ✅ Arrow function — same thing, much shorter
$square = fn($n) => $n * $n;

echo $square(5);  // Output: 25

// Using with array_map, array_filter, etc.
$numbers = [1, 2, 3, 4, 5];

$doubled = array_map(fn($n) => $n * 2, $numbers);
print_r($doubled);  // [2, 4, 6, 8, 10]

$evens = array_filter($numbers, fn($n) => $n % 2 === 0);
print_r($evens);    // [2 => 2, 4 => 4]
?>
```

### Automatic Variable Capture

```php
<?php
$tax = 0.07;

// ✅ No 'use' needed — automatically captures $tax from outer scope
$calculateTotal = fn($price) => $price + ($price * $tax);

echo $calculateTotal(100);  // Output: 107

// Multiple parameters
$add = fn($a, $b) => $a + $b;
echo $add(3, 4);  // Output: 7
?>
```

> ⚠️ **Limitation:** Arrow functions can only contain a **single expression** — no multiple statements, loops, or `if/else` blocks with `{}`. For anything more complex, use a regular anonymous function or named function.

```php
<?php
// ❌ Cannot do this with arrow functions
// $fn = fn($x) => { $y = $x * 2; return $y + 1; };  // Syntax error

// ✅ Use a regular closure instead for multi-statement logic
$fn = function ($x) {
    $y = $x * 2;
    return $y + 1;
};
echo $fn(5);  // Output: 11
?>
```

### Closures vs Arrow Functions — Summary

| Feature | Anonymous Function (`function`) | Arrow Function (`fn`) |
|---|---|---|
| Syntax length | Longer | Shorter |
| Capture outer variables | Manual with `use ($var)` | Automatic (by value) |
| Multiple statements | ✅ Yes | ❌ No — single expression only |
| `use (&$var)` by reference | ✅ Yes | ❌ Not supported |
| Best for | Complex logic, multi-line | Quick, simple one-liners (callbacks) |

---

## Recursive Functions

- A **recursive function** is a function that **calls itself** to solve a problem by breaking it into smaller sub-problems.
- Must have a **base case** to stop the recursion — otherwise it causes infinite recursion and a fatal error.

```php
<?php
// Classic example: factorial
function factorial($n) {
    if ($n <= 1) {
        return 1;  // Base case — stops the recursion
    }
    return $n * factorial($n - 1);  // Recursive call
}

echo factorial(5);  // Output: 120  (5 * 4 * 3 * 2 * 1)

// How it unfolds:
// factorial(5) = 5 * factorial(4)
//              = 5 * (4 * factorial(3))
//              = 5 * (4 * (3 * factorial(2)))
//              = 5 * (4 * (3 * (2 * factorial(1))))
//              = 5 * (4 * (3 * (2 * 1)))
//              = 120


// Another example: Fibonacci sequence
function fibonacci($n) {
    if ($n <= 1) {
        return $n;  // Base case
    }
    return fibonacci($n - 1) + fibonacci($n - 2);  // Recursive call
}

echo fibonacci(6);  // Output: 8  (0, 1, 1, 2, 3, 5, 8)
?>
```

> ⚠️ **Warning:** Always make sure your recursive function has a **base case** that will eventually be reached. Without one, you'll get a `Fatal error: Maximum function nesting level reached` (stack overflow).

> 💡 **Tip:** Recursion is elegant for tree-like or nested structures (folders, comments with replies, JSON data) but can be slower and use more memory than loops for simple repetitive tasks.

---

## Built-in (Internal) Functions

- PHP ships with **thousands of built-in functions** ready to use — no need to define them yourself.
- They cover strings, arrays, math, dates, files, JSON, regular expressions, and much more.

```php
<?php
// String functions
echo strlen("Hello");           // 5
echo strtoupper("hello");       // HELLO
echo str_replace("a", "b", "banana"); // bbnbnb

// Array functions
echo count([1, 2, 3]);          // 3
print_r(array_reverse([1, 2, 3])); // [3, 2, 1]
print_r(array_sum([1, 2, 3]));  // 6

// Math functions
echo round(4.567, 2);  // 4.57
echo max(3, 7, 2);     // 7
echo min(3, 7, 2);     // 2
echo abs(-5);          // 5

// Date functions
echo date("Y-m-d");           // e.g. 2026-06-14
echo date("H:i:s");            // e.g. 14:30:00

// JSON functions
$data = ["name" => "Phyo", "age" => 25];
$json = json_encode($data);    // '{"name":"Phyo","age":25}'
$arr  = json_decode($json, true); // back to array

// Checking if a function exists
var_dump(function_exists("strlen"));    // bool(true)
var_dump(function_exists("my_func"));   // bool(false) — unless you defined it
?>
```

### Finding Built-in Functions

- The official **PHP Manual** (php.net) documents every built-in function with examples.
- Use `php -r "var_dump(get_defined_functions());"` in the CLI to list all available functions.

> 💡 **Tip:** Before writing your own helper function, **search the PHP manual** — there's a very high chance PHP already has a built-in function for what you need (string manipulation, array operations, date formatting, etc.).

---

## Variable Scope in Functions

- Quick recap (covered in depth in the Variables notes): variables declared inside a function are **local** by default and don't affect variables outside.

```php
<?php
$message = "Outside";

function showMessage() {
    $message = "Inside";  // Different variable — local scope
    echo $message;        // Output: Inside
}

showMessage();
echo $message;  // Output: Outside  (unchanged)

// To access outer variables, pass them as arguments (best practice)
function showMessage2($message) {
    echo $message;
}
showMessage2($message);  // Output: Outside
?>
```

> 💡 **Best Practice:** Prefer passing variables as **parameters** and getting results via `return` rather than relying on `global` or external state. This makes functions predictable, reusable, and easy to test.

---

## Quick Revision

- A **function** is a named, reusable block of code defined with `function` and called by its name followed by `()`.
- Function names are **case-insensitive**, unlike variables.
- **Parameters** receive values (**arguments**) passed when calling the function.
- **Default values** (`$x = 10`) let you skip arguments; required parameters must come **first**.
- **Named arguments** (PHP 8+) let you pass by parameter name, in any order — `func(name: "Phyo")`.
- **Variadic functions** (`...$args`) accept unlimited arguments collected into an array. The `...` also **unpacks** an array into individual arguments.
- **Pass by value** (default) gives the function a copy; **pass by reference** (`&$param`) lets the function modify the original variable.
- **Type declarations** (`int`, `string`, `?int`, etc.) specify expected parameter and return types. Add `declare(strict_types=1);` for strict type checking.
- `return` sends a value back and **immediately ends** the function. No `return` = returns `NULL`.
- To "return multiple values," return an **array** and use **destructuring**: `[$a, $b] = func();`
- **Variable functions** call a function whose name is stored in a string variable: `$fn(); `. Always check `function_exists()` first.
- **Anonymous functions (closures)** have no name, can be stored in variables, and use `use ($var)` to capture outer variables (`use (&$var)` for by-reference).
- **Arrow functions** (`fn($x) => expr`, PHP 7.4+) are shorthand closures that **automatically** capture outer variables but only support **single expressions**.
- **Recursive functions** call themselves and must have a **base case** to avoid infinite recursion.
- PHP has **thousands of built-in functions** — always check the PHP manual before writing your own.
- Functions have **local scope** by default — pass data in via parameters and out via `return` for predictable, testable code.