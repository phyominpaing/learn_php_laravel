# PHP Number & Math Functions

PHP comes with a rich set of built-in functions for checking number types, performing calculations, generating random numbers, and formatting numeric output. This note covers the core functions you'll reach for constantly.

---

## Table of Contents

1. [Number Type Checking](#number-type-checking)
   - [`is_int()`](#is_int)
   - [`is_float()`](#is_float)
   - [`is_finite()`](#is_finite)
   - [`is_infinite()`](#is_infinite)
   - [`is_nan()`](#is_nan)
2. [Constants & Comparison Functions](#constants--comparison-functions)
   - [`pi()`](#pi)
   - [`min()` / `max()`](#min--max)
3. [Power & Roots](#power--roots)
   - [`sqrt()`](#sqrt)
   - [`pow()`](#pow)
4. [Rounding Functions](#rounding-functions)
   - [`round()`](#round)
   - [`ceil()`](#ceil)
   - [`floor()`](#floor)
5. [Random Numbers](#random-numbers)
   - [`rand()`](#rand)
   - [`mt_rand()`](#mt_rand)
   - [`random_int()`](#random_int)
6. [Formatting Numbers](#formatting-numbers)
   - [`number_format()`](#number_format)
7. [Other Useful Math Functions](#other-useful-math-functions)
8. [Type Checking + Casting Together](#type-checking--casting-together)
9. [Quick Revision](#quick-revision)

---

## Number Type Checking

PHP provides several functions to check exactly what kind of number a value is — important because PHP is loosely typed and numbers can silently become floats, strings, or special values like `NAN` and `INF`.

---

### `is_int()`

- Checks whether a value is an **integer** (whole number, no decimal).
- Also has an alias: `is_integer()` and `is_long()` — all three do the same thing.

```php
<?php
var_dump(is_int(42));        // bool(true)
var_dump(is_int(-10));       // bool(true)
var_dump(is_int(3.14));      // bool(false) — it's a float
var_dump(is_int("42"));      // bool(false) — it's a STRING, even though it looks numeric
var_dump(is_int(42.0));      // bool(false) — has a decimal point, so it's a float

// Practical use: validating input is a true integer
function validateAge($age) {
    if (!is_int($age)) {
        return "Age must be a whole number";
    }
    return "Valid age: $age";
}

echo validateAge(25);     // Output: Valid age: 25
echo validateAge(25.5);   // Output: Age must be a whole number
echo validateAge("25");   // Output: Age must be a whole number ← string, not int!
?>
```

> ⚠️ **Warning:** `is_int("42")` returns `false` because `"42"` is a **string**, even though it contains only digits. If you need to check whether a string *looks like* a number, use `is_numeric()` instead (covered below).

---

### `is_float()`

- Checks whether a value is a **float** (decimal number).
- Alias: `is_double()` — both do the same thing (PHP calls floats "doubles" internally).

```php
<?php
var_dump(is_float(3.14));     // bool(true)
var_dump(is_float(42.0));     // bool(true) — has decimal point, even if it's a whole number
var_dump(is_float(42));       // bool(false) — this is an integer
var_dump(is_float("3.14"));   // bool(false) — it's a string

// Practical use: detecting precision-sensitive values (like currency)
$price = 9.99;
if (is_float($price)) {
    echo "Price has decimal precision";
}
?>
```

---

### `is_finite()`

- Checks whether a number is a **finite** value (i.e., NOT infinite and NOT NAN).
- Useful after calculations that might produce extreme or undefined results.

```php
<?php
var_dump(is_finite(100));        // bool(true)
var_dump(is_finite(3.14));       // bool(true)
var_dump(is_finite(1 / 0.0));    // bool(false) — division by 0.0 produces INF (see warning below)
var_dump(is_finite(PHP_INT_MAX)); // bool(true)
?>
```

> 💡 **Note:** Dividing an **integer** by zero (`10 / 0`) throws a `DivisionByZeroError` in PHP 8+. But dividing a **float** by `0.0` produces `INF` (infinity) without an error — this is where `is_finite()` becomes useful.

---

### `is_infinite()`

- Checks whether a number is **infinite** (`INF` or `-INF`).
- Can result from operations that exceed the maximum representable float, or float division by zero.

```php
<?php
var_dump(is_infinite(INF));         // bool(true)
var_dump(is_infinite(-INF));        // bool(true)
var_dump(is_infinite(100));         // bool(false)

$result = 1 / 0.0;  // Float division by zero
var_dump(is_infinite($result));     // bool(true)
echo $result;                        // Output: INF

// PHP_FLOAT_MAX exceeded
$huge = PHP_FLOAT_MAX * 2;
var_dump(is_infinite($huge));       // bool(true)
?>
```

---

### `is_nan()`

- Checks whether a value is **NAN** ("Not a Number") — the result of an undefined or unrepresentable mathematical operation.
- `NAN` is a special float value; comparing it with `==` or `===` **never** returns true, even against itself!

```php
<?php
$result = sqrt(-1);          // Square root of a negative number — undefined in real numbers
var_dump(is_nan($result));   // bool(true)
echo $result;                // Output: NAN

// The famous NAN quirk
var_dump(NAN == NAN);    // bool(false)  ← always false!
var_dump(NAN === NAN);   // bool(false)  ← even strict comparison fails!

// The ONLY correct way to check for NAN
var_dump(is_nan(NAN));   // bool(true)  ✅
?>
```

> ⚠️ **Critical Warning:** `NAN` is **never equal to anything**, including itself. Never write `if ($value == NAN)` — it will always be `false`, even when `$value` actually IS `NAN`. Always use `is_nan($value)` to check.

---

### Type Checking Summary Table

| Function | Checks For | Example Returns `true` |
|---|---|---|
| `is_int()` | Integer (alias: `is_integer()`, `is_long()`) | `is_int(42)` |
| `is_float()` | Float (alias: `is_double()`) | `is_float(3.14)` |
| `is_numeric()` | Looks like a number (int, float, or numeric string) | `is_numeric("42")` |
| `is_finite()` | Not infinite, not NAN | `is_finite(100)` |
| `is_infinite()` | Is `INF` or `-INF` | `is_infinite(INF)` |
| `is_nan()` | Is `NAN` | `is_nan(sqrt(-1))` |

```php
<?php
// is_numeric() — bonus function, very commonly paired with the above
var_dump(is_numeric(42));      // bool(true)
var_dump(is_numeric(3.14));    // bool(true)
var_dump(is_numeric("42"));    // bool(true)  ← numeric STRINGS count!
var_dump(is_numeric("42abc")); // bool(false)
var_dump(is_numeric("abc"));   // bool(false)
?>
```

---

## Constants & Comparison Functions

### `pi()`

- Returns the value of **π (pi)** as a float.
- Equivalent to the constant `M_PI` — both give the same value.

```php
<?php
echo pi();      // Output: 3.1415926535898
echo M_PI;      // Output: 3.1415926535898  (same value, as a constant)

// Practical use: calculating circle area
function circleArea($radius) {
    return pi() * $radius ** 2;
}

echo circleArea(5);  // Output: 78.539816339745
?>
```

> 💡 **Tip:** `pi()` and `M_PI` are interchangeable. Use whichever reads better in context — `M_PI` is slightly more common in math-heavy code since it's a constant, not a function call.

---

### `min()` / `max()`

- `min()` returns the **smallest** value; `max()` returns the **largest**.
- Both accept either **multiple individual arguments** OR a **single array**.

```php
<?php
// Multiple arguments
echo min(5, 2, 8, 1, 9);   // Output: 1
echo max(5, 2, 8, 1, 9);   // Output: 9

// Single array argument
$numbers = [5, 2, 8, 1, 9];
echo min($numbers);   // Output: 1
echo max($numbers);   // Output: 9

// Works with strings too (alphabetical comparison)
echo min("banana", "apple", "cherry");  // Output: apple
echo max("banana", "apple", "cherry");  // Output: cherry

// Practical use: clamping a value within a range
function clamp($value, $minVal, $maxVal) {
    return max($minVal, min($value, $maxVal));
}

echo clamp(150, 0, 100);  // Output: 100  (too high, clamped down)
echo clamp(-20, 0, 100);  // Output: 0    (too low, clamped up)
echo clamp(50, 0, 100);   // Output: 50   (within range, unchanged)
?>
```

---

## Power & Roots

### `sqrt()`

- Returns the **square root** of a number.
- Returns `NAN` for negative numbers (square roots of negatives aren't real numbers).

```php
<?php
echo sqrt(16);    // Output: 4
echo sqrt(2);     // Output: 1.4142135623731
echo sqrt(0);     // Output: 0
echo sqrt(-4);    // Output: NAN  — not a real number

// Practical use: distance formula
function distance($x1, $y1, $x2, $y2) {
    return sqrt(($x2 - $x1) ** 2 + ($y2 - $y1) ** 2);
}

echo distance(0, 0, 3, 4);  // Output: 5  (classic 3-4-5 triangle)
?>
```

---

### `pow()`

- Raises a number to a given **power** (exponent).
- Equivalent to the `**` operator — both do the same thing; `pow()` is the function form.

```php
<?php
echo pow(2, 10);     // Output: 1024  (2 to the power of 10)
echo 2 ** 10;        // Output: 1024  (same result using the operator)

echo pow(5, 2);      // Output: 25    (5 squared)
echo pow(2, 0.5);    // Output: 1.4142135623731  (same as sqrt(2)!)
echo pow(8, 1/3);    // Output: 2  (cube root of 8)
echo pow(2, -2);     // Output: 0.25  (negative exponent = 1 / 2^2)

// Practical use: compound interest
function compoundInterest($principal, $rate, $years) {
    return $principal * pow(1 + $rate, $years);
}

echo round(compoundInterest(1000, 0.05, 10), 2);  // Output: 1628.89
?>
```

> 💡 **Tip:** `pow($base, $exp)` and `$base ** $exp` produce identical results. The `**` operator (introduced in PHP 5.6) is generally preferred in modern code for being more concise, but `pow()` is useful when the base/exponent come from variables in a function-call style or for readability in some contexts.

---

## Rounding Functions

### `round()`

- Rounds a float to the nearest whole number, or to a specified number of **decimal places**.
- Standard "round half up" behavior (0.5 rounds up).

```php
<?php
echo round(4.4);          // Output: 4
echo round(4.5);          // Output: 5   (rounds up at .5)
echo round(4.6);          // Output: 5

echo round(3.14159, 2);   // Output: 3.14   (2 decimal places)
echo round(3.14159, 0);   // Output: 3
echo round(1234.5678, -2); // Output: 1200  (negative precision rounds to nearest hundred!)

// Practical use: rounding currency
$price = 19.995;
echo round($price, 2);    // Output: 20    (note: not "20.00" — see number_format() below)
?>
```

> 💡 **Tip:** `round()` returns a number, not a formatted string — `round(20, 2)` displays as `20`, not `20.00`. If you need exactly 2 decimal places **displayed** (like for prices), use `number_format()` instead.

---

### `ceil()`

- **Ceiling** — always rounds **UP** to the nearest whole number, regardless of the decimal value.
- Returns a **float**, even though the result has no decimal part.

```php
<?php
echo ceil(4.1);     // Output: 5
echo ceil(4.9);     // Output: 5
echo ceil(4.0);     // Output: 4
echo ceil(-4.1);    // Output: -4   (rounds UP/toward positive infinity, even for negatives)

// Practical use: calculating pages needed
function pagesNeeded($totalItems, $itemsPerPage) {
    return ceil($totalItems / $itemsPerPage);
}

echo pagesNeeded(45, 10);  // Output: 5  (45 items / 10 per page = 4.5 → rounds up to 5 pages)
?>
```

---

### `floor()`

- **Floor** — always rounds **DOWN** to the nearest whole number, regardless of the decimal value.
- Also returns a **float**.

```php
<?php
echo floor(4.1);     // Output: 4
echo floor(4.9);     // Output: 4
echo floor(4.0);     // Output: 4
echo floor(-4.1);    // Output: -5   (rounds DOWN/toward negative infinity, even for negatives)

// Practical use: calculating complete groups
function completeBoxes($totalItems, $itemsPerBox) {
    return floor($totalItems / $itemsPerBox);
}

echo completeBoxes(45, 10);  // Output: 4  (4 complete boxes, with 5 items leftover)
?>
```

### `round()` vs `ceil()` vs `floor()` — Side by Side

```php
<?php
$num = 4.5;

echo round($num);   // 5  (rounds to nearest)
echo ceil($num);    // 5  (always rounds up)
echo floor($num);   // 4  (always rounds down)

echo "\n";
$num = 4.1;

echo round($num);   // 4
echo ceil($num);    // 5  ← different from round() here!
echo floor($num);   // 4
?>
```

| Function | Behavior | `4.1` → | `4.5` → | `4.9` → | `-4.1` → |
|---|---|---|---|---|---|
| `round()` | Nearest (0.5 rounds up) | `4` | `5` | `5` | `-4` |
| `ceil()` | Always up | `5` | `5` | `5` | `-4` |
| `floor()` | Always down | `4` | `4` | `4` | `-5` |

---

## Random Numbers

### `rand()`

- Generates a **pseudo-random integer**.
- With no arguments: returns a number between `0` and `getrandmax()`.
- With two arguments `rand($min, $max)`: returns a number **within that inclusive range**.

```php
<?php
echo rand();           // Output: some large random number, e.g. 1804289383
echo rand(1, 10);      // Output: random number between 1 and 10 (inclusive)
echo rand(1, 100);     // Output: random number between 1 and 100 (inclusive)

// Practical use: simulating a dice roll
function rollDice() {
    return rand(1, 6);
}

echo rollDice();  // Output: random number 1-6
?>
```

> 💡 **Note:** Since PHP 7.1, `rand()` is actually an alias that uses the same strong algorithm as `mt_rand()` internally. They now behave identically in modern PHP, but `mt_rand()` remains the conventional choice for clarity (see below).

---

### `mt_rand()`

- Stands for **"Mersenne Twister random"** — a faster, statistically better random number generator than the old `rand()`.
- Same usage as `rand()`: no arguments, or `mt_rand($min, $max)`.
- Since PHP 7.1, `rand()` and `mt_rand()` produce the same results — but `mt_rand()` is still the conventionally recommended name to use.

```php
<?php
echo mt_rand();          // Output: random large number
echo mt_rand(1, 10);     // Output: random number between 1 and 10
echo mt_rand(1, 100);    // Output: random number between 1 and 100

// Practical use: shuffling-like selection
$prizes = ["Gold", "Silver", "Bronze", "Nothing"];
$randomIndex = mt_rand(0, count($prizes) - 1);
echo $prizes[$randomIndex];  // Output: a random prize from the array
?>
```

> ⚠️ **Security Warning:** `rand()` and `mt_rand()` are **NOT cryptographically secure**. Never use them for passwords, security tokens, OTP codes, or anything security-sensitive. Use `random_int()` (below) instead for those cases.

---

### `random_int()`

- A **cryptographically secure** random integer generator (PHP 7.0+).
- Same syntax as `rand()`/`mt_rand()`: `random_int($min, $max)`.
- Slower than `mt_rand()`, but safe for security-sensitive purposes.

```php
<?php
echo random_int(1, 100);     // Output: cryptographically secure random number

// Practical use: generating a secure OTP (one-time password)
function generateOTP() {
    return random_int(100000, 999999);  // 6-digit OTP
}

echo generateOTP();  // Output: e.g. 482913
?>
```

### Random Functions — Summary

| Function | Cryptographically Secure? | Use For |
|---|---|---|
| `rand($min, $max)` | ❌ No | General randomness — games, simple shuffles |
| `mt_rand($min, $max)` | ❌ No | General randomness (preferred over `rand()` by convention) |
| `random_int($min, $max)` | ✅ Yes | Passwords, tokens, OTPs, security-sensitive randomness |

---

## Formatting Numbers

### `number_format()`

- Formats a number with **grouped thousands** and a fixed number of **decimal places** — ideal for displaying prices, statistics, and large numbers.
- Syntax: `number_format($number, $decimals, $decimalSeparator, $thousandsSeparator)`

```php
<?php
echo number_format(1234567.891);          // Output: 1,234,568   (rounds, no decimals by default)
echo number_format(1234567.891, 2);       // Output: 1,234,567.89
echo number_format(1000);                 // Output: 1,000
echo number_format(0.5, 2);               // Output: 0.50

// Custom separators — useful for international formatting
echo number_format(1234567.891, 2, ",", "."); // Output: 1.234.567,89  (European style)
echo number_format(1234567.891, 2, ".", " "); // Output: 1 234 567.89  (French style)

// Practical use: displaying a price
$price = 19.5;
echo "$" . number_format($price, 2);  // Output: $19.50

// Practical use: formatting large statistics
$views = 1500000;
echo number_format($views) . " views";  // Output: 1,500,000 views
?>
```

> 💡 **Tip:** `number_format()` is the right tool whenever you need to **display** a number to a user — it always returns a properly formatted **string**, unlike `round()` which returns a plain number that may drop trailing zeros.

```php
<?php
// The classic comparison
$price = 20;
echo round($price, 2);           // Output: 20      (no decimals shown)
echo number_format($price, 2);   // Output: 20.00   (always shows 2 decimals) ✅
?>
```

---

## Other Useful Math Functions

```php
<?php
echo abs(-15);          // Output: 15        — absolute value (removes negative sign)
echo abs(15);            // Output: 15

echo intdiv(10, 3);      // Output: 3         — integer division (discards remainder)
echo fmod(10, 3);        // Output: 1         — float modulus/remainder

echo log(M_E);           // Output: 1         — natural logarithm (base e)
echo log10(1000);        // Output: 3         — base-10 logarithm
echo exp(1);             // Output: 2.718...  — e raised to a power

echo sin(M_PI / 2);      // Output: 1         — sine (radians)
echo cos(0);             // Output: 1         — cosine (radians)
echo tan(0);             // Output: 0         — tangent (radians)

echo deg2rad(180);       // Output: 3.14159...  — degrees to radians
echo rad2deg(M_PI);      // Output: 180          — radians to degrees

echo array_sum([1, 2, 3, 4]);  // Output: 10  — sum of array values
echo array_product([1, 2, 3, 4]); // Output: 24 — product of array values
?>
```

> 💡 **Tip:** Trigonometric functions (`sin`, `cos`, `tan`) expect **radians**, not degrees. Use `deg2rad()` to convert if you're working with degree values (common in design/graphics contexts).

---

## Type Checking + Casting Together

- A practical pattern combining type checks with casting for safely processing user input (e.g., from forms or APIs).

```php
<?php
function safeDivide($a, $b) {
    // Ensure both inputs are numeric
    if (!is_numeric($a) || !is_numeric($b)) {
        return "Both values must be numeric";
    }

    // Cast to float to be safe
    $a = (float) $a;
    $b = (float) $b;

    if ($b === 0.0) {
        return "Cannot divide by zero";
    }

    $result = $a / $b;

    if (is_nan($result)) {
        return "Result is not a number";
    }

    if (is_infinite($result)) {
        return "Result is infinite";
    }

    return round($result, 4);
}

echo safeDivide(10, 3);      // Output: 3.3333
echo safeDivide(10, 0);      // Output: Cannot divide by zero
echo safeDivide("abc", 5);   // Output: Both values must be numeric
?>
```

---

## Quick Revision

- **`is_int()`** checks for a true integer — `"42"` (string) returns `false`. Use `is_numeric()` to check number-like values including numeric strings.
- **`is_float()`** checks for a decimal/float value, including whole numbers written with a decimal point like `42.0`.
- **`is_finite()` / `is_infinite()` / `is_nan()`** check special float states. `NAN` is **never equal to anything**, even itself — always use `is_nan()`, never `== NAN`.
- **`pi()`** returns π; same value as the constant `M_PI`.
- **`min()` / `max()`** work with multiple arguments OR a single array; useful for clamping values into a range.
- **`sqrt()`** returns the square root (or `NAN` for negatives); **`pow($base, $exp)`** is the function form of the `**` operator.
- **`round()`** rounds to the nearest value (optionally to N decimal places); **`ceil()`** always rounds up; **`floor()`** always rounds down.
- **`rand()` / `mt_rand()`** generate fast pseudo-random integers — **not secure**. **`random_int()`** is cryptographically secure — always use it for passwords, tokens, and OTPs.
- **`number_format()`** formats numbers as display-ready strings with thousands separators and a fixed decimal count — use this (not `round()`) when displaying prices or stats to users.
- Other handy functions: `abs()`, `intdiv()`, `fmod()`, `log()`, trig functions (`sin`, `cos`, `tan` — expect radians), and `deg2rad()`/`rad2deg()` for conversions.
- Combine `is_numeric()`, type casting, and `is_nan()`/`is_infinite()` checks for safe, defensive math operations on user input.