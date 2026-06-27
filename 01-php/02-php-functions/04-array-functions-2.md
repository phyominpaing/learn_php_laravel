# PHP Array Functions — Part 2

This note continues from Part 1 and covers the more **functional-style** array functions that let you inspect, transform, and filter arrays without manual loops — plus essential JSON encoding and decoding.

---

## Table of Contents

1. [Inspecting Keys & Values](#inspecting-keys--values)
   - [`array_keys()`](#array_keys)
   - [`array_values()`](#array_values)
2. [Searching by Value or Key](#searching-by-value-or-key)
   - [`array_search()`](#array_search)
   - [`array_key_exists()`](#array_key_exists)
3. [Transforming Arrays (Functional Style)](#transforming-arrays-functional-style)
   - [`array_map()`](#array_map)
   - [`array_filter()`](#array_filter)
   - [`array_reduce()`](#array_reduce)
4. [Sorting Arrays](#sorting-arrays)
   - [`sort()` / `rsort()`](#sort--rsort)
   - [`asort()` / `arsort()`](#asort--arsort)
   - [`ksort()` / `krsort()`](#ksort--krsort)
   - [`usort()` / `uasort()` / `uksort()`](#usort--uasort--uksort)
5. [JSON — Encoding & Decoding](#json--encoding--decoding)
   - [`json_encode()`](#json_encode)
   - [`json_decode()`](#json_decode)
6. [Bonus — Other Frequently Used Functions](#bonus--other-frequently-used-functions)
7. [Quick Reference Table](#quick-reference-table)
8. [Quick Revision](#quick-revision)

---

## Inspecting Keys & Values

### `array_keys()`

- Returns a **new array containing all the keys** (both numeric indexes and string keys) of an array.
- Can optionally filter — return only keys whose value matches a given value.

```php
<?php
// Basic use — get all keys
$person = ["name" => "Phyo", "age" => 25, "city" => "Yangon"];

print_r(array_keys($person));
// Array ( [0] => name [1] => age [2] => city )

// Indexed array — returns numeric keys
$fruits = ["apple", "banana", "cherry"];
print_r(array_keys($fruits));
// Array ( [0] => 0 [1] => 1 [2] => 2 )

// Optional: find keys that match a specific VALUE
$scores = ["Phyo" => 90, "Alice" => 75, "Bob" => 90];
print_r(array_keys($scores, 90));
// Array ( [0] => Phyo [1] => Bob )  ← both entries that have value 90

// Strict mode — third argument true
$data = [1, "1", true, 1.0];
print_r(array_keys($data, 1, true));   // Strict: only exact int 1
// Array ( [0] => 0 )  ← only index 0 (the actual integer 1)

// Practical use: validating required fields in form data
$form = ["name" => "Phyo", "email" => "phyo@example.com"];
$required = ["name", "email", "password"];
$missing = array_diff($required, array_keys($form));

if (!empty($missing)) {
    echo "Missing fields: " . implode(", ", $missing);
}
// Output: Missing fields: password
?>
```

---

### `array_values()`

- Returns a **new array containing all the values**, with **re-indexed numeric keys** starting from `0`.
- Extremely useful after operations that leave gaps in numeric keys (like `unset()` or `array_unique()`).

```php
<?php
// Basic use
$person = ["name" => "Phyo", "age" => 25, "city" => "Yangon"];

print_r(array_values($person));
// Array ( [0] => Phyo [1] => 25 [2] => Yangon )

// Re-indexing after gaps — the main practical use
$letters = ["a", "b", "c", "d", "e"];
unset($letters[1]);
unset($letters[3]);

print_r($letters);
// Array ( [0] => a [2] => c [4] => e )  ← gaps at index 1 and 3!

print_r(array_values($letters));
// Array ( [0] => a [1] => c [2] => e )  ← clean sequential keys

// Pair with array_unique() to re-index after removing duplicates
$names = ["Phyo", "Alice", "Phyo", "Bob"];
$unique = array_values(array_unique($names));
print_r($unique);
// Array ( [0] => Phyo [1] => Alice [2] => Bob )
?>
```

---

### `array_keys()` vs `array_values()` at a Glance

| Function | What it Returns | Keys of Result |
|---|---|---|
| `array_keys()` | All the **keys** of the input array | Always 0, 1, 2… |
| `array_values()` | All the **values** of the input array | Always 0, 1, 2… |

```php
<?php
$data = ["x" => "alpha", "y" => "beta", "z" => "gamma"];

print_r(array_keys($data));    // ["x", "y", "z"]
print_r(array_values($data));  // ["alpha", "beta", "gamma"]
?>
```

---

## Searching by Value or Key

### `array_search()`

- Searches for a **value** in an array and returns the **key** of the first match.
- Returns `false` if not found.
- Just like `strpos()` — use `=== false` to check "not found", because a key of `0` is falsy!

```php
<?php
$fruits = ["apple", "banana", "cherry"];

$key = array_search("banana", $fruits);
echo $key;  // Output: 1  (index 1)

// Use === false to correctly detect "not found"
$key = array_search("mango", $fruits);
if ($key === false) {
    echo "Not found";  // Output: Not found
}

// Associative array — returns the string key
$person = ["name" => "Phyo", "city" => "Yangon"];
echo array_search("Yangon", $person);  // Output: city

// Strict mode — third argument true (recommended)
$values = [0, 1, 2, "1"];
echo array_search("1", $values);         // Output: 1  ← loose match: "1" == 1
echo array_search("1", $values, true);   // Output: 3  ← strict match: "1" === "1"

// Practical use: find the index of an item to remove it
$users = ["Alice", "Bob", "Charlie"];
$key = array_search("Bob", $users, true);

if ($key !== false) {
    unset($users[$key]);
}
print_r(array_values($users));
// Array ( [0] => Alice [1] => Charlie )
?>
```

> ⚠️ **Warning:** Always use `=== false` when checking the return value of `array_search()`. If the value is found at key `0` (the first element), the function returns `0`, which is falsy — a simple `if (!$key)` check would incorrectly treat it as "not found."

---

### `array_key_exists()`

- Checks whether a **key** (or index) exists in an array.
- Returns `true` even if the value at that key is `null`.
- This is the key difference from `isset()` — `isset($array["key"])` returns `false` if the value is `null`; `array_key_exists()` returns `true`.

```php
<?php
$person = ["name" => "Phyo", "age" => null, "city" => "Yangon"];

var_dump(array_key_exists("name", $person));   // bool(true)
var_dump(array_key_exists("email", $person));  // bool(false) — key doesn't exist

// THE KEY DIFFERENCE: null values
var_dump(array_key_exists("age", $person));    // bool(true) ← key EXISTS, value is just null
var_dump(isset($person["age"]));               // bool(false) ← null fools isset()!

// Indexed arrays — check by numeric key
$colors = ["red", "green", "blue"];
var_dump(array_key_exists(0, $colors));   // bool(true)
var_dump(array_key_exists(5, $colors));   // bool(false)

// Practical use: safe default value pattern
function getConfig(array $config, string $key, $default = null) {
    if (array_key_exists($key, $config)) {
        return $config[$key];  // Returns even if value is null — intentional
    }
    return $default;
}

$config = ["debug" => null, "version" => "1.0"];
var_dump(getConfig($config, "debug"));    // NULL   ← found, value was null
var_dump(getConfig($config, "missing"));  // NULL   ← not found, default returned
?>
```

### `array_key_exists()` vs `isset()` — Critical Difference

| Check | Key missing | Key exists, value is `null` | Key exists, value is set |
|---|---|---|---|
| `array_key_exists()` | `false` | ✅ `true` | ✅ `true` |
| `isset()` | `false` | ❌ `false` | ✅ `true` |

> 💡 **Tip:** Use `array_key_exists()` when you need to distinguish "this key was never set" from "this key was explicitly set to `null`." This matters a lot in configuration arrays where `null` can be a meaningful, intentional value.

---

## Transforming Arrays (Functional Style)

These three functions — `array_map()`, `array_filter()`, and `array_reduce()` — are the PHP equivalents of the functional programming trio from the JavaScript notes. They let you transform arrays without writing explicit `foreach` loops.

---

### `array_map()`

- Applies a **callback function** to every element of an array and returns a **new array** of the results.
- The original array is **not modified**.
- Can operate on **multiple arrays** simultaneously (covered below).

```php
<?php
// Basic — double every number
$numbers = [1, 2, 3, 4, 5];
$doubled = array_map(fn($n) => $n * 2, $numbers);

print_r($doubled);   // [2, 4, 6, 8, 10]
print_r($numbers);   // [1, 2, 3, 4, 5]  ← original unchanged ✅

// Uppercase every string
$names = ["phyo", "alice", "bob"];
$upper = array_map("strtoupper", $names);  // Pass a built-in function name as a string
print_r($upper);  // ["PHYO", "ALICE", "BOB"]

// Transform associative array values (keys preserved)
$prices = ["shirt" => 10.0, "pants" => 20.0, "shoes" => 35.0];
$withTax = array_map(fn($p) => round($p * 1.1, 2), $prices);
print_r($withTax);
// Array ( [shirt] => 11 [pants] => 22 [shoes] => 38.5 )

// Operating on MULTIPLE arrays at once
$a = [1, 2, 3];
$b = [10, 20, 30];
$sums = array_map(fn($x, $y) => $x + $y, $a, $b);
print_r($sums);  // [11, 22, 33]
?>
```

> 💡 **Key point:** `array_map()` **always returns an array of the same length** as the input. If you need to reduce the count (keep only some items), use `array_filter()` instead.

---

### `array_filter()`

- Passes every element through a **callback** and keeps only elements where the callback returns `true`.
- Returns a **new array**; the original is **not modified**.
- ⚠️ **Preserves the original keys** — the result may have non-sequential numeric keys. Pair with `array_values()` if you need clean sequential keys.

```php
<?php
// Keep only even numbers
$numbers = [1, 2, 3, 4, 5, 6];
$evens = array_filter($numbers, fn($n) => $n % 2 === 0);

print_r($evens);
// Array ( [1] => 2 [3] => 4 [5] => 6 )  ← keys 1, 3, 5 preserved!

// Re-index to get clean sequential keys
print_r(array_values($evens));
// Array ( [0] => 2 [1] => 4 [2] => 6 )  ✅

// Filter strings
$names = ["Alice", "", "Bob", " ", "Charlie", null];
$nonEmpty = array_filter($names, fn($n) => !empty(trim((string) $n)));
print_r(array_values($nonEmpty));
// Array ( [0] => Alice [1] => Bob [2] => Charlie )

// No callback — removes all "falsy" values (0, "", null, false, [])
$mixed = [1, 0, "hello", "", null, false, true, [], [1, 2]];
print_r(array_values(array_filter($mixed)));
// Array ( [0] => 1 [1] => hello [2] => true [3] => Array([0]=>1 [1]=>2) )

// Practical use: filter active users
$users = [
    ["name" => "Phyo",  "active" => true],
    ["name" => "Alice", "active" => false],
    ["name" => "Bob",   "active" => true],
];

$activeUsers = array_filter($users, fn($u) => $u["active"] === true);
print_r(array_values($activeUsers));
// Array ( [0]=>["name"=>"Phyo","active"=>1] [1]=>["name"=>"Bob","active"=>1] )
?>
```

> 💡 **Tip:** `array_filter()` without a callback is a handy one-liner to remove all falsy values (`0`, `false`, `null`, `""`, `[]`) from an array — like a quick clean-up pass.

---

### `array_reduce()`

- **Folds** an array down into a **single value** by applying a callback to each element, carrying an accumulator between iterations.
- Syntax: `array_reduce($array, $callback, $initial)`
- The callback receives `($carry, $item)` — `$carry` is the running accumulated result.

```php
<?php
$numbers = [1, 2, 3, 4, 5];

// Sum
$sum = array_reduce($numbers, fn($carry, $item) => $carry + $item, 0);
echo $sum;  // Output: 15

// Product (multiply all together)
$product = array_reduce($numbers, fn($carry, $item) => $carry * $item, 1);
echo $product;  // Output: 120

// Find the largest value manually
$max = array_reduce($numbers, fn($carry, $item) => max($carry, $item), 0);
echo $max;  // Output: 5

// Build a string from an array
$words = ["Hello", "World", "from", "PHP"];
$sentence = array_reduce($words, fn($carry, $word) => $carry . $word . " ", "");
echo trim($sentence);  // Output: Hello World from PHP

// Practical use: calculate a cart total with tax
$cart = [
    ["name" => "Shirt", "price" => 10.00, "qty" => 2],
    ["name" => "Pants", "price" => 25.00, "qty" => 1],
    ["name" => "Shoes", "price" => 35.00, "qty" => 1],
];

$total = array_reduce($cart, fn($carry, $item) => $carry + ($item["price"] * $item["qty"]), 0.0);
echo number_format($total, 2);  // Output: 80.00
?>
```

> 💡 **Mental model:** Think of `array_reduce()` like a snowball rolling down a hill. It starts with the initial value (`$carry`), picks up each element (`$item`) one by one, grows the accumulated result, and returns the final snowball at the end.

---

### `array_map()` vs `array_filter()` vs `array_reduce()` — Summary

| Function | What it does | Returns | Length of Result |
|---|---|---|---|
| `array_map()` | Transforms every element | New array | Same length as input |
| `array_filter()` | Keeps elements matching a condition | New array (gaps in keys!) | ≤ input length |
| `array_reduce()` | Folds everything into one value | A single value (any type) | Not an array |

```php
<?php
$numbers = [1, 2, 3, 4, 5, 6];

// map: transform all → same count
$doubled = array_map(fn($n) => $n * 2, $numbers);
// [2, 4, 6, 8, 10, 12]  — still 6 elements

// filter: keep some → fewer elements
$evens = array_filter($numbers, fn($n) => $n % 2 === 0);
// [2, 4, 6]  — only 3 elements

// reduce: collapse all → one value
$sum = array_reduce($numbers, fn($carry, $n) => $carry + $n, 0);
// 21  — just a number, not an array
?>
```

---

## Sorting Arrays

Sorting is a huge part of array work. PHP has separate functions depending on whether you need to preserve keys and whether you're sorting by value or by key.

---

### `sort()` / `rsort()`

- `sort()` — sorts a **numeric-indexed array ascending** (A→Z, 0→9). Re-indexes keys.
- `rsort()` — sorts **descending** (Z→A, 9→0). Re-indexes keys.
- ⚠️ Modifies the **original array in place**.

```php
<?php
$numbers = [3, 1, 4, 1, 5, 9, 2];

sort($numbers);
print_r($numbers);
// Array ( [0] => 1 [1] => 1 [2] => 2 [3] => 3 [4] => 4 [5] => 5 [6] => 9 )

rsort($numbers);
print_r($numbers);
// Array ( [0] => 9 [1] => 5 [2] => 4 [3] => 3 [4] => 2 [5] => 1 [6] => 1 )

$names = ["Banana", "Apple", "Cherry"];
sort($names);
print_r($names);
// Array ( [0] => Apple [1] => Banana [2] => Cherry )
?>
```

---

### `asort()` / `arsort()`

- Like `sort()`/`rsort()`, but **preserves the original keys** (the "a" stands for "associative").
- Essential for associative arrays where keys carry meaning.

```php
<?php
$scores = ["Phyo" => 90, "Alice" => 75, "Bob" => 88];

asort($scores);
print_r($scores);
// Array ( [Alice] => 75 [Bob] => 88 [Phyo] => 90 )  ← sorted by value, keys preserved

arsort($scores);
print_r($scores);
// Array ( [Phyo] => 90 [Bob] => 88 [Alice] => 75 )
?>
```

---

### `ksort()` / `krsort()`

- Sorts an array **by key** (not value), in ascending or descending order.
- Preserves the key-value relationships.

```php
<?php
$person = ["city" => "Yangon", "name" => "Phyo", "age" => 25];

ksort($person);
print_r($person);
// Array ( [age] => 25 [city] => Yangon [name] => Phyo )  ← keys sorted A-Z

krsort($person);
print_r($person);
// Array ( [name] => Phyo [city] => Yangon [age] => 25 )  ← keys sorted Z-A
?>
```

---

### `usort()` / `uasort()` / `uksort()`

- **Custom sorting** with your own comparison callback.
- The callback receives two elements and must return `-1`, `0`, or `1` (perfect use case for the spaceship operator `<=>` from the Operators notes).
- `usort()` — custom sort by value, re-indexes keys.
- `uasort()` — custom sort by value, **preserves keys**.
- `uksort()` — custom sort by **key**.

```php
<?php
// Sort by value — shortest string first
$words = ["banana", "kiwi", "fig", "apple"];

usort($words, fn($a, $b) => strlen($a) <=> strlen($b));
print_r($words);
// Array ( [0] => fig [1] => kiwi [2] => apple [3] => banana )

// Sort array of associative arrays by a specific field — very common in real apps
$users = [
    ["name" => "Charlie", "age" => 30],
    ["name" => "Alice",   "age" => 25],
    ["name" => "Bob",     "age" => 28],
];

// Sort by age ascending
usort($users, fn($a, $b) => $a["age"] <=> $b["age"]);
foreach ($users as $user) {
    echo "{$user['name']}: {$user['age']}\n";
}
// Output:
// Alice: 25
// Bob: 28
// Charlie: 30

// Sort alphabetically by name
usort($users, fn($a, $b) => $a["name"] <=> $b["name"]);
foreach ($users as $user) {
    echo "{$user['name']}\n";
}
// Output:
// Alice
// Bob
// Charlie
?>
```

### Sorting Functions — Summary

| Function | Sorts By | Preserves Keys | Order |
|---|---|---|---|
| `sort()` | Value | ❌ No | Ascending |
| `rsort()` | Value | ❌ No | Descending |
| `asort()` | Value | ✅ Yes | Ascending |
| `arsort()` | Value | ✅ Yes | Descending |
| `ksort()` | Key | ✅ Yes | Ascending |
| `krsort()` | Key | ✅ Yes | Descending |
| `usort()` | Custom (value) | ❌ No | Your choice via callback |
| `uasort()` | Custom (value) | ✅ Yes | Your choice via callback |
| `uksort()` | Custom (key) | ✅ Yes | Your choice via callback |

---

## JSON — Encoding & Decoding

**JSON** (JavaScript Object Notation) is the universal data format for web APIs and services. PHP uses `json_encode()` and `json_decode()` to convert between PHP arrays/objects and JSON strings — you'll use these constantly in backend development.

---

### `json_encode()`

- Converts a **PHP array or object** into a **JSON string**.
- Syntax: `json_encode($value, $flags, $depth)`

```php
<?php
// Basic array to JSON
$user = ["name" => "Phyo", "age" => 25, "city" => "Yangon"];
echo json_encode($user);
// Output: {"name":"Phyo","age":25,"city":"Yangon"}

// Indexed array → JSON array
$fruits = ["apple", "banana", "cherry"];
echo json_encode($fruits);
// Output: ["apple","banana","cherry"]

// Nested / multidimensional
$data = [
    "user"  => ["name" => "Phyo", "age" => 25],
    "tags"  => ["php", "backend", "api"],
    "score" => 99.5,
];
echo json_encode($data);
// Output: {"user":{"name":"Phyo","age":25},"tags":["php","backend","api"],"score":99.5}

// Readable / pretty-printed output (for debugging)
echo json_encode($data, JSON_PRETTY_PRINT);
// Output:
// {
//     "user": {
//         "name": "Phyo",
//         "age": 25
//     },
//     "tags": [
//         "php",
//         "backend",
//         "api"
//     ],
//     "score": 99.5
// }

// Handle non-ASCII characters (like Burmese) — no escaping by default
$burmese = ["greeting" => "မင်္ဂလာပါ"];
echo json_encode($burmese);
// Output: {"greeting":"\u1019\u1004\u103a\u1039\u1002\u101c\u102c\u1015\u102b"}
// ← Unicode escape sequences by default

echo json_encode($burmese, JSON_UNESCAPED_UNICODE);
// Output: {"greeting":"မင်္ဂလာပါ"}  ← actual Burmese characters preserved ✅
?>
```

### Useful `json_encode()` Flags

| Flag | Effect |
|---|---|
| `JSON_PRETTY_PRINT` | Formats the output with indentation — great for debugging |
| `JSON_UNESCAPED_UNICODE` | Keeps non-ASCII characters (Burmese, emoji, etc.) as-is instead of `\uXXXX` |
| `JSON_UNESCAPED_SLASHES` | Prevents `/` from being escaped as `\/` |
| `JSON_THROW_ON_ERROR` | Throws a `JsonException` on error instead of returning `false` |

```php
<?php
// Combine multiple flags with | (bitwise OR)
$data = ["url" => "https://example.com", "note" => "မင်္ဂလာပါ"];
echo json_encode($data, JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT);
// Output:
// {
//     "url": "https://example.com",
//     "note": "မင်္ဂလာပါ"
// }
?>
```

---

### `json_decode()`

- Converts a **JSON string** back into a **PHP value** (array or object).
- Syntax: `json_decode($json, $associative, $depth, $flags)`
- By default returns an **object** (`stdClass`) — pass `true` as the second argument to get an **associative array** instead (almost always what you want in PHP).

```php
<?php
$json = '{"name":"Phyo","age":25,"city":"Yangon"}';

// Default: returns an object
$obj = json_decode($json);
echo $obj->name;   // Output: Phyo
echo $obj->age;    // Output: 25

// ✅ Better for most cases: associative array (pass true as 2nd argument)
$arr = json_decode($json, true);
echo $arr["name"];  // Output: Phyo
echo $arr["age"];   // Output: 25

// Nested JSON
$json = '{"user":{"name":"Phyo","age":25},"tags":["php","backend"]}';
$data = json_decode($json, true);

echo $data["user"]["name"];   // Output: Phyo
echo $data["tags"][0];        // Output: php

// JSON array (not object)
$json = '["apple","banana","cherry"]';
$arr = json_decode($json, true);
echo $arr[1];  // Output: banana
?>
```

---

### Error Handling with JSON

- Always check for errors — malformed JSON or encoding failures are common in real apps (external APIs, user input, file data).

```php
<?php
// Checking for encoding errors
$resource = fopen("test.txt", "r");  // Resources can't be JSON encoded
$result = json_encode(["file" => $resource]);

if ($result === false) {
    echo "Encode error: " . json_last_error_msg();  // Error: Type is not supported
}

// Checking for decoding errors
$badJson = '{name: "Phyo"}';  // Invalid JSON — keys must be quoted
$result = json_decode($badJson, true);

if (json_last_error() !== JSON_ERROR_NONE) {
    echo "Decode error: " . json_last_error_msg();  // Syntax error
}

// ✅ PHP 7.3+ — cleaner error handling with exceptions
try {
    $result = json_decode($badJson, true, 512, JSON_THROW_ON_ERROR);
} catch (\JsonException $e) {
    echo "JSON error: " . $e->getMessage();  // Syntax error
}
?>
```

### Common JSON Functions Summary

| Function | Direction | Returns |
|---|---|---|
| `json_encode($arr)` | PHP → JSON string | JSON string or `false` on error |
| `json_decode($json, true)` | JSON string → PHP array | PHP array, object, or `null` on error |
| `json_last_error()` | — | Error code from last operation |
| `json_last_error_msg()` | — | Human-readable error message |

> 💡 **Practical tip — Building a JSON API response in PHP:**

```php
<?php
// Standard pattern for sending a JSON API response
header("Content-Type: application/json");

$response = [
    "status"  => "success",
    "data"    => [
        "users" => [
            ["id" => 1, "name" => "Phyo"],
            ["id" => 2, "name" => "Alice"],
        ]
    ],
    "message" => "Users fetched successfully",
];

echo json_encode($response, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
?>
```

---

## Bonus — Other Frequently Used Functions

```php
<?php
$arr = [3, 1, 4, 1, 5, 9];

// array_count_values() — count occurrences of each value
$words = ["cat", "dog", "cat", "cat", "dog"];
print_r(array_count_values($words));
// Array ( [cat] => 3 [dog] => 2 )

// array_flip() — swap keys and values
$roles = ["admin" => 1, "editor" => 2, "viewer" => 3];
print_r(array_flip($roles));
// Array ( [1] => admin [2] => editor [3] => viewer )

// array_fill() — create an array filled with a value
print_r(array_fill(0, 5, "PHP"));
// Array ( [0] => PHP [1] => PHP [2] => PHP [3] => PHP [4] => PHP )

// array_diff() — find values in $a NOT in $b
$a = [1, 2, 3, 4, 5];
$b = [3, 4];
print_r(array_diff($a, $b));
// Array ( [0] => 1 [1] => 2 [4] => 5 )  ← keys preserved

// array_intersect() — find values present in ALL given arrays
print_r(array_intersect($a, $b));
// Array ( [2] => 3 [3] => 4 )

// array_combine() — create array from separate keys and values arrays
$keys   = ["name", "age", "city"];
$values = ["Phyo", 25, "Yangon"];
print_r(array_combine($keys, $values));
// Array ( [name] => Phyo [age] => 25 [city] => Yangon )

// compact() — create associative array from variable names and their values
$name = "Phyo";
$age  = 25;
$city = "Yangon";
print_r(compact("name", "age", "city"));
// Array ( [name] => Phyo [age] => 25 [city] => Yangon )

// extract() — reverse of compact: assign array keys as variable names
$data = ["product" => "Shirt", "price" => 10];
extract($data);
echo $product;  // Output: Shirt
echo $price;    // Output: 10
?>
```

> ⚠️ **Warning:** Use `extract()` with care — it creates variables from array keys, which can overwrite existing variables of the same name if you're not careful. Never use it on untrusted input (like `$_POST` or `$_GET`).

---

## Quick Reference Table

| Function | Purpose | Modifies Original? |
|---|---|---|
| `array_keys()` | Get all keys | ❌ No |
| `array_values()` | Get all values, re-indexed | ❌ No |
| `array_search()` | Find a value, return its key | ❌ No |
| `array_key_exists()` | Check if a key exists | ❌ No |
| `array_map()` | Transform every element | ❌ No (returns new) |
| `array_filter()` | Keep elements matching callback | ❌ No (returns new, preserves keys) |
| `array_reduce()` | Collapse array into one value | ❌ No |
| `sort()` / `rsort()` | Sort by value (re-index) | ✅ Yes |
| `asort()` / `arsort()` | Sort by value (preserve keys) | ✅ Yes |
| `ksort()` / `krsort()` | Sort by key | ✅ Yes |
| `usort()` | Custom sort by value (re-index) | ✅ Yes |
| `uasort()` | Custom sort by value (preserve keys) | ✅ Yes |
| `json_encode()` | PHP array/object → JSON string | ❌ No |
| `json_decode()` | JSON string → PHP array/object | ❌ No |
| `array_count_values()` | Count occurrences of each value | ❌ No |
| `array_flip()` | Swap keys and values | ❌ No |
| `array_fill()` | Fill an array with a repeated value | ❌ No |
| `array_diff()` | Values in first array not in others | ❌ No |
| `array_intersect()` | Values present in all arrays | ❌ No |
| `array_combine()` | Merge keys array + values array | ❌ No |
| `compact()` | Variables → associative array | ❌ No |
| `extract()` | Associative array → variables | ✅ (creates vars) |

---

## Quick Revision

- **`array_keys()`** extracts all keys into a new array; with a second argument it returns only keys matching a specific value.
- **`array_values()`** extracts all values and **re-indexes** from 0 — the standard clean-up function after `unset()`, `array_unique()`, or `array_filter()`.
- **`array_search()`** returns the **key** of a found value, or `false` — always check `=== false` because a result of `0` (first element) is falsy.
- **`array_key_exists()`** checks if a **key** exists — unlike `isset()`, it returns `true` even when the value is `null`. Use it when `null` is a meaningful value.
- **`array_map()`** transforms all elements → same-length array. **`array_filter()`** keeps some elements → shorter array (preserves original keys!). **`array_reduce()`** collapses everything → one single value.
- Always pair `array_filter()` with `array_values()` to re-index keys after filtering.
- **Sorting:** `sort()`/`rsort()` for simple indexed arrays; `asort()`/`arsort()` to preserve keys; `ksort()`/`krsort()` to sort by key; `usort()` for custom sorting with `<=>` spaceship operator.
- **`json_encode()`** converts PHP arrays/objects to JSON strings — use `JSON_UNESCAPED_UNICODE` to preserve Burmese characters, `JSON_PRETTY_PRINT` for readable debug output, combine flags with `|`.
- **`json_decode($json, true)`** converts JSON to a PHP **associative array** (always pass `true` for the second argument in most PHP use cases — without it, you get a `stdClass` object).
- Always handle JSON errors with `json_last_error()` or `JSON_THROW_ON_ERROR` in PHP 7.3+.
- **`array_diff()`/`array_intersect()`** compare arrays by value; **`array_combine()`** zips keys and values arrays together; **`compact()`** and **`extract()`** convert between arrays and local variables.