# PHP Miscellaneous Functions & Variable Handling

This note covers two groups of functions you'll use constantly in everyday PHP — the small but powerful "utility" functions that control script execution and generate unique values, and the full set of variable-inspection and type-manipulation functions that help you understand and work with your data safely.

---

## Table of Contents

1. [Miscellaneous Functions](#miscellaneous-functions)
   - [`die()`](#die)
   - [`exit()`](#exit)
   - [`die()` vs `exit()`](#die-vs-exit)
   - [`sleep()`](#sleep)
   - [`uniqid()`](#uniqid)
2. [Variable Handling Functions](#variable-handling-functions)
   - [`empty()`](#empty)
   - [`isset()`](#isset)
   - [`is_null()`](#is_null)
   - [`empty()` vs `isset()` vs `is_null()`](#empty-vs-isset-vs-is_null)
   - [`is_array()`](#is_array)
   - [`intval()`](#intval)
   - [`floatval()`](#floatval)
   - [`settype()`](#settype)
   - [`gettype()`](#gettype)
   - [`print_r()`](#print_r)
   - [`var_dump()`](#var_dump)
   - [`print_r()` vs `var_dump()`](#print_r-vs-var_dump)
3. [All Type-Checking Functions at a Glance](#all-type-checking-functions-at-a-glance)
4. [Quick Revision](#quick-revision)

---

## Miscellaneous Functions

---

### `die()`

- **Stops the script immediately** and optionally outputs a message before stopping.
- Syntax: `die()` or `die("message")` or `die(integer_status_code)`
- The message can be any string — it's printed to the output before the script halts.
- When passed an **integer** (0–254), it sets the exit status code of the PHP process (used in CLI scripts) and prints nothing.

```php
<?php
// Basic usage — stop with a message
die("Something went wrong. Script stopped.");
echo "This never runs.";

// Stop if a required file is missing
$configPath = __DIR__ . "/config.php";
if (!file_exists($configPath)) {
    die("Fatal: config.php not found. Cannot continue.");
}
require($configPath);

// Stop with an error code (no output — used in CLI scripts)
die(1);    // Exit code 1 = general error
die(0);    // Exit code 0 = success (clean exit)

// Stop after a failed database connection
$conn = new mysqli("localhost", "root", "wrong_password", "mydb");
if ($conn->connect_error) {
    die("Database connection failed: " . $conn->connect_error);
}

// Inline: stop right after a redirect (critical pattern!)
header("Location: /login.php");
die();  // Stops execution immediately — nothing runs after the redirect
?>
```

---

### `exit()`

- **Identical to `die()`** — stops the script immediately with an optional message or status code.
- Syntax: `exit()` or `exit("message")` or `exit(integer_status_code)`
- `die()` is literally an **alias** of `exit()` — they compile to the same internal opcode.

```php
<?php
// All identical to die() equivalents
exit();
exit("Permission denied.");
exit(0);    // Clean exit
exit(1);    // Error exit

// Practical patterns
if (!isset($_SESSION["user_id"])) {
    header("Location: /login.php");
    exit();  // Stop execution after redirect
}

// API error response — stop after sending JSON error
function apiError(int $code, string $message): never {
    http_response_code($code);
    header("Content-Type: application/json");
    echo json_encode(["error" => $message]);
    exit();  // Script stops here
}

apiError(403, "Access denied");
echo "Never reached.";

// Stopping inside an included file
// somelib.php
if (PHP_VERSION_ID < 80000) {
    exit("This library requires PHP 8.0 or higher.");
}
?>
```

---

### `die()` vs `exit()`

| Feature | `die()` | `exit()` |
|---|---|---|
| Behavior | Stops execution | Stops execution |
| With string | Prints message and stops | Prints message and stops |
| With integer | Sets exit status, no output | Sets exit status, no output |
| Are they different? | ❌ No — `die()` is an alias of `exit()` |
| Convention | Used for error conditions ("die on failure") | Used for normal termination |

```php
<?php
// By convention — both work identically, but developers often:
$conn = openDatabaseConnection();
if (!$conn) die("Could not connect to database");  // "die" for failure

// vs clean exit
if ($argc > 1 && $argv[1] === "--help") {
    echo "Usage: php script.php [options]\n";
    exit(0);  // "exit" for intended termination
}
?>
```

> 💡 **Convention:** Most PHP developers use `die()` to signal "something failed — stopping here" and `exit()` for a normal, intentional stop. In practice, they're completely interchangeable — choose whichever reads more clearly in context.

---

### `sleep()`

- **Pauses execution** of the current script for a specified number of **seconds**.
- Returns `0` on success, or the number of seconds remaining if interrupted.
- The entire PHP worker process is paused — the server holds the connection open during this time.
- Related: `usleep($microseconds)` — pauses for microseconds (1 second = 1,000,000 microseconds).

```php
<?php
echo "Starting...\n";
sleep(2);              // Pause for 2 seconds
echo "2 seconds later.\n";
sleep(5);              // Pause for 5 seconds
echo "Done after 7 total seconds.\n";

// usleep — finer granularity (microseconds)
echo "Waiting 500ms...\n";
usleep(500000);        // 500,000 microseconds = 0.5 seconds
echo "Done.\n";

// time_nanosleep — even finer (nanoseconds, PHP 5+)
time_nanosleep(0, 100000000);  // 0 seconds, 100,000,000 nanoseconds = 0.1 seconds
?>
```

---

### Common `sleep()` Use Cases

```php
<?php
// 1. Rate limiting — don't hit an external API too fast
function fetchFromApi(string $url): array {
    static $lastCall = 0;

    // Ensure at least 1 second between API calls
    $timeSinceLast = microtime(true) - $lastCall;
    if ($timeSinceLast < 1.0) {
        usleep((int)(($timeSinceLast) * 1000000));  // Sleep the remaining fraction
    }

    $lastCall = microtime(true);
    $response = file_get_contents($url);
    return json_decode($response, true);
}


// 2. Retry logic — wait before retrying a failed operation
function fetchWithRetry(string $url, int $maxAttempts = 3): ?string {
    for ($attempt = 1; $attempt <= $maxAttempts; $attempt++) {
        $result = @file_get_contents($url);

        if ($result !== false) {
            return $result;  // Success
        }

        if ($attempt < $maxAttempts) {
            echo "Attempt $attempt failed. Retrying in 2 seconds...\n";
            sleep(2);  // Wait before retrying
        }
    }
    return null;  // All attempts failed
}


// 3. CLI progress simulation
for ($i = 1; $i <= 5; $i++) {
    echo "Processing step $i...\n";
    sleep(1);
}
echo "Complete!\n";
?>
```

> ⚠️ **Warning:** `sleep()` **blocks the entire PHP process** — the server connection stays open and the worker is unavailable for other requests during the sleep. Never use `sleep()` in a web request handler if avoidable — it wastes server resources and makes your site slow. Use it in **CLI scripts**, **background jobs**, or **queue workers** — not in pages served to browsers.

---

### `uniqid()`

- Generates a **unique identifier** based on the current time (in microseconds).
- Syntax: `uniqid($prefix, $more_entropy)`
- The result is a **string**, not an integer.
- Without extra entropy, IDs generated in the same microsecond may collide — usually fine for most non-security uses.

```php
<?php
// Basic usage — no prefix
echo uniqid();       // Output: e.g. 667b3a2f4c81d  (13 hex chars, time-based)
echo uniqid();       // Output: e.g. 667b3a2f4c81e  (slightly later)

// With a prefix
echo uniqid("user_");      // Output: user_667b3a2f4c81d
echo uniqid("order_");     // Output: order_667b3a2f4c81d
echo uniqid("session_");   // Output: session_667b3a2f4c81d

// With more_entropy = true (adds additional random characters at the end)
echo uniqid("", true);   // Output: 667b3a2f4c81d8.34827491   (longer, more unique)
echo uniqid("id_", true);// Output: id_667b3a2f4c81d8.91827364

// Practical uses
$tempFile   = sys_get_temp_dir() . "/" . uniqid("tmp_") . ".csv";
$uploadName = uniqid("upload_", true) . ".jpg";
$sessionKey = uniqid("sess_", true);

echo $tempFile;    // /tmp/tmp_667b3a2f4c81d.csv
echo $uploadName;  // upload_667b3a2f4c81d8.71827364.jpg
?>
```

> ⚠️ **Security Warning:** `uniqid()` is **not cryptographically secure** — do NOT use it for passwords, tokens, session IDs, API keys, or anything security-sensitive. The output is time-predictable. For secure random strings, always use `bin2hex(random_bytes(16))` or `random_bytes()` instead.

```php
<?php
// ❌ INSECURE — time-predictable, guessable
$resetToken = uniqid("reset_", true);

// ✅ SECURE — cryptographically random
$resetToken  = bin2hex(random_bytes(32));  // 64-char hex string
$apiKey      = bin2hex(random_bytes(16));  // 32-char hex string
$sessionId   = bin2hex(random_bytes(24));  // 48-char hex string

// ✅ Also valid — UUID v4 style
function uuid4(): string {
    $data    = random_bytes(16);
    $data[6] = chr(ord($data[6]) & 0x0f | 0x40);  // Set version 4
    $data[8] = chr(ord($data[8]) & 0x3f | 0x80);  // Set variant
    return vsprintf("%s%s-%s-%s-%s-%s%s%s", str_split(bin2hex($data), 4));
}

echo uuid4();  // Output: e.g. 550e8400-e29b-41d4-a716-446655440000
?>
```

| Function | Secure? | Use For |
|---|---|---|
| `uniqid()` | ❌ No | Temp filenames, cache keys, non-security IDs |
| `bin2hex(random_bytes(16))` | ✅ Yes | Tokens, API keys, session IDs, reset links |
| `uuid4()` (custom) | ✅ Yes | Universally unique IDs for records |

---

## Variable Handling Functions

---

### `empty()`

- Returns `true` if a variable is "empty" — does not exist, or holds a falsy value.
- Does **not** throw a warning if the variable doesn't exist (unlike most functions).
- Works on variables only — cannot be used on function return values directly in older PHP.

```php
<?php
// Values that make empty() return TRUE
var_dump(empty(""));       // bool(true) — empty string
var_dump(empty("0"));      // bool(true) — string "0"
var_dump(empty(0));        // bool(true) — integer zero
var_dump(empty(0.0));      // bool(true) — float zero
var_dump(empty([]));       // bool(true) — empty array
var_dump(empty(false));    // bool(true) — boolean false
var_dump(empty(null));     // bool(true) — null
var_dump(empty($undefined));// bool(true) — undefined variable (no warning!)

// Values that make empty() return FALSE (everything else)
var_dump(empty("hello"));  // bool(false) — non-empty string
var_dump(empty("false"));  // bool(false) — string "false" is NOT empty
var_dump(empty("0.0"));    // bool(false) — string "0.0" is NOT empty
var_dump(empty(1));        // bool(false) — non-zero integer
var_dump(empty(-1));       // bool(false) — negative number
var_dump(empty([0]));      // bool(false) — array with one element
var_dump(empty(true));     // bool(false) — boolean true

// Practical use: validating form input
$username = $_POST["username"] ?? "";
$email    = $_POST["email"]    ?? "";

if (empty($username)) {
    echo "Username is required";
}

if (empty($email)) {
    echo "Email is required";
}

// Checking nested array values safely
$user = ["name" => "Phyo", "bio" => ""];
if (empty($user["bio"])) {
    echo "No bio set";  // Output: No bio set (empty string is empty)
}
?>
```

> ⚠️ **Common Mistake:** `empty(0)` returns `true` and `empty("0")` also returns `true` — so if `0` is a valid value in your context (like a zero price, zero quantity, or the number zero in a math context), `empty()` will falsely report it as "empty." Use `isset()` combined with specific type checks instead.

```php
<?php
// ❌ Bug: empty(0) wrongly says "quantity is missing"
$qty = 0;
if (empty($qty)) {
    echo "No quantity provided";  // ← Wrong! 0 is a valid quantity
}

// ✅ Correct: check that it's actually set and is a number
if (!isset($qty) || !is_numeric($qty)) {
    echo "No quantity provided";
}
?>
```

---

### `isset()`

- Returns `true` if a variable **exists** AND is **not `null`**.
- Returns `false` if the variable doesn't exist, or if it is `null`.
- Doesn't throw a warning for undefined variables.
- Can check multiple variables at once — returns `true` only if ALL are set.

```php
<?php
$name = "Phyo";
$age  = 0;       // Zero — NOT null
$bio  = null;    // Explicitly null

var_dump(isset($name));      // bool(true)  — exists, not null
var_dump(isset($age));       // bool(true)  — 0 is NOT null ← key difference from empty()
var_dump(isset($bio));       // bool(false) — null counts as "not set"
var_dump(isset($unknown));   // bool(false) — doesn't exist (no warning)

// Check multiple variables — ALL must be set
var_dump(isset($name, $age));    // bool(true)  — both set
var_dump(isset($name, $bio));    // bool(false) — $bio is null

// Practical: safe access to $_GET / $_POST / $_SESSION
if (isset($_GET["page"])) {
    $page = (int) $_GET["page"];
} else {
    $page = 1;  // Default
}

// Cleaner with null coalescing operator (modern PHP):
$page = $_GET["page"] ?? 1;  // Equivalent to the above

// Check nested array key exists
$config = ["db" => ["host" => "localhost"]];
var_dump(isset($config["db"]["host"]));    // bool(true)
var_dump(isset($config["db"]["port"]));    // bool(false) — key doesn't exist
var_dump(isset($config["cache"]["ttl"])); // bool(false) — safe, no error
?>
```

---

### `is_null()`

- Returns `true` if and only if the variable is **exactly `null`**.
- Unlike `isset()`, it DOES trigger a notice for undefined variables in some PHP versions.

```php
<?php
$a = null;
$b = 0;
$c = "";
$d = false;

var_dump(is_null($a));    // bool(true)  — exactly null
var_dump(is_null($b));    // bool(false) — 0 is not null
var_dump(is_null($c));    // bool(false) — "" is not null
var_dump(is_null($d));    // bool(false) — false is not null

// Comparing: isset() vs is_null()
$val = null;
var_dump(isset($val));      // bool(false) — null = not set
var_dump(is_null($val));    // bool(true)  — exactly null

// When is is_null() useful?
// When you want to specifically distinguish null from 0, false, ""
function processId(?int $id): string {
    if (is_null($id)) {
        return "No ID provided";
    }
    return "ID is: $id";
}

echo processId(null);  // No ID provided
echo processId(0);     // ID is: 0
echo processId(42);    // ID is: 42
?>
```

---

### `empty()` vs `isset()` vs `is_null()`

This is one of the most important distinctions in PHP variable handling.

```php
<?php
$values = [
    "null_var"    => null,
    "zero_int"    => 0,
    "zero_str"    => "0",
    "empty_str"   => "",
    "empty_arr"   => [],
    "false_bool"  => false,
    "normal"      => "hello",
    "zero_float"  => 0.0,
];

// Results:
//              empty()   isset()   is_null()
// null         TRUE      FALSE     TRUE
// 0            TRUE      TRUE      FALSE  ← isset true, empty true
// "0"          TRUE      TRUE      FALSE  ← isset true, empty true
// ""           TRUE      TRUE      FALSE  ← isset true, empty true
// []           TRUE      TRUE      FALSE
// false        TRUE      TRUE      FALSE
// "hello"      FALSE     TRUE      FALSE
// 0.0          TRUE      TRUE      FALSE
?>
```

| Value | `empty()` | `isset()` | `is_null()` |
|---|---|---|---|
| `null` | `true` | `false` | `true` |
| `0` | `true` | `true` | `false` |
| `"0"` | `true` | `true` | `false` |
| `""` | `true` | `true` | `false` |
| `[]` | `true` | `true` | `false` |
| `false` | `true` | `true` | `false` |
| `"hello"` | `false` | `true` | `false` |
| Undefined | `true` | `false` | ⚠️ Notice |

> 💡 **Decision guide:**
> - Use **`isset($x)`** when you want to know: "does this variable exist and is it not null?" (checking form/array keys, session values)
> - Use **`empty($x)`** when you want: "is this blank/missing/zero/false?" (validation that something was filled in)
> - Use **`is_null($x)`** when you specifically need to test: "is this exactly null (vs 0, false, or '')?"

---

### `is_array()`

- Returns `true` if the variable is an **array** — any array (indexed, associative, empty).

```php
<?php
var_dump(is_array([]));           // bool(true)  — empty array
var_dump(is_array([1, 2, 3]));    // bool(true)  — indexed array
var_dump(is_array(["a" => 1]));   // bool(true)  — associative array
var_dump(is_array("hello"));      // bool(false) — string
var_dump(is_array(42));           // bool(false) — integer
var_dump(is_array(null));         // bool(false) — null

// Practical use: ensure a variable is an array before looping
$data = getSomeData();  // Might return an array or null

if (is_array($data)) {
    foreach ($data as $item) {
        echo $item . "\n";
    }
} else {
    echo "No data to display";
}

// Ensure it's always an array
function ensureArray(mixed $input): array {
    return is_array($input) ? $input : [$input];
}

print_r(ensureArray("single"));   // ["single"]
print_r(ensureArray([1, 2, 3]));  // [1, 2, 3]  — unchanged
print_r(ensureArray(null));       // [null]
?>
```

---

### `intval()`

- Converts a value to an **integer**.
- Syntax: `intval($value, $base)` — `$base` is optional (default 10).
- More flexible than `(int)` casting — supports different number bases.

```php
<?php
// Basic conversions
echo intval("42");         // 42
echo intval("42.9");       // 42   — truncates decimal
echo intval("42abc");      // 42   — reads until non-numeric
echo intval("abc42");      // 0    — starts with letters → 0
echo intval(true);         // 1
echo intval(false);        // 0
echo intval(null);         // 0
echo intval(3.9);          // 3    — truncates, does NOT round

// Base conversion — the unique feature (int) casting doesn't have
echo intval("0x1A", 16);   // 26   — hexadecimal → decimal
echo intval("1A", 16);     // 26   — same (0x prefix optional)
echo intval("10", 2);      // 2    — binary → decimal
echo intval("17", 8);      // 15   — octal → decimal
echo intval("FF", 16);     // 255  — hex → decimal

// Practical: safe integer from URL param
$id = intval($_GET["id"] ?? 0);  // Always an int, never a string
echo $id;
?>
```

---

### `floatval()`

- Converts a value to a **float** (decimal number).
- Aliases: `doubleval()`, `(float)` cast — all identical.

```php
<?php
echo floatval("3.14");       // 3.14
echo floatval("3.14abc");    // 3.14   — stops at "a"
echo floatval("abc");        // 0
echo floatval("1e3");        // 1000   — scientific notation
echo floatval(true);         // 1
echo floatval(false);        // 0
echo floatval(null);         // 0

// Practical use: safe float from form input
$price = floatval($_POST["price"] ?? "0");
echo number_format($price, 2);  // Always a float, formatted to 2 decimal places
?>
```

---

### `settype()`

- Changes the **type of a variable in place** — modifies the original variable directly.
- Unlike casting (`(int)$var`) which returns a new value, `settype()` mutates the variable itself.
- Returns `true` on success, `false` on failure.
- Supported types: `"int"` / `"integer"`, `"float"` / `"double"`, `"string"`, `"bool"` / `"boolean"`, `"array"`, `"object"`, `"null"`.

```php
<?php
// settype() mutates the variable directly
$value = "42";
var_dump($value);          // string(2) "42"

settype($value, "int");
var_dump($value);          // int(42)  ← variable is now an integer

settype($value, "float");
var_dump($value);          // float(42)

settype($value, "string");
var_dump($value);          // string(2) "42"

settype($value, "bool");
var_dump($value);          // bool(true)

settype($value, "array");
var_dump($value);          // array(1) { [0]=> bool(true) }

settype($value, "null");
var_dump($value);          // NULL

// Cast syntax vs settype():
$a = "42";
$b = (int) $a;        // $a stays "42", $b is 42 (creates new value)
settype($a, "int");   // $a is now 42 (mutates in place)

// Practical: normalizing an array of values from a database
$row = ["id" => "1", "price" => "9.99", "active" => "1"];
settype($row["id"],     "int");
settype($row["price"],  "float");
settype($row["active"], "bool");

var_dump($row);
// ["id" => int(1), "price" => float(9.99), "active" => bool(true)]
?>
```

> 💡 **Tip:** In modern PHP, cast operators (`(int)`, `(float)`, `(string)`) are preferred over `settype()` because they're more concise and return a new value without modifying the original. Use `settype()` when you specifically want to mutate the original variable — like normalizing a whole array of database results.

---

### `gettype()`

- Returns the **type name** of a variable as a **string**.
- Returns one of: `"integer"`, `"double"`, `"string"`, `"boolean"`, `"array"`, `"object"`, `"NULL"`, `"resource"`, `"unknown type"`.

> ⚠️ **Note:** PHP internally calls floats "double" — so `gettype(3.14)` returns `"double"`, not `"float"`. This is a historical quirk. `var_dump()` says `float`, but `gettype()` says `double`.

```php
<?php
echo gettype(42);           // "integer"
echo gettype(3.14);         // "double"  ← NOT "float"! (historical quirk)
echo gettype("hello");      // "string"
echo gettype(true);         // "boolean"
echo gettype(false);        // "boolean"
echo gettype([1, 2, 3]);    // "array"
echo gettype(null);         // "NULL"
echo gettype(new stdClass); // "object"

// Practical: routing logic based on type
function processValue(mixed $value): string {
    return match (gettype($value)) {
        "integer" => "Processing int: $value",
        "double"  => "Processing float: $value",
        "string"  => "Processing string: '$value'",
        "array"   => "Processing array with " . count($value) . " items",
        "NULL"    => "No value provided",
        default   => "Unknown type: " . gettype($value),
    };
}

echo processValue(42);          // Processing int: 42
echo processValue(3.14);        // Processing float: 3.14
echo processValue("hello");     // Processing string: 'hello'
echo processValue([1, 2, 3]);   // Processing array with 3 items
echo processValue(null);        // No value provided
?>
```

---

### `print_r()`

- Prints a **human-readable** representation of a variable — especially useful for arrays and objects.
- Syntax: `print_r($value, $return)`
- If `$return` is `true`, returns the output as a string instead of printing it.

```php
<?php
$person = [
    "name"    => "Phyo",
    "age"     => 25,
    "skills"  => ["PHP", "MySQL", "Nginx"],
    "address" => ["city" => "Yangon", "country" => "Myanmar"],
];

print_r($person);
// Array
// (
//     [name]    => Phyo
//     [age]     => 25
//     [skills]  => Array
//         (
//             [0] => PHP
//             [1] => MySQL
//             [2] => Nginx
//         )
//     [address] => Array
//         (
//             [city]    => Yangon
//             [country] => Myanmar
//         )
// )

// Capture as string (for logging or displaying in HTML)
$output = print_r($person, true);  // returns string, doesn't print
file_put_contents("debug.log", $output);

// Display in HTML (wrap in <pre> for formatting)
echo "<pre>" . print_r($person, true) . "</pre>";

// Objects
$obj = new stdClass();
$obj->name = "Phyo";
$obj->role = "admin";
print_r($obj);
// stdClass Object
// (
//     [name] => Phyo
//     [role] => admin
// )
?>
```

---

### `var_dump()`

- Dumps **both the type AND value** of a variable — much more detailed than `print_r()`.
- Shows types at every level of nested arrays/objects.
- Always **prints directly** — no `$return` option (use `ob_start()` to capture it as string).

```php
<?php
var_dump(42);             // int(42)
var_dump(3.14);           // float(3.14)
var_dump("hello");        // string(5) "hello"
var_dump(true);           // bool(true)
var_dump(false);          // bool(false)
var_dump(null);           // NULL
var_dump([1, "two", 3.0]);
// array(3) {
//   [0]=> int(1)
//   [1]=> string(3) "two"
//   [2]=> float(3)
// }

// Nested array — shows type at every level
$data = ["name" => "Phyo", "age" => 25, "active" => true];
var_dump($data);
// array(3) {
//   ["name"]   => string(4) "Phyo"
//   ["age"]    => int(25)
//   ["active"] => bool(true)
// }

// Multiple variables in one call
var_dump($name, $age, $active);

// Capture var_dump as string (for logging)
ob_start();
var_dump($data);
$output = ob_get_clean();
file_put_contents("debug.log", $output);

// Display in HTML
echo "<pre>";
var_dump($data);
echo "</pre>";
?>
```

---

### `print_r()` vs `var_dump()`

| Feature | `print_r()` | `var_dump()` |
|---|---|---|
| Shows type info | ❌ No — values only | ✅ Yes — type + length + value |
| Shows `bool(true/false)` | Shows `1` for `true`, nothing for `false` | Shows `bool(true)` / `bool(false)` |
| Shows `NULL` explicitly | ❌ Shows nothing | ✅ Shows `NULL` |
| String length shown | ❌ No | ✅ Yes — `string(5) "hello"` |
| Can return as string | ✅ Yes — `print_r($v, true)` | ❌ No — use `ob_start()` |
| Multiple args | ❌ One at a time | ✅ `var_dump($a, $b, $c)` |
| Best for | Quick array structure overview | Debugging types, spotting unexpected `null`/`false` |

```php
<?php
$values = [42, "42", 42.0, true, false, null];

// print_r output:
print_r($values);
// Array ( [0] => 42 [1] => 42 [2] => 42 [3] => 1 [4] => [5] => )
// ← Looks identical for int 42, string "42", float 42.0
// ← false and null appear as empty strings — invisible!

// var_dump output:
var_dump($values);
// array(6) {
//   [0]=> int(42)
//   [1]=> string(2) "42"
//   [2]=> float(42)
//   [3]=> bool(true)
//   [4]=> bool(false)
//   [5]=> NULL
// }
// ← Clearly distinct! You can see exactly what type each value is.
?>
```

> 💡 **Rule of thumb:**
> - Use **`print_r()`** when you want a quick, clean overview of an array's structure (keys and values) — especially in simple debugging or logging.
> - Use **`var_dump()`** when you need to know the **exact type** of a value — it's far more informative and should be your default debugging tool.

---

## All Type-Checking Functions at a Glance

```php
<?php
$val = 42;

// Type checkers — all return bool
var_dump(is_int($val));      // bool(true)   — alias: is_integer(), is_long()
var_dump(is_float($val));    // bool(false)  — alias: is_double()
var_dump(is_string($val));   // bool(false)
var_dump(is_bool($val));     // bool(false)
var_dump(is_array($val));    // bool(false)
var_dump(is_object($val));   // bool(false)
var_dump(is_null($val));     // bool(false)
var_dump(is_numeric($val));  // bool(true)   — also true for numeric strings
var_dump(is_callable($val)); // bool(false)  — true for functions, closures

// Type getter
echo gettype($val);          // "integer"

// State checkers
$str = "";
var_dump(isset($str));       // bool(true)   — "" is set (not null)
var_dump(empty($str));       // bool(true)   — "" is empty
?>
```

| Function | Returns | Checks |
|---|---|---|
| `is_int()` | `bool` | Exactly an integer |
| `is_float()` | `bool` | Exactly a float |
| `is_string()` | `bool` | Exactly a string |
| `is_bool()` | `bool` | Exactly a boolean |
| `is_array()` | `bool` | Exactly an array |
| `is_object()` | `bool` | An object instance |
| `is_null()` | `bool` | Exactly null |
| `is_numeric()` | `bool` | Int, float, or numeric string |
| `is_callable()` | `bool` | Callable (function name, closure, etc.) |
| `gettype()` | `string` | Type name as string |
| `isset()` | `bool` | Exists and is not null |
| `empty()` | `bool` | Falsy or doesn't exist |

---

## Quick Revision

- **`die()` and `exit()`** are identical aliases — both stop script execution immediately. Pass a string to print a message, an integer for an exit status code. Always call `exit()`/`die()` after `header("Location: ...")`.
- **`sleep($seconds)`** pauses the script. **`usleep($microseconds)`** for finer control. Only use in CLI scripts and background workers — never in web request handlers.
- **`uniqid()`** generates a time-based unique string — NOT cryptographically secure. Use `bin2hex(random_bytes(16))` for tokens, passwords, and anything security-sensitive.
- **`empty($x)`** returns `true` for: `""`, `"0"`, `0`, `0.0`, `[]`, `false`, `null`, and undefined. Does not warn for undefined variables.
- **`isset($x)`** returns `true` if the variable exists AND is not null. Safe for undefined vars. Can check multiple: `isset($a, $b)`.
- **`is_null($x)`** returns `true` only for exactly `null` — unlike `empty()`, `0` and `""` return `false`.
- **Key difference:** `empty(0)` = `true`, `isset(0)` = `true`, `is_null(0)` = `false`. Only `null` satisfies both `empty()` and `!isset()` and `is_null()`.
- **`is_array()`** checks if something is an array — always use before `foreach` when the source might be null.
- **`intval($val, $base)`** converts to integer — the only casting mechanism that supports different number bases (hex, binary, octal). `floatval()` converts to float.
- **`settype(&$var, "type")`** mutates the variable's type in place. Different from `(int)$var` which creates a new value.
- **`gettype($var)`** returns the type as a string — note it returns `"double"` for floats, not `"float"`.
- **`print_r()`** shows a readable array/object structure (values only). Use `print_r($v, true)` to capture as string.
- **`var_dump()`** shows type + length + value at every level — use this when you need to debug types. It's the more powerful debugger.
- **Debugging combo:** wrap in `<pre>` tags for readable HTML output: `echo "<pre>"; var_dump($data); echo "</pre>";`