# PHP Type Casting

**Type casting** is the process of explicitly converting a value from one data type into another. PHP is a loosely typed language — it often converts types automatically (called **type juggling**). But when you need to be certain about what type a value is, you cast it manually. This gives you full control and prevents subtle bugs caused by PHP's automatic conversions.

---

## Table of Contents

1. [Type Juggling vs Type Casting](#type-juggling-vs-type-casting)
2. [How to Cast in PHP](#how-to-cast-in-php)
3. [(int) — Cast to Integer](#int--cast-to-integer)
4. [(bool) — Cast to Boolean](#bool--cast-to-boolean)
5. [(float) — Cast to Float](#float--cast-to-float)
6. [(string) — Cast to String](#string--cast-to-string)
7. [(array) — Cast to Array](#array--cast-to-array)
8. [(object) — Cast to Object](#object--cast-to-object)
9. [(unset) — Cast to NULL](#unset--cast-to-null)
10. [Casting Functions vs Cast Operators](#casting-functions-vs-cast-operators)
11. [Type Casting in Real Scenarios](#type-casting-in-real-scenarios)
12. [Quick Revision](#quick-revision)

---

## Type Juggling vs Type Casting

Before casting, understand the difference between PHP doing it automatically vs you doing it intentionally.

### Type Juggling (Automatic — PHP decides)

```php
<?php
$result = "5" + 3;   // PHP quietly converts "5" to int 5
echo $result;         // Output: 8
var_dump($result);    // int(8)

// PHP changed the type silently — you didn't ask for it
// This is TYPE JUGGLING
?>
```

### Type Casting (Manual — You decide)

```php
<?php
$value  = "5";
$result = (int)$value + 3;  // YOU explicitly convert "5" to int 5
echo $result;                // Output: 8
var_dump($result);           // int(8)

// You are in control — this is TYPE CASTING
?>
```

| | Type Juggling | Type Casting |
|---|---|---|
| Who decides | PHP automatically | You explicitly |
| When | At runtime based on context | When you write the cast |
| Predictability | Can surprise you | Always clear and explicit |
| Syntax | No extra syntax | `(int)`, `(string)`, etc. |

> 💡 **Golden Rule:** Whenever you are converting a type intentionally, always cast explicitly. Don't rely on PHP to guess — it might guess wrong, especially across PHP versions.

---

## How to Cast in PHP

- Write the **target type in parentheses** before the value or variable.
- Syntax: `(type) $variable`

```php
<?php
$value = "42";

$asInt    = (int)    $value;  // Convert to integer
$asFloat  = (float)  $value;  // Convert to float
$asBool   = (bool)   $value;  // Convert to boolean
$asString = (string) 42;      // Convert to string
$asArray  = (array)  $value;  // Convert to array
$asObject = (object) $value;  // Convert to object

// Each produces a different type:
var_dump($asInt);     // int(42)
var_dump($asFloat);   // float(42)
var_dump($asBool);    // bool(true)
var_dump($asString);  // string(2) "42"
var_dump($asArray);   // array(1) { [0]=> string(2) "42" }
var_dump($asObject);  // object(stdClass)#1
?>
```

### All PHP Cast Operators

| Cast | Alias | Converts To |
|---|---|---|
| `(int)` | `(integer)` | Integer |
| `(bool)` | `(boolean)` | Boolean |
| `(float)` | `(double)` / `(real)` | Float |
| `(string)` | — | String |
| `(array)` | — | Array |
| `(object)` | — | Object (`stdClass`) |
| `(unset)` | — | NULL (deprecated — removed in PHP 8) |

---

## (int) — Cast to Integer

- Converts a value to a **whole number** — drops any decimal part (does NOT round — it truncates).
- Alias: `(integer)`, `intval()` function.

```php
<?php
// From float — truncates (does NOT round!)
var_dump((int) 9.9);     // int(9)   ← not 10! truncated, not rounded
var_dump((int) 9.1);     // int(9)
var_dump((int) -9.9);    // int(-9)  ← negative also truncates toward zero

// From string — reads digits at the start, stops at first non-numeric char
var_dump((int) "42");         // int(42)
var_dump((int) "42.9");       // int(42)   ← ignores decimal part
var_dump((int) "42abc");      // int(42)   ← stops at "a"
var_dump((int) "abc42");      // int(0)    ← starts with letters → 0
var_dump((int) "  10  ");     // int(10)   ← leading spaces ignored
var_dump((int) "");           // int(0)    ← empty string → 0

// From boolean
var_dump((int) true);    // int(1)
var_dump((int) false);   // int(0)

// From null
var_dump((int) null);    // int(0)

// Hexadecimal, octal, binary — only base-10 strings are converted
var_dump((int) "0x1A");  // int(0)   ← hex string NOT auto-converted
var_dump((int) 0x1A);    // int(26)  ← hex LITERAL works fine

// Practical use: safe integer from URL parameter
$page = (int) ($_GET["page"] ?? 1);  // Always an integer, never a string
echo $page;  // 3 if URL was ?page=3, 1 if ?page=one or missing
?>
```

> ⚠️ **Critical Warning:** `(int)` **truncates**, it does NOT round. `(int)9.9` is `9`, not `10`. If you need rounding, use `(int)round($value)`.

```php
<?php
// ❌ Wrong when rounding is needed
echo (int) 9.9;           // 9  ← wrong if you wanted 10

// ✅ Correct when rounding is needed
echo (int) round(9.9);    // 10 ✅
echo (int) ceil(9.1);     // 10 ✅
echo (int) floor(9.9);    // 9  ✅
?>
```

---

## (bool) — Cast to Boolean

- Converts a value to either `true` or `false`.
- Alias: `(boolean)`.
- Knowing which values become `false` (falsy) vs `true` (truthy) is essential.

```php
<?php
// === Values that become FALSE ===
var_dump((bool) false);   // bool(false)
var_dump((bool) 0);       // bool(false)  ← integer zero
var_dump((bool) 0.0);     // bool(false)  ← float zero
var_dump((bool) "");      // bool(false)  ← empty string
var_dump((bool) "0");     // bool(false)  ← the STRING "0" (only this, not "false"!)
var_dump((bool) []);      // bool(false)  ← empty array
var_dump((bool) null);    // bool(false)  ← null

// === Everything else becomes TRUE ===
var_dump((bool) 1);         // bool(true)
var_dump((bool) -1);        // bool(true)  ← any non-zero integer, even negative
var_dump((bool) 0.1);       // bool(true)
var_dump((bool) "false");   // bool(true)  ← the STRING "false" is TRUTHY! (non-empty string)
var_dump((bool) "0.0");     // bool(true)  ← the STRING "0.0" is TRUTHY! (only "0" is falsy)
var_dump((bool) " ");       // bool(true)  ← a single space is TRUTHY! (non-empty)
var_dump((bool) [0]);       // bool(true)  ← array with one element (even 0) is TRUTHY
var_dump((bool) "00");      // bool(true)  ← only "0" is falsy, not "00"
?>
```

> ⚠️ **Classic PHP Traps:**
> - `(bool)"false"` is `true` — because `"false"` is a non-empty string. Only the boolean `false` is falsy.
> - `(bool)"0"` is `false` — but `(bool)"0.0"` is `true`. Only the exact string `"0"` is falsy, not `"0.0"` or `"00"`.
> - `(bool)-1` is `true` — any non-zero number (even negative) is truthy.
> - `(bool)[0]` is `true` — an array with one element (even if it's `0`) is truthy. Only an empty `[]` is falsy.

```php
<?php
// Practical use: convert a form checkbox value to a real boolean
$agreed = (bool) ($_POST["terms"] ?? false);

// Convert database string "1"/"0" to boolean
$isActive = (bool) $row["is_active"];  // "1" → true, "0" → false (since "0" is falsy)

// Convert config string to boolean
$debugMode = (bool) getenv("APP_DEBUG");  // "true" → true, "" → false
?>
```

---

## (float) — Cast to Float

- Converts a value to a **decimal number**.
- Aliases: `(double)`, `(real)` — all identical.
- Behaves similarly to `(int)` for strings — reads leading numeric characters.

```php
<?php
// From integer
var_dump((float) 5);        // float(5)

// From string
var_dump((float) "3.14");   // float(3.14)
var_dump((float) "3.14abc");// float(3.14) ← stops at "a"
var_dump((float) "abc");    // float(0)
var_dump((float) "1e3");    // float(1000) ← scientific notation works!
var_dump((float) "  2.5 "); // float(2.5)  ← leading/trailing spaces ignored

// From boolean
var_dump((float) true);     // float(1)
var_dump((float) false);    // float(0)

// From null
var_dump((float) null);     // float(0)

// Practical use: safe currency value from form input
$price = (float) ($_POST["price"] ?? 0.0);
echo number_format($price, 2);  // Formatted as "0.00" etc.
?>
```

> ⚠️ **Float precision warning:** Casting to float doesn't solve the binary precision problem (from the Data Types notes). `(float)"0.1" + (float)"0.2"` still doesn't equal exactly `0.3`. Use `round()` and `number_format()` when displaying currency.

---

## (string) — Cast to String

- Converts a value to its **string representation**.
- One of the most versatile casts — most types have a clear string form.

```php
<?php
// From integer and float
var_dump((string) 42);       // string(2) "42"
var_dump((string) 3.14);     // string(4) "3.14"
var_dump((string) 1.0e10);   // string(11) "10000000000"

// From boolean
var_dump((string) true);     // string(1) "1"   ← true becomes "1"
var_dump((string) false);    // string(0) ""    ← false becomes EMPTY STRING! (not "false" or "0")

// From null
var_dump((string) null);     // string(0) ""    ← null also becomes empty string!

// From array — you can't meaningfully cast an array to string
// var_dump((string) [1, 2, 3]); // ⚠️ Notice: Array to string conversion → "Array"

// Practical use: building a query string
$id    = 42;
$param = "page=" . (string) $id;  // "page=42"

// Practical use: format a float as a currency string
$price     = 19.9;
$formatted = (string) round($price, 2);  // "19.9" — still may lack trailing zero

// Better: use number_format for display
echo number_format($price, 2);  // "19.90"
?>
```

> ⚠️ **Critical Warning:** `(string)false` produces `""` (empty string), NOT `"false"`. And `(string)null` also produces `""`. This means if you cast a boolean or null to string and check for an empty string, you can't tell whether the original was `false`, `null`, or an actual empty string.

```php
<?php
// The trap
var_dump((string) false === "");  // bool(true)  — false → ""
var_dump((string) null === "");   // bool(true)  — null → ""

// Cannot tell them apart after casting to string!
// Always preserve the original type when the distinction matters
?>
```

---

## (array) — Cast to Array

- Converts a value into an **array**.
- Different types produce very different results — each case covered below.

```php
<?php
// From scalar (int, float, string, bool) — wraps in indexed array at [0]
var_dump((array) 42);       // array(1) { [0]=> int(42) }
var_dump((array) 3.14);     // array(1) { [0]=> float(3.14) }
var_dump((array) "hello");  // array(1) { [0]=> string(5) "hello" }
var_dump((array) true);     // array(1) { [0]=> bool(true) }

// From null — produces an EMPTY array (special case!)
var_dump((array) null);     // array(0) {}  ← NOT [null], just []

// From array — unchanged (no-op)
$arr = [1, 2, 3];
var_dump((array) $arr);     // array(3) { same array — not changed }

// From object — converts properties to key-value pairs
class User {
    public string $name = "Phyo";
    public int    $age  = 25;
}

$user  = new User();
$arr   = (array) $user;
print_r($arr);
// Array ( [name] => Phyo [age] => 25 )

// Private and protected properties get mangled key names
class Example {
    public    $pub  = "public";
    protected $prot = "protected";
    private   $priv = "private";
}

$arr = (array) new Example();
print_r($arr);
// Array (
//   [pub]                        => public
//   [*prot]                      => protected  ← key is "\0*\0prot"
//   [ExampleprIv]                => private    ← key is "\0Example\0priv"
// )
// The mangled keys contain null bytes — be careful when working with these

// Practical use: ensure a variable is always an array
function ensureArray($input): array {
    return is_array($input) ? $input : (array) $input;
}

print_r(ensureArray("hello"));   // ["hello"]
print_r(ensureArray([1, 2, 3])); // [1, 2, 3]  — unchanged
print_r(ensureArray(null));      // []  — empty array
?>
```

> 💡 **Tip:** The `(array) null === []` behavior is intentional and useful. It means you can safely do `(array) $possiblyNullConfig` and always get an array back — even if the source was null.

---

## (object) — Cast to Object

- Converts a value into an **`stdClass` object** — PHP's generic, anonymous object class.
- Each array key becomes a **property** of the object.

```php
<?php
// From associative array — keys become properties
$data = ["name" => "Phyo", "age" => 25, "city" => "Yangon"];
$obj  = (object) $data;

echo $obj->name;  // Output: Phyo
echo $obj->age;   // Output: 25
echo $obj->city;  // Output: Yangon

var_dump($obj);
// object(stdClass)#1 (3) {
//   ["name"]=> string(4) "Phyo"
//   ["age"] => int(25)
//   ["city"]=> string(6) "Yangon"
// }

// From indexed array — numeric keys become properties (accessed with curly braces)
$arr = [10, 20, 30];
$obj = (object) $arr;
echo $obj->{0};   // Output: 10
echo $obj->{1};   // Output: 20

// From scalar — wraps value in a "scalar" property
$obj = (object) "hello";
echo $obj->scalar;  // Output: hello

// From null — produces an empty stdClass object
$obj = (object) null;
var_dump($obj);  // object(stdClass)#1 (0) {}

// From object — no-op, stays the same object
class Car { public string $make = "Toyota"; }
$car = new Car();
$obj = (object) $car;  // Still a Car object, not converted to stdClass
echo $obj->make;  // Output: Toyota

// Practical use: converting JSON-decoded data (from object to array and back)
$json   = '{"name": "Phyo", "age": 25}';
$arr    = json_decode($json, true);     // Associative array
$obj    = (object) $arr;               // Convert to object

echo $obj->name;    // Output: Phyo
echo $arr["name"];  // Output: Phyo
?>
```

---

## (unset) — Cast to NULL

- Casts a value to `null`.
- **Deprecated in PHP 7.2** and **removed in PHP 8.0**.
- Included here because you may encounter it in legacy codebases.

```php
<?php
// ⚠️ DEPRECATED/REMOVED — do NOT use in PHP 7.2+ / 8+
$value = "Hello";
$null  = (unset) $value;   // null — but DON'T use this!
var_dump($null);            // NULL

// ✅ Modern alternatives:
$value = null;              // Direct assignment
// or
unset($value);              // Destroy the variable entirely
var_dump($value);           // Notice + NULL
?>
```

> ⚠️ **Important:** The `(unset)` cast does NOT destroy the variable — it just converts the value to null. The `unset()` function (without the cast syntax) actually destroys the variable. These are two different things. Since `(unset)` cast was removed in PHP 8, always use `= null` or `unset()` instead.

---

## Casting Functions vs Cast Operators

PHP also provides **functions** that do the same job as cast operators — sometimes more flexibly.

```php
<?php
$value = "42.9abc";

// Cast operators — direct and concise
echo (int)   $value;    // 42
echo (float) $value;    // 42.9
echo (bool)  $value;    // true (outputs 1)

// Equivalent functions
echo intval($value);         // 42
echo floatval($value);       // 42.9
echo boolval($value);        // true (outputs 1)
echo strval(42);             // "42"

// intval() with base — extra capability not available in the cast operator
echo intval("1A", 16);  // 26  ← parse hexadecimal string
echo intval("10", 2);   // 2   ← parse binary string
echo intval("17", 8);   // 15  ← parse octal string
?>
```

### Cast Operator vs Function — When to Use Which

| Situation | Prefer |
|---|---|
| Simple, straightforward conversion | `(int)`, `(float)`, etc. — shorter and cleaner |
| Need to specify a base (hex, binary, octal) | `intval($val, $base)` |
| Passing a converter as a callback | `"intval"` (function name as string) |
| Reading code aloud / extra clarity | Either — both are readable |

> 💡 **Common convention:** In modern PHP code, cast operators `(int)`, `(float)`, `(string)`, `(bool)` are more common for simple conversions. Functions like `intval()` are used when you need the extra `$base` argument or when passing as a callable.

---

## Type Casting in Real Scenarios

Here are the most common real-world situations where type casting is essential:

### Scenario 1 — Sanitizing User Input

```php
<?php
// URL: shop.php?product_id=5&qty=3&discount=0.1

$productId = (int)   ($_GET["product_id"] ?? 0);   // "5"    → 5
$qty       = (int)   ($_GET["qty"]        ?? 1);   // "3"    → 3
$discount  = (float) ($_GET["discount"]   ?? 0.0); // "0.1"  → 0.1

// Now you know exactly what types you're working with
if ($productId <= 0) {
    die("Invalid product ID");
}

$basePrice = 19.99;
$total     = $basePrice * $qty * (1 - $discount);
echo "Total: $" . number_format($total, 2);
// Output: Total: $53.97
?>
```

---

### Scenario 2 — Working with Database Results

```php
<?php
// Database rows come back as STRINGS by default in many drivers
$row = [
    "id"         => "1",       // Strings from PDO!
    "username"   => "Phyo",
    "age"        => "25",
    "is_active"  => "1",
    "balance"    => "99.50",
    "created_at" => "2024-01-15",
];

// Cast to correct types for use in logic
$id       = (int)    $row["id"];
$age      = (int)    $row["age"];
$isActive = (bool)   $row["is_active"];   // "1" → true,  "0" → false
$balance  = (float)  $row["balance"];

// Now safe to use in comparisons and calculations
if ($isActive && $age >= 18) {
    $fee       = $balance * 0.02;
    $newBalance = $balance - $fee;
    echo number_format($newBalance, 2);
}
?>
```

---

### Scenario 3 — JSON Output

```php
<?php
// Ensure correct types in JSON output — matters for API consumers
$userData = [
    "id"       => (int)   $row["id"],       // int, not string "1"
    "username" => (string)$row["username"],
    "age"      => (int)   $row["age"],
    "premium"  => (bool)  $row["is_premium"], // true/false, not "1"/"0"
    "balance"  => (float) $row["balance"],
];

header("Content-Type: application/json");
echo json_encode($userData);
// Output: {"id":1,"username":"Phyo","age":25,"premium":true,"balance":99.5}
// Without casting: {"id":"1","username":"Phyo","age":"25","premium":"1","balance":"99.50"}
// The difference matters a LOT for API clients!
?>
```

---

### Scenario 4 — Boolean Flags

```php
<?php
$config = [
    "debug"       => "true",    // String from config file
    "maintenance" => "0",       // String "0"
    "cache"       => "",        // Empty string
];

// ⚠️ Wrong — (bool)"true" is TRUE, but so is (bool)"false"!
$debug = (bool) $config["debug"];   // true  ← correct for "true"
$maint = (bool) $config["maintenance"]; // FALSE ← "0" is falsy — correct here

// ✅ Explicit string-to-bool conversion when values are "true"/"false" strings
function toBool(string $val): bool {
    return in_array(strtolower(trim($val)), ["1", "true", "yes", "on"], true);
}

var_dump(toBool("true"));    // bool(true)
var_dump(toBool("false"));   // bool(false)
var_dump(toBool("yes"));     // bool(true)
var_dump(toBool("0"));       // bool(false)
var_dump(toBool(""));        // bool(false)
?>
```

---

### Complete Type Casting Cheat Sheet

```php
<?php
// Starting values
$str    = "42.9abc";
$int    = 42;
$float  = 3.14;
$bool   = true;
$null   = null;
$arr    = ["a", "b"];

// Casting to INT
echo (int) $str;    // 42    (reads leading digits, stops at ".")
echo (int) $float;  // 3     (truncates, does not round)
echo (int) $bool;   // 1
echo (int) $null;   // 0

// Casting to FLOAT
echo (float) $str;  // 42.9  (reads up to first non-numeric except "." and "e")
echo (float) $int;  // 42
echo (float) $bool; // 1
echo (float) $null; // 0

// Casting to STRING
echo (string) $int;     // "42"
echo (string) $float;   // "3.14"
echo (string) true;     // "1"
echo (string) false;    // ""    ← EMPTY STRING, not "false"!
echo (string) $null;    // ""    ← EMPTY STRING

// Casting to BOOL
var_dump((bool) "0");       // false ← only "0" among non-empty strings
var_dump((bool) "");        // false
var_dump((bool) []);        // false
var_dump((bool) 0.0);       // false
var_dump((bool) null);      // false
var_dump((bool) "false");   // true  ← non-empty string!
var_dump((bool) [0]);       // true  ← non-empty array!
var_dump((bool) "0.0");     // true  ← not "0"!

// Casting to ARRAY
var_dump((array) "hello");  // ["hello"]
var_dump((array) 42);       // [42]
var_dump((array) null);     // []      ← empty array!
var_dump((array) $arr);     // ["a","b"]  ← unchanged

// Casting to OBJECT
$obj = (object) ["x" => 1, "y" => 2];
echo $obj->x;   // 1
echo $obj->y;   // 2
?>
```

---

## Quick Revision

- **Type casting** is explicitly converting a value to a specific type using `(type)` syntax — you are in control, unlike type juggling where PHP guesses.
- **`(int)`** — truncates (doesn't round!), strings read leading digits, `"abc"` → `0`, `true` → `1`, `null` → `0`. Use `(int)round()` when rounding matters.
- **`(bool)`** — falsy values: `false`, `0`, `0.0`, `""`, `"0"`, `[]`, `null`. Everything else is truthy — including `"false"`, `"0.0"`, `-1`, and `[0]`.
- **`(float)`** — like `(int)` for strings but keeps the decimal part; `"1e3"` → `1000.0` (scientific notation works).
- **`(string)`** — `true` → `"1"`, `false` → `""` (empty!), `null` → `""` (empty!). Never cast arrays to string — use `implode()`.
- **`(array)`** — scalars are wrapped in `[value]`; `null` → `[]` (empty, NOT `[null]`); objects become property-keyed arrays (private/protected keys get mangled null-byte names).
- **`(object)`** — converts arrays to `stdClass` objects; associative keys become property names; numeric keys need `$obj->{0}` syntax.
- **`(unset)`** — deprecated PHP 7.2, removed PHP 8.0. Use `= null` or `unset()` instead.
- Use **`intval($val, $base)`** when parsing hex/binary/octal strings — the only case where the function beats the cast operator.
- **Three real-world must-cast scenarios:** (1) sanitizing `$_GET`/`$_POST` input, (2) fixing string-typed database results, (3) ensuring correct JSON output types for API consumers.
- **`toBool()` helper** — when values are the strings `"true"`/`"false"` from config files, `(bool)` alone is not enough; build a helper that checks against `["1","true","yes","on"]`.