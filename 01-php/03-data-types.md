# PHP Data Types

A **data type** tells PHP what kind of value a variable holds. PHP automatically detects the type based on the value you assign — you don't have to declare it manually. This is called **dynamic typing** or **loosely typed** behavior.

---

## Table of Contents

1. [Overview of PHP Data Types](#overview-of-php-data-types)
2. [String](#string)
3. [Integer](#integer)
4. [Float](#float)
5. [Boolean](#boolean)
6. [NULL](#null)
7. [Array](#array)
8. [Object](#object)
9. [Resource](#resource)
10. [Checking & Inspecting Types](#checking--inspecting-types)
11. [Type Juggling & Type Casting](#type-juggling--type-casting)
12. [Quick Revision](#quick-revision)

---

## Overview of PHP Data Types

PHP has **8 data types** split into three categories:

| Category | Types |
|---|---|
| **Scalar** (single value) | String, Integer, Float, Boolean |
| **Compound** (multiple values) | Array, Object |
| **Special** | NULL, Resource |

```php
<?php
$name    = "Phyo";       // String
$age     = 25;           // Integer
$height  = 5.9;          // Float
$isAdmin = true;         // Boolean
$empty   = null;         // NULL
$colors  = ["red", "green", "blue"];  // Array
?>
```

> 💡 **PHP is loosely typed** — you never write `int $age = 25;` like in Java or C. PHP figures out the type on its own.

---

## String

- A **string** is a sequence of characters — letters, numbers, symbols, spaces.
- Strings are always wrapped in **quotes**.
- PHP supports two types of quotes: **single** `' '` and **double** `" "` — they behave differently.

---

### Single Quotes `' '`

- Everything inside single quotes is treated as **literal text**.
- Variables and escape sequences (except `\'` and `\\`) are **not processed**.
- Faster for simple static strings.

```php
<?php
$name = "Phyo";

echo 'Hello, $name';       // Output: Hello, $name  (variable NOT parsed)
echo 'It\'s a nice day';   // Output: It's a nice day  (\' to escape a quote)
echo 'Line1\\Line2';       // Output: Line1\Line2  (\\ to write a backslash)
?>
```

---

### Double Quotes `" "`

- Variables and escape sequences **are processed** inside double quotes.
- Use when you need to embed variables or special characters directly in a string.

```php
<?php
$name = "Phyo";
$age  = 25;

echo "Hello, $name";         // Output: Hello, Phyo
echo "Age: {$age} years";    // Output: Age: 25 years  (curly braces for clarity)
echo "Line1\nLine2";         // Output: Line1  (newline)
                             //         Line2
?>
```

#### Common Escape Sequences (Double Quotes Only)

| Escape | Meaning |
|---|---|
| `\n` | New line |
| `\t` | Tab |
| `\\` | Backslash `\` |
| `\"` | Double quote `"` |
| `\$` | Dollar sign `$` (to avoid variable parsing) |

---

### Heredoc & Nowdoc

- For long, multi-line strings — cleaner than concatenating many lines.

#### Heredoc (like double quotes — variables ARE parsed)

```php
<?php
$name = "Phyo";

$text = <<<EOT
Hello, $name.
Welcome to PHP.
This is a heredoc string.
EOT;

echo $text;
// Output:
// Hello, Phyo.
// Welcome to PHP.
// This is a heredoc string.
?>
```

#### Nowdoc (like single quotes — variables NOT parsed)

```php
<?php
$name = "Phyo";

$text = <<<'EOT'
Hello, $name.
This will NOT parse the variable.
EOT;

echo $text;
// Output:
// Hello, $name.
// This will NOT parse the variable.
?>
```

---

### String Concatenation

- Use the `.` dot operator to join strings together.
- Use `.=` to append to an existing string.

```php
<?php
$first = "Hello";
$last  = "World";

echo $first . ", " . $last . "!";  // Output: Hello, World!

$greeting = "Hello";
$greeting .= ", World!";           // Append to existing string
echo $greeting;                    // Output: Hello, World!
?>
```

---

### Useful String Functions

```php
<?php
$str = "  Hello, PHP World!  ";

echo strlen($str);             // 22 — length including spaces
echo strtoupper($str);         // "  HELLO, PHP WORLD!  "
echo strtolower($str);         // "  hello, php world!  "
echo trim($str);               // "Hello, PHP World!" — removes surrounding spaces
echo str_replace("PHP", "the", $str); // "  Hello, the World!  "
echo strpos($str, "PHP");      // 9 — position of first occurrence (0-indexed)
echo substr($str, 8, 3);       // "lo," — 3 chars starting at position 8
echo str_repeat("Ha", 3);      // "HaHaHa"
echo str_word_count(trim($str)); // 3
echo str_contains($str, "PHP"); // 1 (true) — PHP 8+
echo str_starts_with(trim($str), "Hello"); // 1 (true) — PHP 8+
echo str_ends_with(trim($str), "World!");  // 1 (true) — PHP 8+
?>
```

> 💡 **Tip:** `str_contains()`, `str_starts_with()`, and `str_ends_with()` were added in **PHP 8.0**. Use `strpos()` for older versions.

---

### Single vs Double Quotes — Summary

| Feature | Single `' '` | Double `" "` |
|---|---|---|
| Variable parsing | ❌ No | ✅ Yes |
| Escape sequences | ❌ Most don't work | ✅ All work |
| Speed | Slightly faster | Slightly slower |
| Use when | Static/literal text | Embedding variables |

---

## Integer

- An **integer** is a whole number — no decimal point.
- Can be **positive**, **negative**, or **zero**.
- PHP integers are 64-bit on most modern systems.

```php
<?php
$a =  42;      // Positive integer
$b = -10;      // Negative integer
$c =  0;       // Zero

// Other number formats PHP accepts
$hex  = 0x1A;  // Hexadecimal (base 16) → equals 26
$oct  = 0755;  // Octal (base 8)        → equals 493
$bin  = 0b1010; // Binary (base 2)      → equals 10

// PHP 7.4+ underscores for readability
$big = 1_000_000;  // Same as 1000000

echo PHP_INT_MAX;  // 9223372036854775807  (largest integer PHP can hold)
echo PHP_INT_MIN;  // -9223372036854775808 (smallest)
echo PHP_INT_SIZE; // 8 (bytes = 64 bits on most systems)
?>
```

> ⚠️ **Warning:** If a number exceeds `PHP_INT_MAX`, PHP automatically converts it to a **Float**. This can cause unexpected behavior in calculations.

---

### Integer Math Operators

```php
<?php
$a = 10;
$b = 3;

echo $a + $b;   // 13 — addition
echo $a - $b;   // 7  — subtraction
echo $a * $b;   // 30 — multiplication
echo $a / $b;   // 3.333... — division (returns float if not evenly divisible)
echo $a % $b;   // 1  — modulus (remainder)
echo $a ** $b;  // 1000 — exponentiation (10 to the power of 3)
echo intdiv($a, $b); // 3 — integer division (discards remainder)
?>
```

---

## Float

- A **float** (also called **double** or **floating-point number**) is a number with a decimal point or in exponential notation.
- Used for prices, measurements, scientific values, percentages, etc.

```php
<?php
$price    = 9.99;
$pi       = 3.14159;
$negative = -2.5;
$sci      = 1.5e3;   // Scientific notation → 1500.0
$small    = 2.5e-4;  // → 0.00025

echo PHP_FLOAT_MAX;     // 1.7976931348623E+308 (largest float)
echo PHP_FLOAT_EPSILON; // 2.2204460492503E-16  (smallest difference between floats)
?>
```

---

### ⚠️ The Float Precision Problem

- Floats are stored in **binary**, and most decimal fractions cannot be represented exactly.
- This causes tiny rounding errors that can break comparisons.

```php
<?php
$a = 0.1 + 0.2;
echo $a;                   // Output: 0.3  (looks fine...)
var_dump($a == 0.3);       // bool(false) ← WRONG! ❌

// Why? Because internally:
// 0.1 + 0.2 = 0.30000000000000004...

// ✅ Correct way — compare with a tolerance (epsilon)
$epsilon = 1.0E-9;
if (abs($a - 0.3) < $epsilon) {
    echo "They are equal!";  // ✅
}

// ✅ Or use round() for currency
echo round(0.1 + 0.2, 2);  // 0.3
?>
```

> ⚠️ **Warning:** **Never use `==` to compare floats directly.** Always use `round()` or an epsilon comparison. This catches many developers off guard.

---

### Useful Float Functions

```php
<?php
echo round(4.567, 2);   // 4.57  — round to 2 decimal places
echo ceil(4.1);         // 5     — round UP to nearest integer
echo floor(4.9);        // 4     — round DOWN to nearest integer
echo abs(-7.5);         // 7.5   — absolute value
echo fmod(10, 3);       // 1     — float modulus
echo number_format(1234567.891, 2, '.', ','); // 1,234,567.89
?>
```

---

## Boolean

- A **boolean** holds only one of two values: `true` or `false`.
- Used for conditions, flags, and controlling program flow.
- Keywords `true` and `false` are **case-insensitive** in PHP.

```php
<?php
$isLoggedIn = true;
$hasError   = false;
$isAdmin    = TRUE;   // Same as true

var_dump($isLoggedIn);  // bool(true)
var_dump($hasError);    // bool(false)
?>
```

---

### Truthy & Falsy Values

- PHP automatically converts other types to boolean in conditions.
- Knowing what counts as `false` is crucial.

**Values that evaluate to `false` (Falsy):**

| Value | Type |
|---|---|
| `false` | Boolean |
| `0` | Integer |
| `0.0` | Float |
| `""` | Empty string |
| `"0"` | String zero |
| `[]` | Empty array |
| `null` | NULL |

```php
<?php
// Everything else is truthy
if (0)       { echo "truthy"; } else { echo "falsy"; }  // falsy
if ("")      { echo "truthy"; } else { echo "falsy"; }  // falsy
if ("0")     { echo "truthy"; } else { echo "falsy"; }  // falsy
if ([])      { echo "truthy"; } else { echo "falsy"; }  // falsy
if (null)    { echo "truthy"; } else { echo "falsy"; }  // falsy
if (0.0)     { echo "truthy"; } else { echo "falsy"; }  // falsy

if (1)       { echo "truthy"; } else { echo "falsy"; }  // truthy
if ("hello") { echo "truthy"; } else { echo "falsy"; }  // truthy
if ("false") { echo "truthy"; } else { echo "falsy"; }  // truthy ← the STRING "false"!
if ([0])     { echo "truthy"; } else { echo "falsy"; }  // truthy ← non-empty array
?>
```

> ⚠️ **Common Mistake:** The string `"false"` is **truthy** because it is a non-empty string. Only the boolean `false` is falsy.

---

## NULL

- **NULL** represents a variable with **no value** — it is empty, absent, undefined.
- A variable is NULL if:
  - It was assigned `null` directly.
  - It was never assigned any value.
  - It was unset with `unset()`.

```php
<?php
$a = null;         // Explicitly set to NULL
$b;                // Also NULL — declared but never assigned

$c = "Hello";
unset($c);         // $c is now NULL — destroyed

var_dump($a);      // NULL
var_dump(is_null($a)); // bool(true)
var_dump(isset($a));   // bool(false) — NULL is treated as "not set"
?>
```

---

### NULL vs Empty String vs Zero

```php
<?php
$a = null;
$b = "";
$c = 0;

var_dump($a == $b);   // bool(true)  ← loose comparison
var_dump($a === $b);  // bool(false) ← strict comparison (different types)
var_dump($a == $c);   // bool(true)  ← loose comparison
var_dump($a === $c);  // bool(false) ← strict comparison

// Always use === to tell them apart
?>
```

> 💡 **Tip:** Use `isset()` to check if a variable exists and is not NULL. Use `is_null()` when you specifically want to check for NULL.

---

## Array

- An **array** stores **multiple values** in a single variable.
- PHP arrays are flexible — they can hold any mix of data types.
- PHP has two types of arrays: **indexed** and **associative**.

---

### Indexed Array

- Elements are stored with **numeric keys** starting from `0`.

```php
<?php
// Two ways to create an indexed array
$fruits = ["apple", "banana", "cherry"];  // ✅ Modern syntax (PHP 5.4+)
$colors = array("red", "green", "blue");  // Old syntax (still valid)

// Accessing elements
echo $fruits[0];   // apple
echo $fruits[1];   // banana
echo $fruits[2];   // cherry

// Modifying elements
$fruits[1] = "mango";
echo $fruits[1];   // mango

// Adding new elements
$fruits[] = "grape";  // Appends to the end
echo $fruits[3];      // grape
?>
```

---

### Associative Array

- Elements are stored with **custom string keys** (key => value pairs).
- Like a dictionary or object — access by name, not by position.

```php
<?php
$person = [
    "name"  => "Phyo",
    "age"   => 25,
    "city"  => "Yangon",
    "admin" => true,
];

// Accessing elements by key
echo $person["name"];   // Phyo
echo $person["age"];    // 25

// Modifying
$person["age"] = 26;

// Adding new key
$person["email"] = "phyo@example.com";

// Removing a key
unset($person["admin"]);
?>
```

---

### Multidimensional Array

- An array inside another array — for storing structured, nested data.

```php
<?php
$users = [
    ["name" => "Phyo",  "age" => 25],
    ["name" => "Alice", "age" => 30],
    ["name" => "Bob",   "age" => 22],
];

// Accessing nested values
echo $users[0]["name"];   // Phyo
echo $users[1]["age"];    // 30
echo $users[2]["name"];   // Bob
?>
```

---

### Useful Array Functions

```php
<?php
$numbers = [3, 1, 4, 1, 5, 9, 2];

// Information
echo count($numbers);         // 7 — number of elements

// Adding & removing
array_push($numbers, 6);      // Add to end
array_pop($numbers);          // Remove from end
array_unshift($numbers, 0);   // Add to beginning
array_shift($numbers);        // Remove from beginning

// Searching
echo in_array(5, $numbers);               // 1 (true) — check if value exists
echo array_search(4, $numbers);           // 2 — returns the key/index of the value

// Sorting
sort($numbers);           // Sort indexed array ascending (resets keys)
rsort($numbers);          // Sort indexed array descending
asort($numbers);          // Sort associative array by value (preserves keys)
ksort($numbers);          // Sort associative array by key

// Transforming
$doubled = array_map(fn($n) => $n * 2, $numbers);     // Apply function to each element
$evens   = array_filter($numbers, fn($n) => $n % 2 === 0); // Filter elements
$sum     = array_reduce($numbers, fn($carry, $n) => $carry + $n, 0); // Reduce to one value

// Slicing & splicing
$slice   = array_slice($numbers, 1, 3);   // Extract 3 elements starting at index 1
$merged  = array_merge([1, 2], [3, 4]);   // Merge two arrays → [1, 2, 3, 4]
$unique  = array_unique([1, 2, 2, 3, 3]); // Remove duplicates → [1, 2, 3]
$flipped = array_flip(["a" => 1, "b" => 2]); // Swap keys & values → [1 => "a", 2 => "b"]

// Converting
$str   = implode(", ", ["a", "b", "c"]);  // Array to string → "a, b, c"
$arr   = explode(", ", "a, b, c");        // String to array → ["a", "b", "c"]
?>
```

---

### Indexed vs Associative — Summary

| Feature | Indexed Array | Associative Array |
|---|---|---|
| Keys | Numbers (0, 1, 2...) | Custom strings (`"name"`, `"age"`) |
| Access | `$arr[0]` | `$arr["name"]` |
| Best for | Lists of items | Named properties / records |

---

## Object

- An **object** is an instance of a **class** — a blueprint that defines properties and methods.
- Objects are the foundation of **Object-Oriented Programming (OOP)** in PHP.

```php
<?php
class Car {
    public $make;
    public $model;

    public function __construct($make, $model) {
        $this->make  = $make;
        $this->model = $model;
    }

    public function describe() {
        return "{$this->make} {$this->model}";
    }
}

$myCar = new Car("Toyota", "Corolla");
echo $myCar->describe();    // Output: Toyota Corolla
echo $myCar->make;          // Output: Toyota
?>
```

> 💡 **Tip:** OOP and objects are a major topic of their own. This is just an introduction — we'll cover it in depth later.

---

## Resource

- A **resource** is a special variable that holds a **reference to an external resource** — like a file, database connection, or image handle.
- You don't create resources manually; they are returned by special PHP functions.
- Resources are automatically freed when no longer needed (garbage collected).

```php
<?php
// Opening a file returns a resource
$file = fopen("notes.txt", "r");  // $file is a Resource
var_dump($file);                  // resource(3) of type (stream)

fclose($file);  // Always close resources when done

// Database connection also returns a resource
$conn = mysqli_connect("localhost", "root", "", "mydb");  // Resource
?>
```

> ⚠️ **Warning:** Always close resources with `fclose()`, `mysqli_close()`, etc. when you're done. Leaving them open wastes server memory.

---

## Checking & Inspecting Types

- PHP gives you several functions to check what type a variable is.

```php
<?php
$name  = "Phyo";
$age   = 25;
$price = 9.99;
$flag  = true;
$empty = null;
$list  = [1, 2, 3];

// gettype() — returns the type name as a string
echo gettype($name);   // string
echo gettype($age);    // integer
echo gettype($price);  // double  ← PHP calls floats "double"
echo gettype($flag);   // boolean
echo gettype($empty);  // NULL
echo gettype($list);   // array

// var_dump() — shows type AND value (best for debugging)
var_dump($name);    // string(4) "Phyo"
var_dump($age);     // int(25)
var_dump($price);   // float(9.99)
var_dump($flag);    // bool(true)
var_dump($empty);   // NULL
var_dump($list);    // array(3) { [0]=> int(1) [1]=> int(2) [2]=> int(3) }

// print_r() — human-readable output (great for arrays)
print_r($list);
// Array ( [0] => 1 [1] => 2 [2] => 3 )

// Type-checking functions
var_dump(is_string($name));   // bool(true)
var_dump(is_int($age));       // bool(true)
var_dump(is_float($price));   // bool(true)
var_dump(is_bool($flag));     // bool(true)
var_dump(is_null($empty));    // bool(true)
var_dump(is_array($list));    // bool(true)
?>
```

> 💡 **Tip:** Use `var_dump()` when debugging — it shows both the **type** and **value**, which `echo` alone can't always tell you.

---

## Type Juggling & Type Casting

### Type Juggling (Automatic)

- PHP **automatically converts** types when needed based on the context.
- This is convenient but can cause surprising bugs.

```php
<?php
$a = "5" + 3;       // "5" (string) becomes 5 (int) → result: 8
$b = "5.5" + 1.5;   // "5.5" (string) becomes 5.5 (float) → result: 7.0
$c = "10 apples" + 5; // "10 apples" → 10 → result: 15 (ignores " apples")
$d = "apples" + 5;  // "apples" → 0 → result: 5

echo $a;  // 8
echo $b;  // 7
echo $c;  // 15
echo $d;  // 5
?>
```

> ⚠️ **Warning:** Type juggling can silently produce unexpected results. Use **strict comparisons** (`===`) and **explicit casting** to stay in control.

---

### Type Casting (Manual)

- You can **force** a value into a specific type using a cast.

```php
<?php
$val = "42.7abc";

$asInt    = (int)    $val;   // 42     — truncates decimal and non-numeric chars
$asFloat  = (float)  $val;   // 42.7   — truncates non-numeric chars
$asString = (string) 100;    // "100"
$asBool   = (bool)   0;      // false
$asArray  = (array)  "hello"; // ["hello"] — wraps in array

var_dump($asInt);    // int(42)
var_dump($asFloat);  // float(42.7)
var_dump($asString); // string(3) "100"
var_dump($asBool);   // bool(false)
var_dump($asArray);  // array(1) { [0]=> string(5) "hello" }

// Functions for casting
$num = intval("42abc");   // 42
$num = floatval("3.14x"); // 3.14
$str = strval(100);       // "100"
?>
```

---

### `==` vs `===` — Loose vs Strict Comparison

- This is one of the most important distinctions in PHP.

```php
<?php
// == (loose) — compares VALUE only, allows type juggling
var_dump(0    == false);  // bool(true) ← same value after conversion
var_dump(0    == null);   // bool(true) ← same value after conversion
var_dump(0    == "");     // bool(true) in PHP 7 / bool(false) in PHP 8
var_dump("1"  == 1);      // bool(true) ← string "1" converts to int 1
var_dump(100  == "1e2");  // bool(true) ← "1e2" is scientific notation for 100

// === (strict) — compares VALUE and TYPE — always prefer this
var_dump(0    === false);  // bool(false) ← different types
var_dump(0    === null);   // bool(false) ← different types
var_dump("1"  === 1);      // bool(false) ← different types
var_dump(1    === 1);      // bool(true)  ← same value, same type ✅
?>
```

> ⚠️ **Rule of Thumb:** Always use `===` (strict) unless you have a specific reason for loose comparison. `==` can hide bugs.

---

## Quick Revision

- PHP has **8 data types**: String, Integer, Float, Boolean, NULL, Array, Object, Resource.
- **String** — text in quotes. Single quotes are literal; double quotes parse variables. Use `.` to concatenate.
- **Integer** — whole numbers. Supports hex `0x`, octal `0`, binary `0b`, and underscores `1_000` for readability.
- **Float** — decimal numbers. **Never compare floats with `==`** — use `round()` or epsilon comparison instead.
- **Boolean** — `true` or `false`. Know your falsy values: `0`, `""`, `"0"`, `[]`, `null`, `false`.
- **NULL** — no value. A variable is NULL if unassigned, set to `null`, or `unset()`. Use `isset()` to check.
- **Array** — stores multiple values. Indexed arrays use number keys; associative arrays use string keys.
- **Object** — instance of a class. Core of OOP in PHP (covered in depth later).
- **Resource** — a handle to an external resource (file, DB). Always close resources when done.
- Use `var_dump()` for debugging — it shows both type and value.
- PHP is **loosely typed** — it auto-converts types (type juggling). Use `===` to avoid surprises.
- Use `(int)`, `(float)`, `(string)`, `(bool)`, `(array)` to **cast** values manually.
- **`==` is loose** (compares value only); **`===` is strict** (compares value AND type). Prefer `===`.