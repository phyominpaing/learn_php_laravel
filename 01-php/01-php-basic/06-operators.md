# PHP Operators

An **operator** is a symbol that tells PHP to perform a specific operation on one or more values (called **operands**). Operators are the building blocks of every expression — from simple math to complex conditional logic.

---

## Table of Contents

1. [Overview of Operator Types](#overview-of-operator-types)
2. [Arithmetic Operators](#arithmetic-operators)
3. [Assignment Operators](#assignment-operators)
4. [Incrementing & Decrementing Operators](#incrementing--decrementing-operators)
5. [Comparison Operators](#comparison-operators)
6. [Logical Operators](#logical-operators)
7. [String Operators](#string-operators)
8. [Array Operators](#array-operators)
9. [Null Coalescing & Null-Safe Operators](#null-coalescing--null-safe-operators)
10. [Ternary & Match Operators](#ternary--match-operators)
11. [Spaceship Operator (`<=>`)](#spaceship-operator-)
12. [Operator Precedence & Associativity](#operator-precedence--associativity)
13. [Quick Revision](#quick-revision)

---

## Overview of Operator Types

PHP groups operators into these main categories:

| Category | Symbols/Examples | Purpose |
|---|---|---|
| Arithmetic | `+ - * / % **` | Math calculations |
| Assignment | `= += -= *= /=` | Assign values to variables |
| Increment/Decrement | `++ --` | Increase/decrease by 1 |
| Comparison | `== != === !== < > <=>` | Compare two values |
| Logical | `&& \|\| ! and or xor` | Combine boolean conditions |
| String | `. .=` | Combine/append strings |
| Array | `+ == === != !==` | Combine and compare arrays |
| Null-related | `?? ??= ?->` | Handle NULL values safely |
| Conditional | `?: match` | Shorthand conditionals |

---

## Arithmetic Operators

- Used for **mathematical calculations** between numbers.
- If you use them on numeric strings, PHP automatically converts the strings to numbers (type juggling).

```php
<?php
$a = 10;
$b = 3;

echo $a + $b;   // 13   — Addition
echo $a - $b;   // 7    — Subtraction
echo $a * $b;   // 30   — Multiplication
echo $a / $b;   // 3.333... — Division (returns float if not exact)
echo $a % $b;   // 1    — Modulus (remainder after division)
echo $a ** $b;  // 1000 — Exponentiation (10 raised to the power of 3)
echo -$a;       // -10  — Negation (unary minus)
?>
```

### Arithmetic Operators Table

| Operator | Name | Example | Result |
|---|---|---|---|
| `+` | Addition | `10 + 3` | `13` |
| `-` | Subtraction | `10 - 3` | `7` |
| `*` | Multiplication | `10 * 3` | `30` |
| `/` | Division | `10 / 3` | `3.333...` |
| `%` | Modulus | `10 % 3` | `1` |
| `**` | Exponentiation | `10 ** 3` | `1000` |

> ⚠️ **Warning:** Division by zero (`10 / 0`) throws a `DivisionByZeroError` in PHP 8+. Always check the divisor before dividing.

```php
<?php
$divisor = 0;

if ($divisor !== 0) {
    echo 10 / $divisor;
} else {
    echo "Cannot divide by zero!";
}
?>
```

> 💡 **Tip:** `%` (modulus) is commonly used to check if a number is even or odd: `$num % 2 === 0` means **even**.

---

## Assignment Operators

- The basic assignment operator `=` assigns the value on the right to the variable on the left.
- **Combined assignment operators** perform an operation AND assign the result in one step — a shorthand.

```php
<?php
$x = 10;        // Basic assignment

$x += 5;   // Same as: $x = $x + 5;  → $x is now 15
$x -= 3;   // Same as: $x = $x - 3;  → $x is now 12
$x *= 2;   // Same as: $x = $x * 2;  → $x is now 24
$x /= 4;   // Same as: $x = $x / 4;  → $x is now 6
$x %= 4;   // Same as: $x = $x % 4;  → $x is now 2
$x **= 3;  // Same as: $x = $x ** 3; → $x is now 8

echo $x;   // Output: 8
?>
```

### Assignment Operators Table

| Operator | Equivalent To | Example (starts `$x = 10`) | Result |
|---|---|---|---|
| `=` | Assign | `$x = 10` | `10` |
| `+=` | `$x = $x + n` | `$x += 5` | `15` |
| `-=` | `$x = $x - n` | `$x -= 5` | `5` |
| `*=` | `$x = $x * n` | `$x *= 5` | `50` |
| `/=` | `$x = $x / n` | `$x /= 5` | `2` |
| `%=` | `$x = $x % n` | `$x %= 3` | `1` |
| `**=` | `$x = $x ** n` | `$x **= 2` | `100` |
| `.=` | String append | `$x .= "px"` | `"10px"` |
| `??=` | Assign if NULL | `$x ??= 5` | `10` (unchanged, already set) |

> 💡 **Tip:** Combined assignment operators make code shorter and clearer. `$total += $price;` reads naturally as "add price to total."

---

## Incrementing & Decrementing Operators

- `++` increases a variable's value by **1**.
- `--` decreases a variable's value by **1**.
- Can be placed **before** (pre) or **after** (post) the variable — the position matters!

```php
<?php
$a = 5;

echo $a++;  // Output: 5  (returns OLD value, THEN increments)
echo $a;    // Output: 6  (now incremented)

$b = 5;

echo ++$b;  // Output: 6  (increments FIRST, then returns new value)
echo $b;    // Output: 6  (already incremented)
?>
```

### Pre vs Post — Side by Side

```php
<?php
// Post-increment: use value, THEN increment
$x = 10;
$result = $x++;
echo $result;  // 10 (old value used)
echo $x;       // 11 (incremented after)

// Pre-increment: increment FIRST, THEN use value
$y = 10;
$result = ++$y;
echo $result;  // 11 (new value used)
echo $y;       // 11 (already incremented)
?>
```

### Increment/Decrement Operators Table

| Operator | Name | Effect |
|---|---|---|
| `++$a` | Pre-increment | Increments `$a` by 1, THEN returns the new value |
| `$a++` | Post-increment | Returns the current value of `$a`, THEN increments by 1 |
| `--$a` | Pre-decrement | Decrements `$a` by 1, THEN returns the new value |
| `$a--` | Post-decrement | Returns the current value of `$a`, THEN decrements by 1 |

> ⚠️ **Common Mistake:** Mixing up `$a++` and `++$a` inside expressions (like array indices or function calls) can produce off-by-one bugs. When in doubt, increment on a separate line.

```php
<?php
// Tricky example — be careful in loops!
$i = 0;
$arr = ["a", "b", "c"];

echo $arr[$i++];  // Output: a  (uses $i=0, THEN $i becomes 1)
echo $arr[$i++];  // Output: b  (uses $i=1, THEN $i becomes 2)
echo $arr[$i];    // Output: c  (uses $i=2)
?>
```

---

### Incrementing Special Types

- PHP allows `++` on **strings** too — it increments alphabetically.
- `++` and `--` have **no effect on `NULL`** or **booleans**.

```php
<?php
$letter = "a";
$letter++;
echo $letter;   // Output: b

$letter = "z";
$letter++;
echo $letter;   // Output: aa  (rolls over like a counter!)

$str = "Az";
$str++;
echo $str;      // Output: Ba

// ++ on NULL
$n = null;
$n++;
var_dump($n);   // int(1)  ← NULL becomes 1 when incremented

// -- on NULL has no effect
$m = null;
$m--;
var_dump($m);   // NULL    ← stays NULL when decremented
?>
```

> 💡 **Fun fact:** Incrementing strings like spreadsheet columns (`"z"++` → `"aa"`) is a unique PHP quirk — rarely used but good to know.

---

## Comparison Operators

- Used to **compare two values** and return a **boolean** (`true`/`false`).
- The most important distinction in PHP: **loose (`==`)** vs **strict (`===`)** comparison.

```php
<?php
$a = 5;
$b = "5";

var_dump($a == $b);   // bool(true)  — loose: only checks VALUE
var_dump($a === $b);  // bool(false) — strict: checks VALUE and TYPE

var_dump($a != $b);   // bool(false) — loose "not equal"
var_dump($a !== $b);  // bool(true)  — strict "not equal"

var_dump(5 > 3);    // bool(true)
var_dump(5 < 3);    // bool(false)
var_dump(5 >= 5);   // bool(true)
var_dump(5 <= 4);   // bool(false)
?>
```

### Comparison Operators Table

| Operator | Name | Example | Result |
|---|---|---|---|
| `==` | Equal (loose) | `5 == "5"` | `true` |
| `===` | Identical (strict) | `5 === "5"` | `false` |
| `!=` | Not equal (loose) | `5 != "5"` | `false` |
| `<>` | Not equal (alt syntax) | `5 <> "5"` | `false` |
| `!==` | Not identical (strict) | `5 !== "5"` | `true` |
| `<` | Less than | `3 < 5` | `true` |
| `>` | Greater than | `5 > 3` | `true` |
| `<=` | Less than or equal | `5 <= 5` | `true` |
| `>=` | Greater than or equal | `5 >= 6` | `false` |
| `<=>` | Spaceship (3-way compare) | `5 <=> 3` | `1` |

---

### Why `==` Can Be Dangerous

```php
<?php
// These all return true with == (loose comparison) — surprising!
var_dump(0 == "abc");     // bool(false) in PHP 8 / bool(true) in PHP 7 — behavior CHANGED!
var_dump("1" == "01");    // bool(true)  — both are numeric strings, compared as numbers
var_dump("10" == "1e1");  // bool(true)  — "1e1" is scientific notation for 10
var_dump(100 == "1e2");   // bool(true)  — same reason
var_dump(null == false);  // bool(true)  — both are "falsy"
var_dump(0 == false);     // bool(true)  — both are "falsy"
var_dump("" == null);     // bool(true)  — both are "falsy"

// === avoids ALL of these surprises
var_dump(0 === "abc");    // bool(false) — different types, end of story
var_dump("1" === "01");   // bool(false) — different strings
?>
```

> ⚠️ **PHP 8 Behavior Change:** In PHP 8, comparing a number to a non-numeric string (`0 == "abc"`) now returns `false` (it used to return `true` in PHP 7). Always know which PHP version your code runs on.

> ⚠️ **Golden Rule:** Always prefer `===` and `!==` over `==` and `!=` unless you have a very specific reason for loose comparison. This single habit prevents a huge category of bugs.

---

## Logical Operators

- Used to **combine multiple boolean conditions**.
- PHP has two sets of logical operators: **symbols** (`&& || !`) and **words** (`and or xor not`) — they work similarly but have **different precedence** (covered later).

```php
<?php
$age      = 25;
$hasID    = true;

// AND — both conditions must be true
if ($age >= 18 && $hasID) {
    echo "Can enter";   // ✅ This runs
}

// OR — at least one condition must be true
$isVIP = false;
$isMember = true;
if ($isVIP || $isMember) {
    echo "Skip the line";  // ✅ This runs
}

// NOT — flips true to false, and false to true
$isBanned = false;
if (!$isBanned) {
    echo "Welcome!";  // ✅ This runs
}

// XOR — true if EXACTLY ONE side is true (not both, not neither)
var_dump(true xor false);  // bool(true)
var_dump(true xor true);   // bool(false)
?>
```

### Logical Operators Table

| Operator | Name | Returns `true` when |
|---|---|---|
| `&&` or `and` | AND | Both sides are `true` |
| `\|\|` or `or` | OR | At least one side is `true` |
| `!` | NOT | The value is `false` (flips it) |
| `xor` | Exclusive OR | Exactly one side is `true` (not both) |

---

### Short-Circuit Evaluation

- PHP **stops evaluating** as soon as the result is determined — this is called **short-circuiting**.
- With `&&`, if the first condition is `false`, the second is never checked.
- With `||`, if the first condition is `true`, the second is never checked.

```php
<?php
function expensiveCheck() {
    echo "Running expensive check... ";
    return true;
}

$a = false;

// Short-circuit: expensiveCheck() is NEVER called because $a is already false
if ($a && expensiveCheck()) {
    echo "Both true";
}
echo "Done";
// Output: Done  (expensiveCheck() never printed anything!)

// Useful pattern: avoid errors by checking existence first
$user = null;
if ($user !== null && $user["name"] === "Phyo") {
    // Safe — $user["name"] is only checked if $user is not null
    echo "Hello";
}
?>
```

> 💡 **Tip:** Short-circuiting is useful for performance (skip expensive checks) and safety (avoid errors from accessing properties on `null`).

---

### `&&`/`||` vs `and`/`or` — Precedence Difference

```php
<?php
// This looks like it should work the same, but it doesn't!
$result = false || true;
var_dump($result);  // bool(true)  ✅

$result = false or true;
var_dump($result);  // bool(false) ❌ — surprising!

// Why? Because = has HIGHER precedence than 'or', but LOWER than '||'
// The second line is actually evaluated as: ($result = false) or true;
// $result becomes false FIRST, then "or true" is just discarded
?>
```

> ⚠️ **Warning:** Avoid `and`/`or`/`xor` in assignments due to this precedence trap. Stick to `&&` and `||` for combining conditions — they behave as expected.

---

## String Operators

- PHP has only **two** string operators: **concatenation** and **concatenation assignment**.

```php
<?php
$first = "Hello";
$last  = "World";

// . — Concatenation: joins two strings
$greeting = $first . ", " . $last . "!";
echo $greeting;   // Output: Hello, World!

// .= — Concatenation assignment: appends to existing string
$greeting = "Hello";
$greeting .= ", World!";
echo $greeting;   // Output: Hello, World!

// Concatenating different types (auto-converted to string)
$age = 25;
echo "I am " . $age . " years old.";  // Output: I am 25 years old.
?>
```

### String Operators Table

| Operator | Name | Example | Result |
|---|---|---|---|
| `.` | Concatenation | `"Hello" . " World"` | `"Hello World"` |
| `.=` | Concatenation assignment | `$s = "Hi"; $s .= "!"` | `"Hi!"` |

> ⚠️ **Common Mistake:** Don't confuse `.` (string concatenation) with `+` (arithmetic addition). Using `+` on strings tries to convert them to numbers!

```php
<?php
echo "5" . "5";   // Output: "55"  (string concatenation)
echo "5" + "5";   // Output: 10    (numeric addition — both converted to int)

echo "Hello" + "World";  // ❌ TypeError in PHP 8 — non-numeric strings can't add
?>
```

---

## Array Operators

- Special operators for **combining**, **comparing**, and checking **equality** of arrays.

```php
<?php
$a = ["a" => "red",   "b" => "green"];
$b = ["c" => "blue",  "b" => "yellow"];

// + — Union: combines arrays. Left array's keys take priority on duplicates.
$union = $a + $b;
print_r($union);
// Array ( [a] => red [b] => green [c] => blue )
// Note: "b" stays "green" because $a comes first

// == — Equal: true if same key/value pairs (order doesn't matter)
$x = ["a" => 1, "b" => 2];
$y = ["b" => 2, "a" => 1];
var_dump($x == $y);    // bool(true) — same keys & values, order ignored

// === — Identical: true if same key/value pairs in the SAME ORDER and SAME TYPES
var_dump($x === $y);   // bool(false) — different order

// != and !== — opposite of == and ===
var_dump($x != $y);    // bool(false)
var_dump($x !== $y);   // bool(true)
?>
```

### Array Operators Table

| Operator | Name | Result |
|---|---|---|
| `+` | Union | Combines arrays; left array's keys win on duplicates |
| `==` | Equal | `true` if same key/value pairs (any order) |
| `===` | Identical | `true` if same key/value pairs in same order with same types |
| `!=` / `<>` | Not equal | Opposite of `==` |
| `!==` | Not identical | Opposite of `===` |

> 💡 **Tip:** To merge arrays where **later values overwrite earlier ones** (the opposite of `+`), use `array_merge()` instead.

```php
<?php
$a = ["a" => "red",  "b" => "green"];
$b = ["b" => "blue", "c" => "yellow"];

print_r($a + $b);              // [a => red, b => green, c => yellow]  (left wins)
print_r(array_merge($a, $b));  // [a => red, b => blue, c => yellow]   (right wins)
?>
```

---

## Null Coalescing & Null-Safe Operators

- Modern PHP operators (7.0+ and 8.0+) for **safely handling NULL values** without verbose `isset()` checks.

```php
<?php
// ?? — Null Coalescing: returns left side if it's NOT null, otherwise right side
$username = $_GET["username"] ?? "Guest";
// Equivalent to:
// $username = isset($_GET["username"]) ? $_GET["username"] : "Guest";

echo $username;  // "Guest" if no ?username= in URL

// Chaining multiple fallbacks
$name = $_GET["name"] ?? $_POST["name"] ?? "Anonymous";

// ??= — Null Coalescing Assignment: assigns ONLY if variable is null
$config = null;
$config ??= "default-config";
echo $config;   // Output: default-config

$config ??= "another-value";
echo $config;   // Output: default-config  (unchanged — already not null)
```

```php
// ?-> — Nullsafe Operator (PHP 8.0+): safely access object properties/methods
// Returns null instead of throwing an error if the object is null

class Address {
    public $city = "Yangon";
}
class User {
    public $address = null;  // No address set
}

$user = new User();

// ❌ Without nullsafe — Fatal Error: Attempt to read property on null
// echo $user->address->city;

// ✅ With nullsafe — returns null safely, no error
echo $user->address?->city ?? "No city set";  // Output: No city set
?>
```

### Null-Related Operators Table

| Operator | Name | Behavior |
|---|---|---|
| `??` | Null coalescing | Returns left if not null, else right |
| `??=` | Null coalescing assignment | Assigns right only if left is null |
| `?->` | Nullsafe (PHP 8+) | Returns null instead of error if object is null |

---

## Ternary & Match Operators

### Ternary Operator `?:`

- A shorthand for a simple `if/else` that returns a value.
- Syntax: `condition ? value_if_true : value_if_false`

```php
<?php
$age = 20;

$status = ($age >= 18) ? "Adult" : "Minor";
echo $status;  // Output: Adult

// Shorthand ternary (Elvis operator) — returns left side if truthy, else right
$name = "";
$displayName = $name ?: "Anonymous";
echo $displayName;  // Output: Anonymous  ($name is empty/falsy)

// Nested ternary (use sparingly — gets hard to read!)
$score = 75;
$grade = ($score >= 90) ? "A" : (($score >= 70) ? "B" : "C");
echo $grade;  // Output: B
?>
```

> ⚠️ **Warning:** `?:` (Elvis operator) checks **truthiness**, while `??` checks for **null only**. `$name ?: "Anonymous"` would also trigger for `""`, `0`, or `false` — not just `null`.

---

### `match` Expression (PHP 8.0+)

- A modern, cleaner alternative to `switch` for returning values based on a condition.
- Uses **strict comparison** (`===`) — no type juggling.
- Must have an exhaustive match or an `default` arm, or it throws an `UnhandledMatchError`.

```php
<?php
$role = "editor";

$permission = match ($role) {
    "admin"  => "Full access",
    "editor" => "Can edit content",
    "viewer" => "Read-only access",
    default  => "No access",
};

echo $permission;  // Output: Can edit content

// Multiple conditions per arm
$score = 85;
$grade = match (true) {
    $score >= 90 => "A",
    $score >= 80 => "B",
    $score >= 70 => "C",
    default      => "F",
};
echo $grade;  // Output: B
?>
```

> 💡 **`match` vs `switch`:** `match` uses strict comparison (`===`), doesn't need `break`, and returns a value directly. Prefer `match` over `switch` in PHP 8+ for cleaner code.

---

## Spaceship Operator (`<=>`)

- Also called the **combined comparison operator** (PHP 7.0+).
- Compares two values and returns:
  - `-1` if left is **less than** right
  - `0` if they are **equal**
  - `1` if left is **greater than** right
- Extremely useful for **custom sorting**.

```php
<?php
echo 1 <=> 2;   // -1  (1 is less than 2)
echo 2 <=> 2;   //  0  (equal)
echo 3 <=> 2;   //  1  (3 is greater than 2)

echo "a" <=> "b";  // -1 (alphabetically "a" comes before "b")

// Real-world use: custom sort function
$numbers = [5, 2, 8, 1, 9];

usort($numbers, function ($a, $b) {
    return $a <=> $b;  // Ascending order
});
print_r($numbers);  // [1, 2, 5, 8, 9]

usort($numbers, function ($a, $b) {
    return $b <=> $a;  // Descending order
});
print_r($numbers);  // [9, 8, 5, 2, 1]
?>
```

---

## Operator Precedence & Associativity

- **Precedence** determines which operator is evaluated **first** when an expression has multiple operators.
- **Associativity** determines the **direction** (left-to-right or right-to-left) operators of the same precedence are evaluated.
- Just like in math: `*` and `/` happen before `+` and `-`.

```php
<?php
echo 2 + 3 * 4;     // Output: 14  (not 20) — * happens before +
echo (2 + 3) * 4;   // Output: 20  — parentheses force order

echo 10 - 2 - 3;    // Output: 5  — left-to-right: (10 - 2) - 3
echo 2 ** 3 ** 2;   // Output: 512 — right-to-left: 2 ** (3 ** 2) = 2 ** 9
?>
```

### Operator Precedence Table (Highest to Lowest)

| Precedence | Operators | Associativity |
|---|---|---|
| 1 (highest) | `clone` `new` | — |
| 2 | `**` | Right-to-left |
| 3 | `++` `--` `~` `(int)` `(float)` `(string)` `!` | Right-to-left (unary) |
| 4 | `instanceof` | Left-to-right |
| 5 | `*` `/` `%` | Left-to-right |
| 6 | `+` `-` | Left-to-right |
| 7 | `.` (string concat) | Left-to-right |
| 8 | `<<` `>>` (bitwise shift) | Left-to-right |
| 9 | `<` `<=` `>` `>=` | Left-to-right |
| 10 | `==` `!=` `===` `!==` `<>` `<=>` | Left-to-right |
| 11 | `&` (bitwise AND) | Left-to-right |
| 12 | `^` (bitwise XOR) | Left-to-right |
| 13 | `\|` (bitwise OR) | Left-to-right |
| 14 | `&&` | Left-to-right |
| 15 | `\|\|` | Left-to-right |
| 16 | `??` | Right-to-left |
| 17 | `? :` (ternary) | Left-to-right |
| 18 | `=` `+=` `-=` `*=` `/=` etc. | Right-to-left |
| 19 | `and` | Left-to-right |
| 20 | `xor` | Left-to-right |
| 21 (lowest) | `or` | Left-to-right |

> 💡 **Practical Tip:** You don't need to memorize this whole table. Just remember:
> 1. `**` (exponent) is highest among common operators.
> 2. `* / %` before `+ -`.
> 3. Comparison (`== === <`) before logical (`&& ||`).
> 4. `=` (assignment) is nearly the **lowest** — almost everything happens before assignment.
> 5. **When unsure, use parentheses `()`** — they always make intent clear and override default precedence.

```php
<?php
// Using parentheses for clarity — ALWAYS a good habit
$total = ($price * $quantity) + ($tax_rate * $price * $quantity);

// vs. relying on precedence (harder to read)
$total = $price * $quantity + $tax_rate * $price * $quantity;
?>
```

---

## Quick Revision

- **Arithmetic** (`+ - * / % **`): standard math. Watch out for `DivisionByZeroError` on `/0` in PHP 8+.
- **Assignment** (`= += -= *= /= %= **= .= ??=`): shorthand for "do operation, then assign."
- **Increment/Decrement** (`++ --`): pre (`++$a`) changes first then returns; post (`$a++`) returns first then changes. Works on strings too (`"z"++` → `"aa"`).
- **Comparison** (`== === != !== < > <= >= <=>`): always prefer `===`/`!==` (strict) over `==`/`!=` (loose) to avoid type-juggling surprises. PHP 8 changed `0 == "abc"` to `false`.
- **Logical** (`&& || ! xor` and `and or xor`): combine conditions. `&&`/`||` short-circuit. Avoid `and`/`or` in assignments due to precedence traps.
- **String** (`. .=`): only two operators — concatenation and append. Don't confuse `.` with `+`.
- **Array** (`+ == === != !==`): `+` is union (left wins on duplicate keys); use `array_merge()` if you want right side to win.
- **Null operators** (`?? ??= ?->`): `??` for fallback values, `??=` to assign only if null, `?->` (PHP 8+) to safely chain object properties.
- **Ternary `?:`** is shorthand `if/else`; the Elvis operator `?:` checks truthiness (not just null) — different from `??`.
- **`match` (PHP 8+)** is a cleaner, strict-comparison alternative to `switch` that returns a value directly.
- **Spaceship `<=>`** returns `-1`, `0`, or `1` — perfect for `usort()` custom sorting.
- **Precedence**: `**` > unary (`!`, `++`) > `* / %` > `+ -` > comparisons > `&&` > `||` > `??` > ternary > `=` > `and/or/xor`.
- **When unsure about precedence, just use `()`** — it's always clearer and removes all doubt.