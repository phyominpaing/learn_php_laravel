# PHP Control Structures — Conditionals & Loops

**Control structures** determine the **flow** of your program — which code runs, when it runs, and how many times it runs. Without them, code would run top-to-bottom in a straight line every time. Conditionals let you make decisions; loops let you repeat actions.

---

## Table of Contents

1. [Conditionals](#conditionals)
   - [`if` Statement](#if-statement)
   - [`if...else`](#ifelse)
   - [`if...elseif...else`](#ifelseifelse)
   - [Nested `if` Statements](#nested-if-statements)
   - [Alternative Syntax (Template Style)](#alternative-syntax-template-style)
   - [`switch` Statement](#switch-statement)
   - [`match` Expression (PHP 8+)](#match-expression-php-8)
   - [`switch` vs `match`](#switch-vs-match)
2. [Loops](#loops)
   - [`while` Loop](#while-loop)
   - [`do...while` Loop](#dowhile-loop)
   - [`for` Loop](#for-loop)
   - [`foreach` Loop](#foreach-loop)
   - [`break` Statement](#break-statement)
   - [`continue` Statement](#continue-statement)
   - [Nested Loops](#nested-loops)
   - [Infinite Loops](#infinite-loops)
3. [Choosing the Right Structure](#choosing-the-right-structure)
4. [Quick Revision](#quick-revision)

---

## Conditionals

**Conditionals** let your program make decisions — running different code depending on whether a condition is `true` or `false`.

---

### `if` Statement

- The simplest conditional. Runs the code block **only if** the condition is `true`.
- Syntax: `if (condition) { code }`

```php
<?php
$age = 20;

if ($age >= 18) {
    echo "You are an adult.";
}
// Output: You are an adult.

// If the condition is false, nothing happens
$age = 15;
if ($age >= 18) {
    echo "You are an adult.";
}
// Output: (nothing — condition is false)
?>
```

> 💡 **Tip:** The condition inside `()` must evaluate to a **boolean** (or something PHP can convert to boolean). Remember the **truthy/falsy** rules from the Data Types notes — `0`, `""`, `"0"`, `[]`, and `null` are all falsy.

---

### `if...else`

- Runs ONE block **or** the other — never both.
- Syntax: `if (condition) { ... } else { ... }`

```php
<?php
$age = 15;

if ($age >= 18) {
    echo "You are an adult.";
} else {
    echo "You are a minor.";
}
// Output: You are a minor.
?>
```

---

### `if...elseif...else`

- For **multiple conditions** checked in order, top to bottom.
- PHP checks each condition and runs the **first one that's true**, then stops — it does NOT check the rest.
- The final `else` is optional — it runs if NONE of the conditions are true.

```php
<?php
$score = 75;

if ($score >= 90) {
    echo "Grade: A";
} elseif ($score >= 80) {
    echo "Grade: B";
} elseif ($score >= 70) {
    echo "Grade: C";
} elseif ($score >= 60) {
    echo "Grade: D";
} else {
    echo "Grade: F";
}
// Output: Grade: C
// (90 is false, 80 is false, 70 is TRUE → stops here, never checks 60)
?>
```

> ⚠️ **Common Mistake:** Order matters! If you wrote `$score >= 60` BEFORE `$score >= 70`, a score of 75 would incorrectly match the 60 condition first (since 75 >= 60 is also true). **Always order conditions from most specific (highest) to least specific (lowest)** when ranges overlap.

```php
<?php
// ❌ Wrong order — 75 incorrectly gets "D"
$score = 75;
if ($score >= 60) {
    echo "Grade: D";  // Matches first! Wrong.
} elseif ($score >= 70) {
    echo "Grade: C";  // Never reached
}
?>
```

> 💡 **Note:** `elseif` and `else if` (with a space) both work in PHP, but `elseif` (one word) is the conventional style.

---

### Nested `if` Statements

- An `if` statement **inside** another `if` statement — for checking conditions that depend on each other.

```php
<?php
$isLoggedIn = true;
$role       = "admin";

if ($isLoggedIn) {
    echo "Welcome back! ";

    if ($role === "admin") {
        echo "You have admin access.";
    } else {
        echo "You have standard access.";
    }
} else {
    echo "Please log in.";
}
// Output: Welcome back! You have admin access.
```

> 💡 **Tip:** Deeply nested `if` statements (3+ levels) become hard to read. Consider combining conditions with `&&`/`||`, or using early `return` statements inside functions to "flatten" the logic.

```php
// ✅ Flattened with combined condition
if ($isLoggedIn && $role === "admin") {
    echo "You have admin access.";
}
?>
```

---

### Alternative Syntax (Template Style)

- PHP offers an **alternative syntax** using `:` and `end...` keywords instead of `{}`.
- Commonly used when **mixing PHP with HTML** — makes templates more readable.

```php
<?php $isLoggedIn = true; ?>

<?php if ($isLoggedIn): ?>
    <p>Welcome back!</p>
<?php else: ?>
    <p>Please log in.</p>
<?php endif; ?>
```

```php
<?php
// Same logic in regular syntax (harder to read when mixed with HTML)
if ($isLoggedIn) {
    echo "<p>Welcome back!</p>";
} else {
    echo "<p>Please log in.</p>";
}
?>
```

> 💡 **When to use:** The alternative syntax (`:` ... `endif;`) is common in **template files** (`.phtml`, Laravel/legacy templates) where PHP is mixed heavily with HTML. For pure PHP logic files, the `{}` syntax is more common.

---

### `switch` Statement

- Compares ONE value against **multiple possible values** — a cleaner alternative to many `elseif` checks.
- Uses **loose comparison** (`==`) by default.
- Each `case` needs a `break;` or execution will **fall through** to the next case.

```php
<?php
$day = "Wed";

switch ($day) {
    case "Mon":
        echo "Start of the work week";
        break;
    case "Tue":
    case "Wed":
    case "Thu":
        echo "Midweek grind";
        break;
    case "Fri":
        echo "Almost the weekend!";
        break;
    case "Sat":
    case "Sun":
        echo "Weekend!";
        break;
    default:
        echo "Invalid day";
}
// Output: Midweek grind
// (Tue, Wed, Thu share the same code — "fall through" grouping)
?>
```

---

### The Fall-Through Trap

```php
<?php
$num = 2;

switch ($num) {
    case 1:
        echo "One";
    case 2:
        echo "Two";   // Output starts here
    case 3:
        echo "Three"; // ⚠️ Still runs! No break
        break;
    case 4:
        echo "Four";  // Not reached — break stopped it
}
// Output: TwoThree   ← both ran because case 2 has no break!
?>
```

> ⚠️ **Critical Warning:** Forgetting `break;` is one of the **most common PHP bugs**. Without it, execution "falls through" to the next case(s) regardless of whether they match. Always add `break;` unless you **intentionally** want fall-through (like grouping cases together, as shown in the weekday example).

---

### `switch` Uses Loose Comparison

```php
<?php
$value = "0";

switch ($value) {
    case 0:
        echo "Matched 0 (int)";  // ⚠️ This runs! "0" == 0 is true (loose)
        break;
    case "0":
        echo "Matched '0' (string)";
        break;
}
// Output: Matched 0 (int)   ← surprising for many beginners!
?>
```

> ⚠️ **Warning:** `switch` uses `==` (loose comparison), which can cause unexpected matches with mixed types — similar to the `==` pitfalls covered in the Operators notes. If you need strict comparison, use `match` (PHP 8+) or `if/elseif` with `===`.

---

### `match` Expression (PHP 8+)

- A modern, **expression-based** alternative to `switch`.
- Uses **strict comparison** (`===`) — no type-juggling surprises.
- **Returns a value** directly — no need for a variable to be set inside each branch.
- **No fall-through** — and no `break` needed.
- Throws `UnhandledMatchError` if no condition matches and there's no `default`.

```php
<?php
$day = "Wed";

$result = match ($day) {
    "Mon" => "Start of the work week",
    "Tue", "Wed", "Thu" => "Midweek grind",  // Comma = multiple matches, like OR
    "Fri" => "Almost the weekend!",
    "Sat", "Sun" => "Weekend!",
    default => "Invalid day",
};

echo $result;  // Output: Midweek grind
?>
```

```php
<?php
// Strict comparison demonstrated
$value = "0";

$result = match ($value) {
    0   => "Matched 0 (int)",
    "0" => "Matched '0' (string)",  // ✅ This runs — strict match
    default => "No match",
};
echo $result;  // Output: Matched '0' (string)

// match(true) pattern for range checks
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

> ⚠️ **Warning:** If no case matches and there's **no `default`**, `match` throws an `UnhandledMatchError` (a runtime error). Always include `default` unless you're certain every possible value is covered.

---

### `switch` vs `match`

| Feature | `switch` | `match` (PHP 8+) |
|---|---|---|
| Comparison | Loose (`==`) | Strict (`===`) |
| Returns a value | ❌ No (use variables) | ✅ Yes — it's an expression |
| Needs `break` | ✅ Yes (or falls through) | ❌ No — never falls through |
| Multiple values per case | `case A: case B:` | `A, B => ...` |
| Unmatched value (no default) | Does nothing | Throws `UnhandledMatchError` |
| Syntax style | Statement (multi-line blocks) | Expression (concise) |

> 💡 **Recommendation:** In PHP 8+, prefer `match` for simple value-to-value mapping. Use `switch` (or `if/elseif`) when you need to run **multiple statements** or complex logic per case.

---

## Loops

**Loops** repeat a block of code multiple times — until a condition becomes false, or for each item in a collection.

---

### `while` Loop

- Repeats code **as long as** the condition is `true`.
- The condition is checked **before** each iteration — if it's false from the start, the loop body never runs.

```php
<?php
$i = 1;

while ($i <= 5) {
    echo $i . " ";
    $i++;  // ⚠️ Don't forget this, or it loops forever!
}
// Output: 1 2 3 4 5

// Condition false from the start — loop never runs
$count = 10;
while ($count < 5) {
    echo "This never prints";
}
?>
```

> ⚠️ **Warning:** Always make sure something inside the loop eventually makes the condition `false`. Forgetting to update the loop variable (`$i++`) creates an **infinite loop** that will crash or hang your script.

---

### `do...while` Loop

- Similar to `while`, but the condition is checked **AFTER** each iteration.
- This guarantees the loop body runs **at least once**, even if the condition is false from the start.

```php
<?php
$i = 1;

do {
    echo $i . " ";
    $i++;
} while ($i <= 5);
// Output: 1 2 3 4 5  (same result as while in this case)


// The KEY difference — runs once even if condition is initially false
$count = 10;

do {
    echo "This runs once, even though count is already 10";
} while ($count < 5);
// Output: This runs once, even though count is already 10
?>
```

> 💡 **Use case:** `do...while` is useful when you need to **run something first**, then decide whether to repeat — like showing a menu, then asking "Do you want to continue?"

---

### `for` Loop

- The most common loop for repeating something a **known number of times**.
- Syntax: `for (initialization; condition; increment) { code }`
- All three parts are optional but typically all are used.

```php
<?php
for ($i = 1; $i <= 5; $i++) {
    echo $i . " ";
}
// Output: 1 2 3 4 5

// Breaking down the syntax:
// 1. $i = 1        → runs ONCE before the loop starts
// 2. $i <= 5       → checked BEFORE each iteration; loop continues if true
// 3. $i++          → runs AFTER each iteration


// Counting down
for ($i = 5; $i >= 1; $i--) {
    echo $i . " ";
}
// Output: 5 4 3 2 1

// Stepping by 2
for ($i = 0; $i <= 10; $i += 2) {
    echo $i . " ";
}
// Output: 0 2 4 6 8 10

// Looping through an indexed array using a counter
$fruits = ["apple", "banana", "cherry"];
for ($i = 0; $i < count($fruits); $i++) {
    echo $fruits[$i] . " ";
}
// Output: apple banana cherry
?>
```

> 💡 **Tip:** `for` is ideal when you know the **number of iterations** in advance or need access to an **index/counter**. For looping through arrays without needing an index, `foreach` (below) is usually cleaner.

---

### `foreach` Loop

- Specifically designed for iterating over **arrays** (and objects).
- No need to manage a counter — PHP handles it automatically.
- Two forms: `foreach ($array as $value)` and `foreach ($array as $key => $value)`.

```php
<?php
// Indexed array — just values
$fruits = ["apple", "banana", "cherry"];

foreach ($fruits as $fruit) {
    echo $fruit . " ";
}
// Output: apple banana cherry


// Associative array — keys and values
$person = ["name" => "Phyo", "age" => 25, "city" => "Yangon"];

foreach ($person as $key => $value) {
    echo "$key: $value\n";
}
// Output:
// name: Phyo
// age: 25
// city: Yangon


// Indexed array — also access the index with key => value
$colors = ["red", "green", "blue"];
foreach ($colors as $index => $color) {
    echo "$index => $color\n";
}
// Output:
// 0 => red
// 1 => green
// 2 => blue
?>
```

---

### Modifying Array Values with `foreach` (by Reference)

- By default, `foreach` gives you a **copy** of each value — modifying it does NOT change the original array.
- Use `&` to modify the array **in place**.

```php
<?php
$numbers = [1, 2, 3];

// ❌ Without reference — original array unchanged
foreach ($numbers as $num) {
    $num = $num * 2;
}
print_r($numbers);  // [1, 2, 3]  ← unchanged!

// ✅ With reference — original array IS changed
foreach ($numbers as &$num) {
    $num = $num * 2;
}
unset($num);  // Good practice: unset the reference after the loop
print_r($numbers);  // [2, 4, 6]  ← changed!
?>
```

> ⚠️ **Warning:** Always `unset($num)` after a `foreach` with `&$num`. If you reuse `$num` in another loop without unsetting it, it can still be a reference and cause confusing bugs.

---

### Nested `foreach` (Multidimensional Arrays)

```php
<?php
$users = [
    ["name" => "Phyo",  "age" => 25],
    ["name" => "Alice", "age" => 30],
    ["name" => "Bob",   "age" => 22],
];

foreach ($users as $user) {
    echo "{$user['name']} is {$user['age']} years old.\n";
}
// Output:
// Phyo is 25 years old.
// Alice is 30 years old.
// Bob is 22 years old.
?>
```

---

### `break` Statement

- **Immediately exits** the loop entirely — no further iterations happen.
- Works in `for`, `foreach`, `while`, `do...while`, and `switch`.

```php
<?php
for ($i = 1; $i <= 10; $i++) {
    if ($i == 5) {
        break;  // Exit the loop completely when $i reaches 5
    }
    echo $i . " ";
}
// Output: 1 2 3 4
// (5 onwards never printed — loop exited entirely)


// Practical example: search and stop once found
$names = ["Alice", "Bob", "Charlie", "Dave"];
$search = "Charlie";

foreach ($names as $name) {
    if ($name === $search) {
        echo "Found $search!";
        break;  // No need to keep checking after finding it
    }
}
// Output: Found Charlie!
?>
```

### `break N` — Breaking Out of Nested Loops

- `break` can take a number to break out of **multiple nested loops** at once.

```php
<?php
for ($i = 1; $i <= 3; $i++) {
    for ($j = 1; $j <= 3; $j++) {
        if ($j == 2) {
            break 2;  // Breaks BOTH loops, not just the inner one
        }
        echo "i=$i, j=$j\n";
    }
}
// Output: i=1, j=1
// (stops completely after the first inner iteration where $j==2)
?>
```

---

### `continue` Statement

- **Skips the rest of the current iteration** and moves on to the **next** one.
- The loop **does NOT exit** — it just skips ahead.

```php
<?php
for ($i = 1; $i <= 10; $i++) {
    if ($i % 2 === 0) {
        continue;  // Skip even numbers
    }
    echo $i . " ";
}
// Output: 1 3 5 7 9   (even numbers skipped)


// Practical example: skip invalid entries
$ages = [25, -5, 30, 0, 22];

foreach ($ages as $age) {
    if ($age <= 0) {
        continue;  // Skip invalid ages
    }
    echo "Valid age: $age\n";
}
// Output:
// Valid age: 25
// Valid age: 30
// Valid age: 22
?>
```

### `break` vs `continue` — Side by Side

```php
<?php
echo "break:    ";
for ($i = 1; $i <= 5; $i++) {
    if ($i == 3) break;
    echo $i . " ";
}
// Output: break:    1 2

echo "\ncontinue: ";
for ($i = 1; $i <= 5; $i++) {
    if ($i == 3) continue;
    echo $i . " ";
}
// Output: continue: 1 2 4 5
?>
```

| Statement | Effect |
|---|---|
| `break` | Exits the loop **entirely** — no more iterations |
| `continue` | Skips **only the current iteration** — loop keeps going |
| `break N` | Exits N levels of **nested loops** |
| `continue N` | Skips to the next iteration of the loop N levels up |

---

### Nested Loops

- A loop **inside** another loop. The inner loop runs **completely** for each iteration of the outer loop.

```php
<?php
// Multiplication table (1-3)
for ($i = 1; $i <= 3; $i++) {
    for ($j = 1; $j <= 3; $j++) {
        echo "$i x $j = " . ($i * $j) . "\n";
    }
    echo "---\n";
}
// Output:
// 1 x 1 = 1
// 1 x 2 = 2
// 1 x 3 = 3
// ---
// 2 x 1 = 2
// 2 x 2 = 4
// 2 x 3 = 6
// ---
// 3 x 1 = 3
// 3 x 2 = 6
// 3 x 3 = 9
// ---
?>
```

> 💡 **Tip:** Nested loops are common for grids, tables, and matrices, but each level of nesting **multiplies** the number of iterations (3x3 = 9 total here). Be mindful of performance with large datasets — a triple-nested loop over 1,000 items each runs **1 billion times**.

---

### Infinite Loops

- A loop that **never ends** on its own — usually a mistake, but sometimes intentional (with a `break` inside).

```php
<?php
// ❌ Accidental infinite loop — missing increment
// $i = 1;
// while ($i <= 5) {
//     echo $i;
//     // forgot $i++  → loops forever!
// }

// ✅ Intentional infinite loop with break (common in real applications)
$attempts = 0;

while (true) {
    $attempts++;
    echo "Attempt #$attempts\n";

    if ($attempts >= 3) {
        echo "Giving up after 3 attempts.";
        break;  // The only way out
    }
}
// Output:
// Attempt #1
// Attempt #2
// Attempt #3
// Giving up after 3 attempts.
?>
```

> ⚠️ **Warning:** If you write `while (true)`, `for (;;)`, or similar, **always** make sure there's a `break` (or `exit`/`return`) inside that will eventually be reached. Otherwise your script will run forever (until it times out or crashes).

---

## Choosing the Right Structure

| Situation | Best Choice |
|---|---|
| Run code only if a condition is true | `if` |
| Choose between two outcomes | `if...else` |
| Choose among several conditions/ranges | `if...elseif...else` or `match (true)` |
| Compare ONE value against many fixed options | `switch` or `match` |
| Need a return value from a comparison | `match` (PHP 8+) |
| Repeat while a condition holds, unknown count | `while` |
| Repeat at least once, then check condition | `do...while` |
| Repeat a known number of times / need a counter | `for` |
| Loop through every element of an array | `foreach` |
| Stop a loop early | `break` |
| Skip an item but keep looping | `continue` |

---

## Quick Revision

- **`if`** runs code only when a condition is `true`; **`if...else`** picks one of two paths; **`if...elseif...else`** checks multiple conditions in order (first match wins — order matters!).
- The **alternative syntax** (`if (...): ... endif;`) is used mainly in templates mixing PHP and HTML.
- **`switch`** compares one value against many cases using **loose (`==`)** comparison. Forgetting `break;` causes **fall-through** — one of the most common PHP bugs.
- **`match`** (PHP 8+) is the modern alternative: uses **strict (`===`)** comparison, returns a value directly, never falls through, and throws `UnhandledMatchError` if nothing matches and there's no `default`.
- **`while`** checks the condition **before** running — may never execute.
- **`do...while`** checks the condition **after** running — always executes **at least once**.
- **`for`** is best when you know the number of iterations or need a counter: `for (init; condition; step)`.
- **`foreach`** is the go-to for looping through arrays — use `as $value` for values, `as $key => $value` for keys too. Use `as &$value` (with `unset()` after) to modify the array in place.
- **`break`** exits a loop entirely; **`continue`** skips to the next iteration. Both accept a number (`break 2`, `continue 2`) to affect outer loops in nested structures.
- **Nested loops** multiply iteration counts — watch performance with large datasets.
- **Infinite loops** (`while (true)`) are valid but **must** contain a `break` (or similar exit) reachable under normal conditions.