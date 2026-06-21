# PHP Array Functions

PHP arrays are one of the most powerful and frequently used data structures in the language, and PHP ships with **hundreds of built-in array functions** to manipulate them. This note covers the essential ones you'll use constantly — adding/removing elements, merging, searching, slicing, and more.

---

## Table of Contents

1. [Adding & Removing Elements](#adding--removing-elements)
   - [`array_push()`](#array_push)
   - [`array_pop()`](#array_pop)
   - [`array_shift()`](#array_shift)
   - [`array_unshift()`](#array_unshift)
2. [Combining & Splitting Arrays](#combining--splitting-arrays)
   - [`array_merge()`](#array_merge)
   - [`array_chunk()`](#array_chunk)
3. [Math on Arrays](#math-on-arrays)
   - [`array_sum()`](#array_sum)
   - [`array_product()`](#array_product)
4. [Random & Shuffling](#random--shuffling)
   - [`array_rand()`](#array_rand)
   - [`shuffle()`](#shuffle)
5. [Extracting Portions of Arrays](#extracting-portions-of-arrays)
   - [`array_slice()`](#array_slice)
   - [`array_splice()`](#array_splice)
6. [Navigating the Internal Pointer](#navigating-the-internal-pointer)
   - [`current()`](#current)
   - [`end()`](#end)
7. [Searching Arrays](#searching-arrays)
   - [`in_array()`](#in_array)
8. [Assigning to Variables](#assigning-to-variables)
   - [`list()`](#list)
9. [Generating Arrays](#generating-arrays)
   - [`range()`](#range)
10. [Reversing & Deduplicating](#reversing--deduplicating)
    - [`array_reverse()`](#array_reverse)
    - [`array_unique()`](#array_unique)
11. [Converting Arrays to Strings](#converting-arrays-to-strings)
    - [`implode()` / `join()`](#implode--join)
12. [Quick Reference Table](#quick-reference-table)
13. [Quick Revision](#quick-revision)

---

## Adding & Removing Elements

These four functions let you add or remove items from **either end** of an array.

```
array_unshift()  ←  [ a, b, c ]  →  array_push()
                       ↑                ↓
                array_shift()      array_pop()
```

---

### `array_push()`

- Adds one or more elements to the **end** of an array.
- Modifies the original array directly (passed by reference) and returns the **new length**.
- Equivalent to `$array[] = $value;` for a single element.

```php
<?php
$fruits = ["apple", "banana"];

array_push($fruits, "cherry");
print_r($fruits);
// Array ( [0] => apple [1] => banana [2] => cherry )

// Push multiple elements at once
array_push($fruits, "mango", "grape");
print_r($fruits);
// Array ( [0] => apple [1] => banana [2] => cherry [3] => mango [4] => grape )

// Returns the new count
$newCount = array_push($fruits, "kiwi");
echo $newCount;  // Output: 6
?>
```

> 💡 **Tip:** For pushing just **one** element, `$fruits[] = "cherry";` is slightly faster and more common in everyday code than calling `array_push()`. Use `array_push()` when adding **multiple** elements at once.

---

### `array_pop()`

- Removes and **returns** the **last** element of an array.
- Modifies the original array — shrinks it by one.

```php
<?php
$fruits = ["apple", "banana", "cherry"];

$last = array_pop($fruits);
echo $last;          // Output: cherry
print_r($fruits);     // Array ( [0] => apple [1] => banana )

// Practical use: undo / stack behavior (Last In, First Out)
$history = ["page1", "page2", "page3"];
$currentPage = array_pop($history);  // Go back one page
echo $currentPage;   // Output: page3
print_r($history);    // Array ( [0] => page1 [1] => page2 )
?>
```

---

### `array_shift()`

- Removes and **returns** the **first** element of an array.
- ⚠️ **Re-indexes** all remaining numeric keys, starting again from `0`.

```php
<?php
$fruits = ["apple", "banana", "cherry"];

$first = array_shift($fruits);
echo $first;          // Output: apple
print_r($fruits);
// Array ( [0] => banana [1] => cherry )  ← keys re-indexed from 0

// Practical use: processing a queue (First In, First Out)
$queue = ["task1", "task2", "task3"];
$nextTask = array_shift($queue);  // Process the oldest task first
echo $nextTask;   // Output: task1
print_r($queue);   // Array ( [0] => task2 [1] => task3 )
?>
```

---

### `array_unshift()`

- Adds one or more elements to the **beginning** of an array.
- ⚠️ **Re-indexes** all numeric keys after inserting.

```php
<?php
$fruits = ["banana", "cherry"];

array_unshift($fruits, "apple");
print_r($fruits);
// Array ( [0] => apple [1] => banana [2] => cherry )

// Add multiple elements to the front
array_unshift($fruits, "mango", "grape");
print_r($fruits);
// Array ( [0] => mango [1] => grape [2] => apple [3] => banana [4] => cherry )
?>
```

### Add/Remove Functions — Summary

| Function | Adds/Removes From | Returns | Re-indexes Keys? |
|---|---|---|---|
| `array_push()` | End (add) | New array length | No |
| `array_pop()` | End (remove) | The removed value | No |
| `array_shift()` | Start (remove) | The removed value | ✅ Yes |
| `array_unshift()` | Start (add) | New array length | ✅ Yes |

> ⚠️ **Warning:** Be careful with **associative arrays** — `array_shift()` and `array_unshift()` re-index **numeric** keys but leave **string** keys untouched. This can cause confusion if you mix numeric and string keys.

---

## Combining & Splitting Arrays

### `array_merge()`

- Combines two or more arrays into **one new array**.
- For **string keys**, later arrays **overwrite** earlier ones if keys match.
- For **numeric keys**, values are **re-indexed** sequentially (no overwriting — duplicates are kept).

```php
<?php
// Numeric arrays — values are re-indexed, nothing is overwritten
$a = ["red", "green"];
$b = ["blue", "yellow"];
print_r(array_merge($a, $b));
// Array ( [0] => red [1] => green [2] => blue [3] => yellow )

// Associative arrays — matching string keys get OVERWRITTEN by the later array
$user1 = ["name" => "Phyo", "age" => 25];
$user2 = ["age" => 26, "city" => "Yangon"];
print_r(array_merge($user1, $user2));
// Array ( [name] => Phyo [age] => 26 [city] => Yangon )
// "age" became 26 — $user2's value overwrote $user1's

// Merging multiple arrays at once
$combined = array_merge([1, 2], [3, 4], [5, 6]);
print_r($combined);  // Array ( [0] => 1 [1] => 2 [2] => 3 [3] => 4 [4] => 5 [5] => 6 )
?>
```

> 💡 **Tip:** Compare this to the `+` array union operator — `$a + $b` keeps the **left** array's values on key conflicts, while `array_merge($a, $b)` lets the **right** (later) array win. Choose based on which behavior you need.

---

### `array_chunk()`

- Splits an array into **multiple smaller arrays** ("chunks") of a specified size.
- The last chunk may have fewer elements if the array doesn't divide evenly.

```php
<?php
$numbers = [1, 2, 3, 4, 5, 6, 7];

print_r(array_chunk($numbers, 3));
// Array (
//   [0] => Array ( [0] => 1 [1] => 2 [2] => 3 )
//   [1] => Array ( [0] => 4 [1] => 5 [2] => 6 )
//   [2] => Array ( [0] => 7 )            ← last chunk has just 1 leftover element
// )

// Practical use: displaying items in a grid (e.g., 3 products per row)
$products = ["Shirt", "Pants", "Shoes", "Hat", "Socks"];
$rows = array_chunk($products, 3);

foreach ($rows as $row) {
    echo implode(" | ", $row) . "\n";
}
// Output:
// Shirt | Pants | Shoes
// Hat | Socks

// Preserve original keys with the 3rd parameter
print_r(array_chunk(["a", "b", "c", "d"], 2, true));
// Array ( [0] => Array([0]=>a [1]=>b) [1] => Array([2]=>c [3]=>d) )
?>
```

---

## Math on Arrays

### `array_sum()`

- Returns the **sum** of all values in an array.
- Non-numeric values are treated as `0`.

```php
<?php
echo array_sum([1, 2, 3, 4, 5]);     // Output: 15
echo array_sum([1.5, 2.5, 3]);       // Output: 7
echo array_sum([]);                  // Output: 0

// Practical use: calculating a cart total
$prices = [19.99, 5.50, 12.25];
echo array_sum($prices);  // Output: 37.74

// Mixed valid/invalid values
echo array_sum([1, "2", "abc", 3]);  // Output: 6  ("2" → 2, "abc" → 0)
?>
```

---

### `array_product()`

- Returns the **product** (result of multiplying everything together) of all values.

```php
<?php
echo array_product([1, 2, 3, 4]);   // Output: 24  (1*2*3*4)
echo array_product([5]);            // Output: 5
echo array_product([]);             // Output: 1  (empty array → identity value)

// Practical use: calculating factorial using array_product + range
function factorial($n) {
    return array_product(range(1, $n));
}
echo factorial(5);  // Output: 120  (5! = 5*4*3*2*1)
?>
```

---

## Random & Shuffling

### `array_rand()`

- Picks one or more **random keys** from an array (NOT the values directly — you use the keys to look up values).
- Does **not** modify the original array.

```php
<?php
$fruits = ["apple", "banana", "cherry", "mango"];

$randomKey = array_rand($fruits);
echo $fruits[$randomKey];  // Output: a random fruit, e.g. "cherry"

// Getting multiple random elements
$randomKeys = array_rand($fruits, 2);  // Returns an array of 2 random keys
print_r($randomKeys);   // e.g. Array ( [0] => 0 [1] => 3 )

foreach ($randomKeys as $key) {
    echo $fruits[$key] . " ";
}
// Output: e.g. "apple mango"
?>
```

> ⚠️ **Warning:** `array_rand()` returns **keys**, not values — a very common beginner mistake is to assume it returns the actual values directly. Always use the returned key(s) to look up the value(s) in the original array.

---

### `shuffle()`

- **Randomly reorders** all elements of an array **in place**.
- ⚠️ Destroys any existing keys — always re-indexes numerically from `0`, even for associative arrays (so avoid using it on associative arrays where keys matter).

```php
<?php
$deck = [1, 2, 3, 4, 5];
shuffle($deck);
print_r($deck);  // e.g. Array ( [0] => 3 [1] => 1 [2] => 5 [3] => 2 [4] => 4 )

// Practical use: randomizing quiz question order
$questions = ["Q1", "Q2", "Q3", "Q4"];
shuffle($questions);
print_r($questions);  // Random order each time
?>
```

> ⚠️ **Warning:** Like `rand()`/`mt_rand()`, `shuffle()` is **not cryptographically secure**. Don't use it for anything security-sensitive (like generating secure tokens or shuffling in a real-money gambling context).

---

## Extracting Portions of Arrays

### `array_slice()`

- Extracts a **portion** of an array **without modifying** the original.
- Syntax: `array_slice($array, $offset, $length, $preserveKeys)`
- Does NOT change the original array — returns a new one.

```php
<?php
$letters = ["a", "b", "c", "d", "e"];

print_r(array_slice($letters, 1, 3));
// Array ( [0] => b [1] => c [2] => d )   ← keys re-indexed by default

print_r(array_slice($letters, 2));
// Array ( [0] => c [1] => d [2] => e )   ← from index 2 to the end

// Negative offset — counts from the END
print_r(array_slice($letters, -2));
// Array ( [0] => d [1] => e )   ← last 2 elements

// Preserving original keys
print_r(array_slice($letters, 1, 3, true));
// Array ( [1] => b [2] => c [3] => d )   ← original keys kept

// Practical use: pagination
$allItems = range(1, 100);
$page = 2;
$perPage = 10;
$pageItems = array_slice($allItems, ($page - 1) * $perPage, $perPage);
print_r($pageItems);  // Items 11 through 20
?>
```

---

### `array_splice()`

- **Removes and/or replaces** a portion of an array, and can also **insert** new elements.
- ⚠️ Unlike `array_slice()`, this function **MODIFIES the original array directly**.
- Syntax: `array_splice($array, $offset, $length, $replacement)`

```php
<?php
$letters = ["a", "b", "c", "d", "e"];

// Removing elements (returns the removed portion)
$removed = array_splice($letters, 1, 2);
print_r($removed);   // Array ( [0] => b [1] => c )  ← what was removed
print_r($letters);   // Array ( [0] => a [1] => d [2] => e )  ← original array CHANGED

// Replacing elements
$letters = ["a", "b", "c", "d", "e"];
array_splice($letters, 1, 2, ["X", "Y", "Z"]);
print_r($letters);
// Array ( [0] => a [1] => X [2] => Y [3] => Z [4] => d [5] => e )
// 2 elements removed, 3 elements inserted in their place

// Inserting without removing (length = 0)
$letters = ["a", "b", "c"];
array_splice($letters, 1, 0, ["NEW"]);
print_r($letters);
// Array ( [0] => a [1] => NEW [2] => b [3] => c )
?>
```

### `array_slice()` vs `array_splice()`

| Feature | `array_slice()` | `array_splice()` |
|---|---|---|
| Modifies original array | ❌ No | ✅ Yes |
| Can insert/replace elements | ❌ No | ✅ Yes |
| Returns | The extracted portion (new array) | The removed portion |
| Use when | You just want to **read** a portion | You want to **edit** the array directly |

---

## Navigating the Internal Pointer

PHP arrays have an **internal pointer** that tracks "where you are" when manually navigating — separate from `foreach`, which uses its own mechanism.

### `current()`

- Returns the value at the array's **current internal pointer position**.
- By default, the pointer starts at the **first** element.
- Related functions: `next()` (move pointer forward), `prev()` (move back), `reset()` (move to start), `end()` (move to last — see below), `key()` (get current key).

```php
<?php
$fruits = ["apple", "banana", "cherry"];

echo current($fruits);  // Output: apple   (pointer starts at the first element)
echo next($fruits);     // Output: banana  (moves pointer forward, returns new current)
echo next($fruits);     // Output: cherry
echo next($fruits);     // Output: (nothing — false, no more elements)

echo reset($fruits);    // Output: apple   (resets pointer back to the start)
echo key($fruits);      // Output: 0       (gets the current key)
?>
```

> 💡 **Note:** In modern PHP code, `foreach` is used far more often than manually moving the pointer with `current()`/`next()`. These pointer functions are mostly useful in specific scenarios — like manually stepping through an array alongside other logic, or working with legacy code.

---

### `end()`

- Moves the internal pointer to the **last** element and returns its value.

```php
<?php
$fruits = ["apple", "banana", "cherry"];

echo end($fruits);      // Output: cherry  (pointer now at the last element)
echo current($fruits);  // Output: cherry  (confirms pointer moved)
echo prev($fruits);     // Output: banana  (move pointer one step back)

reset($fruits);         // Always good practice to reset when done navigating
?>
```

> 💡 **Common practical use:** `end($array)` is a quick way to get the **last element's value** without knowing its key — especially handy for arrays with non-sequential or string keys.

```php
<?php
$prices = [10, 20, 30, 45];
echo end($prices);   // Output: 45  — quick way to get the last price

// Note: end() requires a real variable, not a function's return value directly
// echo end(getArray());  // ❌ This causes a notice in PHP — assign to a variable first
$arr = getPrices();
echo end($arr);  // ✅ Correct
?>
```

---

## Searching Arrays

### `in_array()`

- Checks whether a **value exists** anywhere in an array.
- Returns `true` or `false`.
- Uses **loose comparison** by default — pass `true` as the third argument for **strict** comparison.

```php
<?php
$fruits = ["apple", "banana", "cherry"];

var_dump(in_array("banana", $fruits));   // bool(true)
var_dump(in_array("mango", $fruits));    // bool(false)

// Loose comparison can cause surprises
$numbers = [0, 1, 2];
var_dump(in_array("apple", $numbers));   // bool(true) in PHP 7! ⚠️ "apple" loosely == 0
                                          // bool(false) in PHP 8 (behavior fixed)

// Strict comparison avoids the issue — ALWAYS prefer this
var_dump(in_array("apple", $numbers, true));  // bool(false)  ✅ correct, regardless of PHP version

// Practical use: validating allowed values
$allowedRoles = ["admin", "editor", "viewer"];
$userRole = "editor";

if (in_array($userRole, $allowedRoles, true)) {
    echo "Valid role";  // Output: Valid role
}
?>
```

> ⚠️ **Warning:** Always pass `true` as the third argument (`in_array($needle, $haystack, true)`) to use **strict comparison**, especially when checking against mixed-type arrays. This avoids the classic PHP type-juggling traps covered in the Operators notes.

---

## Assigning to Variables

### `list()`

- Assigns array elements to **multiple variables** in one statement (older syntax — the modern equivalent is short array destructuring `[$a, $b] = ...`, covered in the Functions notes).

```php
<?php
$person = ["Phyo", 25, "Yangon"];

list($name, $age, $city) = $person;
echo "$name is $age years old, from $city.";
// Output: Phyo is 25 years old, from Yangon.

// ✅ Modern equivalent — shorthand square bracket syntax (preferred in PHP 7.1+)
[$name, $age, $city] = $person;
echo "$name is $age years old, from $city.";  // Same result

// Skipping elements
list(, $age, ) = $person;   // Skip first and third
echo $age;  // Output: 25

// Destructuring associative arrays (PHP 7.1+)
$user = ["name" => "Alice", "age" => 30];
["name" => $userName, "age" => $userAge] = $user;
echo "$userName, $userAge";  // Output: Alice, 30
?>
```

> 💡 **Tip:** `list()` and `[...]` destructuring do the exact same thing — `[...]` is just shorter and more modern. You'll still see `list()` in older codebases, so it's worth recognizing, but prefer `[$a, $b] = ...` in new code.

---

## Generating Arrays

### `range()`

- Generates an array containing a **sequence of numbers or letters** between a start and end value.
- Syntax: `range($start, $end, $step)`

```php
<?php
print_r(range(1, 5));
// Array ( [0] => 1 [1] => 2 [2] => 3 [3] => 4 [4] => 5 )

print_r(range(5, 1));         // Counts DOWN automatically if start > end
// Array ( [0] => 5 [1] => 4 [2] => 3 [3] => 2 [4] => 1 )

print_r(range(0, 10, 2));     // With a step value
// Array ( [0] => 0 [1] => 2 [2] => 4 [3] => 6 [4] => 8 [5] => 10 )

print_r(range("a", "e"));     // Works with letters too!
// Array ( [0] => a [1] => b [2] => c [3] => d [4] => e )

print_r(range("z", "v"));     // Letters can count down too
// Array ( [0] => z [1] => y [2] => x [3] => w [4] => v )

// Practical use: generating year options for a dropdown
$years = range(2020, 2026);
print_r($years);  // [2020, 2021, 2022, 2023, 2024, 2025, 2026]
?>
```

> 💡 **Tip:** `range()` is commonly combined with `array_sum()`, `array_product()`, or `foreach` for quick number sequences — like generating page numbers, calendar days, or simple test data.

---

## Reversing & Deduplicating

### `array_reverse()`

- Returns a **new array** with elements in **reverse order**. Does not modify the original.
- By default, **re-indexes** numeric keys; string keys are always preserved.

```php
<?php
$numbers = [1, 2, 3, 4, 5];

print_r(array_reverse($numbers));
// Array ( [0] => 5 [1] => 4 [2] => 3 [3] => 2 [4] => 1 )

// Preserving original numeric keys with the 2nd parameter
print_r(array_reverse($numbers, true));
// Array ( [4] => 5 [3] => 4 [2] => 3 [1] => 2 [0] => 1 )

// String keys are ALWAYS preserved, regardless of the 2nd parameter
$person = ["name" => "Phyo", "age" => 25, "city" => "Yangon"];
print_r(array_reverse($person));
// Array ( [city] => Yangon [age] => 25 [name] => Phyo )
?>
```

---

### `array_unique()`

- Removes **duplicate values** from an array, keeping the **first occurrence** of each.
- Keys from the original array are **preserved** (not re-indexed) — often combined with `array_values()` to clean them up afterward.

```php
<?php
$numbers = [1, 2, 2, 3, 3, 3, 4];

print_r(array_unique($numbers));
// Array ( [0] => 1 [1] => 2 [3] => 3 [6] => 4 )
// Notice: original keys preserved — indexes 2, 4, 5 are "missing" (they were duplicates)

// Re-indexing after removing duplicates
print_r(array_values(array_unique($numbers)));
// Array ( [0] => 1 [1] => 2 [2] => 3 [3] => 4 )  ← clean sequential keys

// Works with strings too
$names = ["Phyo", "Alice", "Phyo", "Bob", "Alice"];
print_r(array_values(array_unique($names)));
// Array ( [0] => Phyo [1] => Alice [2] => Bob )
?>
```

> ⚠️ **Note:** `array_unique()` compares values as **strings** by default — this can occasionally cause unexpected matches with mixed types (e.g., `1` and `"1"` are considered duplicates). For most everyday use with plain numbers or strings, this isn't an issue.

---

## Converting Arrays to Strings

### `implode()` / `join()`

- `implode()` joins array elements into a single string, separated by a delimiter — already covered in depth in the String Functions notes, included here for completeness since it's so closely tied to array work.
- `join()` is simply an **alias** for `implode()` — they are 100% identical in behavior.

```php
<?php
$fruits = ["apple", "banana", "cherry"];

echo implode(", ", $fruits);   // Output: apple, banana, cherry
echo join(", ", $fruits);      // Output: apple, banana, cherry  ← exact same result

// Practical use: combining with array functions covered above
$numbers = range(1, 5);
echo implode(" + ", $numbers) . " = " . array_sum($numbers);
// Output: 1 + 2 + 3 + 4 + 5 = 15
?>
```

> 💡 **Tip:** `implode()` is by far the more commonly used name in real-world PHP code — `join()` exists mainly for developers coming from other languages (like JavaScript) where `join()` is the standard name. Stick with `implode()` for consistency with PHP conventions.

---

## Quick Reference Table

| Function | Purpose | Modifies Original? |
|---|---|---|
| `array_push()` | Add to end | ✅ Yes |
| `array_pop()` | Remove from end | ✅ Yes |
| `array_shift()` | Remove from start | ✅ Yes (re-indexes) |
| `array_unshift()` | Add to start | ✅ Yes (re-indexes) |
| `array_merge()` | Combine arrays | ❌ No (returns new) |
| `array_chunk()` | Split into smaller arrays | ❌ No |
| `array_sum()` | Total of all values | ❌ No |
| `array_product()` | Multiply all values | ❌ No |
| `array_rand()` | Pick random key(s) | ❌ No |
| `shuffle()` | Randomize order | ✅ Yes |
| `array_slice()` | Extract a portion | ❌ No |
| `array_splice()` | Remove/replace/insert a portion | ✅ Yes |
| `current()` | Get value at pointer | ❌ No (moves pointer only) |
| `end()` | Move pointer to last, get value | ❌ No (moves pointer only) |
| `in_array()` | Check if value exists | ❌ No |
| `list()` / `[...]` | Destructure into variables | ❌ No |
| `range()` | Generate a sequence | N/A (creates new) |
| `array_reverse()` | Reverse order | ❌ No (returns new) |
| `array_unique()` | Remove duplicates | ❌ No (returns new) |
| `implode()` / `join()` | Array → string | ❌ No |

---

## Quick Revision

- **`array_push()`/`array_pop()`** work on the **end**; **`array_unshift()`/`array_shift()`** work on the **start**. Shift/unshift **re-index** numeric keys; push/pop don't.
- **`array_merge()`** combines arrays — string keys get **overwritten** by later arrays, numeric keys get **re-indexed** (no overwriting).
- **`array_chunk()`** splits one array into multiple smaller arrays of a given size.
- **`array_sum()`** adds all values; **`array_product()`** multiplies them all together.
- **`array_rand()`** returns random **keys**, not values — look up the value yourself. **`shuffle()`** randomizes the array in place and **destroys keys** — neither is cryptographically secure.
- **`array_slice()`** reads a portion **without** modifying the original; **`array_splice()`** removes/replaces/inserts and **does** modify the original.
- **`current()`/`end()`/`next()`/`prev()`/`reset()`** navigate the array's internal pointer — mostly superseded by `foreach` in modern code, but `end()` is still handy for quickly grabbing the last value.
- **`in_array()`** checks for a value's existence — **always pass `true`** as the third argument for strict comparison to avoid type-juggling bugs.
- **`list()`** (or modern `[$a, $b] = $array`) destructures array values into separate variables.
- **`range()`** generates sequences of numbers or letters, ascending or descending, with optional step.
- **`array_reverse()`** flips element order (re-indexes numeric keys by default, preserves string keys); **`array_unique()`** removes duplicate values (preserves original keys — pair with `array_values()` to re-index).
- **`implode()`** and **`join()`** are identical — `implode()` is the conventional PHP name to use.
- **Mental shortcut:** functions that "build a new array" (slice, merge, reverse, unique, chunk) generally **don't** touch the original; functions that "edit in place" (push, pop, shift, unshift, splice, shuffle) **do**.