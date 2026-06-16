# PHP String Functions

PHP has hundreds of **built-in string functions** for measuring, searching, transforming, and formatting text. This is your reference for the most commonly used ones — including special handling for multibyte text like Burmese.

---

## Table of Contents

1. [Why String Functions Matter](#why-string-functions-matter)
2. [Measuring Strings](#measuring-strings)
   - [`strlen()`](#strlen)
   - [`str_word_count()`](#str_word_count)
3. [Changing Case](#changing-case)
   - [`strtolower()` / `strtoupper()`](#strtolower--strtoupper)
   - [`ucfirst()` / `ucwords()`](#ucfirst--ucwords)
4. [Reversing & Searching](#reversing--searching)
   - [`strrev()`](#strrev)
   - [`strpos()` / `strrpos()`](#strpos--strrpos)
5. [Replacing Text](#replacing-text)
   - [`str_replace()`](#str_replace)
6. [Extracting Substrings](#extracting-substrings)
   - [`substr()`](#substr)
   - [`mb_substr()` — Multibyte Strings & Burmese](#mb_substr--multibyte-strings--burmese)
7. [Padding Strings](#padding-strings)
   - [`str_pad()`](#str_pad)
8. [Splitting & Joining](#splitting--joining)
   - [`explode()`](#explode)
   - [`implode()`](#implode)
9. [Trimming Whitespace](#trimming-whitespace)
   - [`trim()` / `ltrim()` / `rtrim()`](#trim--ltrim--rtrim)
10. [HTML Escaping](#html-escaping)
    - [`htmlentities()`](#htmlentities)
    - [`html_entity_decode()`](#html_entity_decode)
    - [`htmlspecialchars()`](#htmlspecialchars)
11. [Hashing Strings](#hashing-strings)
    - [`md5()`](#md5)
12. [Multibyte Functions Reference (`mb_*`)](#multibyte-functions-reference-mb_)
13. [Quick Revision](#quick-revision)

---

## Why String Functions Matter

- Almost every PHP application processes text — form input, database content, file data, API responses.
- Knowing the right built-in function saves you from writing manual character-by-character logic.
- PHP's native string functions (`strlen`, `substr`, etc.) work on **bytes**, not characters. This matters a lot for non-English text — covered in detail in the `mb_substr()` section.

---

## Measuring Strings

### `strlen()`

- Returns the **length of a string** in **bytes**.
- For plain ASCII/English text, byte count = character count, so this works fine.
- For multibyte text (like Burmese, Chinese, emoji), this gives a **misleading** result — see the warning below.

```php
<?php
echo strlen("Hello");        // Output: 5
echo strlen("Hello World");  // Output: 11  (includes the space)
echo strlen("");             // Output: 0

// Practical use: validating input length
$username = "phyo123";
if (strlen($username) < 5) {
    echo "Username too short";
} else {
    echo "Username OK";  // Output: Username OK
}
?>
```

> ⚠️ **Warning:** `strlen("မင်္ဂလာပါ")` does NOT return the number of Burmese characters — it returns the number of **bytes** used to encode them in UTF-8 (often 3 bytes per character). Use `mb_strlen()` for accurate character counts in non-English text. Covered in the [multibyte section](#mb_substr--multibyte-strings--burmese).

```php
<?php
$burmese = "မင်္ဂလာပါ";  // "Hello" in Burmese
echo strlen($burmese);     // Output: 27 (bytes) — NOT the actual character count!
echo mb_strlen($burmese);  // Output: 9  (actual character count) ✅
?>
```

---

### `str_word_count()`

- Counts the **number of words** in a string (English/Latin-script text).
- By default returns an integer count; can also return the words as an array.

```php
<?php
$text = "The quick brown fox jumps";

echo str_word_count($text);        // Output: 5

// Return mode 1: array of words
print_r(str_word_count($text, 1));
// Array ( [0] => The [1] => quick [2] => brown [3] => fox [4] => jumps )

// Return mode 2: array with position as key
print_r(str_word_count($text, 2));
// Array ( [0] => The [4] => quick [10] => brown [16] => fox [20] => jumps )
?>
```

> ⚠️ **Warning:** `str_word_count()` is designed for Latin-script languages and counts based on spaces and letters — it does **not** reliably count words in Burmese or other scripts that don't use spaces the same way. For Burmese word counting, you typically need a dedicated word-segmentation library.

---

## Changing Case

### `strtolower()` / `strtoupper()`

- Convert a string to **all lowercase** or **all uppercase**.
- Works on ASCII letters by default (A-Z, a-z).

```php
<?php
echo strtolower("HELLO WORLD");  // Output: hello world
echo strtoupper("hello world");  // Output: HELLO WORLD

// Practical use: case-insensitive comparison
$input = "YES";
if (strtolower($input) === "yes") {
    echo "Confirmed!";  // Output: Confirmed!
}
?>
```

> 💡 **Tip:** Use `mb_strtolower()` / `mb_strtoupper()` if you need to handle accented characters (like é, ü) or non-Latin scripts correctly.

---

### `ucfirst()` / `ucwords()`

- `ucfirst()` — capitalizes only the **first letter** of a string.
- `ucwords()` — capitalizes the **first letter of every word**.

```php
<?php
echo ucfirst("hello world");   // Output: Hello world  (only "h" capitalized)
echo ucwords("hello world");   // Output: Hello World  (every word capitalized)

// Practical use: formatting names
$name = "phyo min paing";
echo ucwords($name);   // Output: Phyo Min Paing

// ucwords with custom delimiters
echo ucwords("hello-world_test", "-_");  // Output: Hello-World_Test
?>
```

---

## Reversing & Searching

### `strrev()`

- Reverses a string — last character becomes first.

```php
<?php
echo strrev("Hello");     // Output: olleH
echo strrev("PHP");       // Output: PHP   ← palindrome stays the same!

// Practical use: checking palindromes
function isPalindrome($str) {
    $clean = strtolower(str_replace(" ", "", $str));
    return $clean === strrev($clean);
}

var_dump(isPalindrome("Racecar"));     // bool(true)
var_dump(isPalindrome("Hello"));       // bool(false)
?>
```

> ⚠️ **Warning:** `strrev()` reverses **bytes**, not characters. It will break multibyte text like Burmese or emoji. There is no direct `mb_strrev()` — for multibyte-safe reversal, you'd split into an array of characters with `mb_str_split()` and reverse the array.

---

### `strpos()` / `strrpos()`

- `strpos()` finds the position of the **first occurrence** of a substring.
- `strrpos()` finds the position of the **last occurrence**.
- Returns the position (an integer, **0-indexed**) or `false` if not found.

```php
<?php
$text = "Hello World, Hello PHP";

echo strpos($text, "Hello");    // Output: 0   (first "Hello" starts at index 0)
echo strpos($text, "World");    // Output: 6
echo strrpos($text, "Hello");   // Output: 13  (last "Hello" starts at index 13)
echo strpos($text, "Python");   // Output: (nothing — returns false)

// ⚠️ CRITICAL: must use === to check for "not found"
$pos = strpos($text, "Python");
if ($pos === false) {           // ✅ Correct
    echo "Not found";           // Output: Not found
}

// Why not just "if (!$pos)"? Because position 0 is also falsy!
$pos = strpos($text, "Hello");  // Returns 0 (found at the very start)
if (!$pos) {
    echo "Not found";  // ❌ WRONG! Prints "Not found" even though it WAS found at index 0
}
?>
```

> ⚠️ **Critical Warning:** Always compare with `=== false` (strict), never `== false` or just `!$pos`. Since a match at position `0` is falsy, a loose check would incorrectly treat a valid match (at the very beginning of the string) as "not found." This is one of PHP's most common beginner bugs.

```php
<?php
// Practical use: check if a string contains a substring
$email = "phyo@example.com";

if (strpos($email, "@") !== false) {
    echo "Looks like a valid format";
}

// PHP 8+ alternative — clearer intent, no false-vs-0 trap
if (str_contains($email, "@")) {
    echo "Contains @ symbol";
}
?>
```

---

## Replacing Text

### `str_replace()`

- Replaces all occurrences of a search string with a replacement string.
- Syntax: `str_replace($search, $replace, $subject)`
- Case-sensitive (use `str_ireplace()` for case-insensitive replacement).

```php
<?php
echo str_replace("World", "PHP", "Hello World");  // Output: Hello PHP

// Replacing multiple occurrences
echo str_replace("a", "o", "banana");  // Output: bonono

// Multiple search/replace pairs using arrays
$search  = ["cat", "dog"];
$replace = ["lion", "wolf"];
echo str_replace($search, $replace, "The cat chased the dog");
// Output: The lion chased the wolf

// Counting replacements (pass a variable by reference as 4th argument)
$count = 0;
str_replace("a", "o", "banana", $count);
echo $count;  // Output: 3  (three "a"s were replaced)

// Case-insensitive version
echo str_ireplace("WORLD", "PHP", "Hello World");  // Output: Hello PHP
?>
```

---

## Extracting Substrings

### `substr()`

- Extracts a **portion of a string** based on starting position and length.
- Syntax: `substr($string, $start, $length)`
- Works on **bytes** — same multibyte caveat as `strlen()` and `strrev()`.

```php
<?php
$text = "Hello World";

echo substr($text, 0, 5);    // Output: Hello   (start at 0, take 5 chars)
echo substr($text, 6);       // Output: World   (start at 6, take the rest)
echo substr($text, 6, 3);    // Output: Wor     (start at 6, take 3 chars)

// Negative start — counts from the END of the string
echo substr($text, -5);      // Output: World   (last 5 characters)
echo substr($text, -5, 3);   // Output: Wor     (3 chars starting 5 from the end)

// Negative length — stop that many characters before the end
echo substr($text, 0, -6);   // Output: Hello   (everything except last 6 chars)
?>
```

> ⚠️ **Warning:** `substr()` operates on **bytes**, not characters. Using it on Burmese, Chinese, Japanese, or emoji text will likely **cut a character in half**, producing garbled or broken output (often shown as `�`). Always use `mb_substr()` for non-ASCII text.

---

### `mb_substr()` — Multibyte Strings & Burmese

- The **multibyte-safe** version of `substr()` — operates on **characters**, not bytes.
- Essential for any text that isn't plain English — Burmese, Thai, Chinese, Japanese, Korean, Arabic, emoji, etc.
- Syntax: `mb_substr($string, $start, $length, $encoding)` — encoding defaults to UTF-8.

```php
<?php
$burmese = "မင်္ဂလာပါ";  // "Hello" in Burmese

// ❌ Using substr() on Burmese text — BROKEN
echo substr($burmese, 0, 3);
// Output: garbled/broken characters (cuts bytes mid-character)

// ✅ Using mb_substr() on Burmese text — CORRECT
echo mb_substr($burmese, 0, 3);
// Output: မင်္  (first 3 actual characters, correctly extracted)

echo mb_substr($burmese, 0, 1);   // Output: မ   (just the first character)
echo mb_substr($burmese, -2);     // Output: last 2 characters
?>
```

```php
<?php
// Comparing strlen() vs mb_strlen() on the same Burmese string
$name = "ဖြိုးမင်းပိုင်";  // A Burmese name

echo strlen($name);     // Output: e.g. 39  (bytes — NOT useful here)
echo mb_strlen($name);  // Output: e.g. 13  (actual character count) ✅

// Looping through Burmese characters correctly
$chars = mb_str_split($name);  // Splits into an array of actual characters
print_r($chars);
// Array of individual Burmese characters/glyphs, NOT broken bytes

foreach ($chars as $char) {
    echo $char . " | ";
}
?>
```

> 💡 **Golden Rule for Non-English Text:** Whenever working with Burmese, or any language outside basic English, always reach for the `mb_*` (multibyte) version of a string function: `mb_strlen()` instead of `strlen()`, `mb_substr()` instead of `substr()`, `mb_strtoupper()` instead of `strtoupper()`, and so on. Regular string functions assume single-byte characters and will corrupt multibyte text.

> ⚠️ **Setup Note:** The `mb_*` functions require the **mbstring extension**. It's enabled by default in most modern PHP installations, but if you get an "undefined function" error, install it: `sudo apt install php8.3-mbstring` (Ubuntu) — covered in the installation notes.

---

## Padding Strings

### `str_pad()`

- Pads (fills) a string to a certain **total length** using a specified character.
- Syntax: `str_pad($string, $length, $padString, $padType)`
- `$padType` options: `STR_PAD_RIGHT` (default), `STR_PAD_LEFT`, `STR_PAD_BOTH`.

```php
<?php
echo str_pad("5", 3, "0", STR_PAD_LEFT);    // Output: 005
echo str_pad("5", 3, "0", STR_PAD_RIGHT);   // Output: 500
echo str_pad("5", 5, "0", STR_PAD_BOTH);    // Output: 00500

// Practical use: zero-padding invoice numbers
$invoiceId = 42;
echo "INV-" . str_pad($invoiceId, 5, "0", STR_PAD_LEFT);
// Output: INV-00042

// Padding with multi-character strings
echo str_pad("Hi", 10, "-=");  // Output: Hi-=-=-=-=
?>
```

> 💡 **Tip:** `str_pad()` is great for formatting IDs, aligning text in console/CLI output, and generating fixed-width fields for reports or legacy file formats.

---

## Splitting & Joining

### `explode()`

- Splits a string into an **array** based on a delimiter.
- Syntax: `explode($delimiter, $string, $limit)`

```php
<?php
$csv = "apple,banana,cherry";
$fruits = explode(",", $csv);
print_r($fruits);
// Array ( [0] => apple [1] => banana [2] => cherry )

// Splitting a sentence into words
$sentence = "The quick brown fox";
$words = explode(" ", $sentence);
print_r($words);
// Array ( [0] => The [1] => quick [2] => brown [3] => fox )

// Using the limit parameter
$parts = explode(",", "a,b,c,d", 2);
print_r($parts);
// Array ( [0] => a [1] => b,c,d )   ← only splits into 2 parts max

// Practical use: parsing "key:value" pairs
$line = "name:Phyo";
[$key, $value] = explode(":", $line);
echo "$key => $value";  // Output: name => Phyo
?>
```

---

### `implode()`

- Joins an array's elements into a **single string**, separated by a delimiter.
- Also has an alias: `join()` (works identically).

```php
<?php
$fruits = ["apple", "banana", "cherry"];

echo implode(", ", $fruits);   // Output: apple, banana, cherry
echo implode(" - ", $fruits);  // Output: apple - banana - cherry
echo implode("", $fruits);     // Output: applebananacherry  (no separator)

// Practical use: building a CSV line from an array
$row = ["Phyo", "25", "Yangon"];
echo implode(",", $row);  // Output: Phyo,25,Yangon

// explode() and implode() together — round trip
$csv = "apple,banana,cherry";
$arr = explode(",", $csv);   // String → Array
$str = implode(" | ", $arr); // Array → String
echo $str;  // Output: apple | banana | cherry
?>
```

> 💡 **Tip:** `explode()` and `implode()` are a perfect pair — one breaks a string apart, the other puts pieces back together. Extremely common for handling CSV data, tags, comma-separated IDs, etc.

---

## Trimming Whitespace

### `trim()` / `ltrim()` / `rtrim()`

- `trim()` removes whitespace (or specified characters) from **both ends** of a string.
- `ltrim()` removes only from the **left** (start).
- `rtrim()` removes only from the **right** (end).
- By default, removes: spaces, tabs (`\t`), newlines (`\n`), carriage returns (`\r`), null bytes (`\0`), and vertical tabs (`\x0B`).

```php
<?php
$text = "   Hello World   ";

echo trim($text);    // Output: "Hello World"        (both sides trimmed)
echo ltrim($text);   // Output: "Hello World   "      (only left trimmed)
echo rtrim($text);   // Output: "   Hello World"      (only right trimmed)

// Trimming specific characters
echo trim("***Hello***", "*");      // Output: Hello
echo trim("##Title##", "#");        // Output: Title
echo trim("/path/to/file/", "/");   // Output: path/to/file

// Practical use: cleaning form input
$username = trim("  phyo123  ");
echo strlen($username);  // Output: 7  (no longer counting surrounding spaces)
?>
```

> 💡 **Tip:** Always `trim()` user-submitted form data before validating or storing it — users often accidentally include leading/trailing spaces, especially on mobile keyboards.

---

## HTML Escaping

### `htmlentities()`

- Converts **all applicable characters** to their HTML entities — protects against broken layout and basic XSS when outputting user content into HTML.
- Converts things like `<`, `>`, `&`, quotes, AND accented/special characters.

```php
<?php
$input = "<script>alert('XSS')</script>";
echo htmlentities($input);
// Output: &lt;script&gt;alert(&#039;XSS&#039;)&lt;/script&gt;
// (renders as literal text in the browser instead of executing as HTML/JS)

$text = "Café & Restaurant";
echo htmlentities($text);
// Output: Caf&eacute; &amp; Restaurant
?>
```

---

### `html_entity_decode()`

- The **reverse** of `htmlentities()` — converts HTML entities back into their original characters.

```php
<?php
$encoded = "&lt;b&gt;Bold&lt;/b&gt; &amp; Caf&eacute;";
echo html_entity_decode($encoded);
// Output: <b>Bold</b> & Café
?>
```

---

### `htmlspecialchars()`

- Similar to `htmlentities()`, but converts **only the essential HTML-special characters**: `< > & " '`
- This is the **more commonly recommended** function for general-purpose output escaping (faster, and sufficient for most security needs).

```php
<?php
$input = "<script>alert('XSS')</script>";
echo htmlspecialchars($input);
// Output: &lt;script&gt;alert(&#039;XSS&#039;)&lt;/script&gt;

// Practical, safe output of user-submitted data
$comment = $_POST["comment"] ?? "";
echo "<p>" . htmlspecialchars($comment, ENT_QUOTES, "UTF-8") . "</p>";
?>
```

### `htmlentities()` vs `htmlspecialchars()`

| Feature | `htmlentities()` | `htmlspecialchars()` |
|---|---|---|
| Escapes `< > & " '` | ✅ Yes | ✅ Yes |
| Escapes accented chars (é, ü, etc.) | ✅ Yes | ❌ No |
| Performance | Slightly slower | Slightly faster |
| Common use | Full HTML entity conversion | General security escaping (most common choice) |

> ⚠️ **Security Reminder:** Always escape user-generated content before outputting it into HTML to prevent **XSS (Cross-Site Scripting)** attacks. `htmlspecialchars()` is the standard go-to for this in most PHP applications.

---

## Hashing Strings

### `md5()`

- Generates a **128-bit hash** (as a 32-character hexadecimal string) from input data.
- A **hash** is a one-way fingerprint — you cannot reverse it back to the original text.
- Same input **always** produces the same hash output.

```php
<?php
echo md5("hello");           // Output: 5d41402abc4b2a76b9719d911017c592
echo md5("Hello");           // Output: 8b1a9953c4611296a827abf8c47804d7  (different! case-sensitive)
echo md5("hello");           // Output: 5d41402abc4b2a76b9719d911017c592  (same input = same hash, always)

// Practical use: generating a unique cache key
$cacheKey = md5("user_profile_" . $userId);

// Practical use: checking file integrity
$fileHash = md5_file("document.pdf");
?>
```

> ⚠️ **Critical Security Warning:** **Never use `md5()` (or `sha1()`) to hash passwords.** MD5 is fast and was never designed for password security — it's vulnerable to brute-force and "rainbow table" attacks. For passwords, **always use `password_hash()`** and `password_verify()` instead, which use strong, slow, salted algorithms designed specifically for this purpose.

```php
<?php
// ❌ NEVER do this for passwords
$hashedPassword = md5($password);  // Insecure!

// ✅ Correct way to hash passwords
$hashedPassword = password_hash($password, PASSWORD_DEFAULT);

// ✅ Correct way to verify a password
if (password_verify($enteredPassword, $hashedPassword)) {
    echo "Password correct!";
}
?>
```

> 💡 **Appropriate uses for `md5()`:** Generating cache keys, checksums for file integrity, generating non-security-critical unique identifiers. **Inappropriate use:** passwords, security tokens, anything sensitive.

---

## Multibyte Functions Reference (`mb_*`)

- A quick reference of common multibyte equivalents — use these whenever working with Burmese or other non-Latin scripts.

| Standard Function | Multibyte-Safe Version | Use For |
|---|---|---|
| `strlen()` | `mb_strlen()` | Counting actual characters |
| `substr()` | `mb_substr()` | Extracting parts of a string |
| `strtolower()` | `mb_strtolower()` | Lowercasing (mainly for accented Latin) |
| `strtoupper()` | `mb_strtoupper()` | Uppercasing (mainly for accented Latin) |
| `strpos()` | `mb_strpos()` | Finding character position (not byte position) |
| `str_split()` | `mb_str_split()` | Splitting into an array of characters |
| `wordwrap()` | `mb_strimwidth()` | Truncating to a display width |
| (n/a) | `mb_convert_encoding()` | Converting between character encodings |
| (n/a) | `mb_detect_encoding()` | Detecting the encoding of a string |

```php
<?php
$burmese = "မြန်မာစာ";  // "Burmese language" in Burmese

echo mb_strlen($burmese);           // Character count (correct)
echo mb_substr($burmese, 0, 2);     // First 2 characters (correct)
print_r(mb_str_split($burmese));    // Array of individual characters (correct)

// Always set encoding explicitly to avoid ambiguity
mb_internal_encoding("UTF-8");
?>
```

> 💡 **Best Practice:** At the top of any PHP project handling multilingual text (especially Burmese, Thai, Chinese, Japanese, Korean, Arabic), set `mb_internal_encoding("UTF-8");` once, and consistently use `mb_*` functions throughout your codebase.

---

## Quick Revision

- **`strlen()`** counts **bytes**; for Burmese/multibyte text use **`mb_strlen()`** to count actual characters.
- **`str_word_count()`** counts words — reliable for English, not for Burmese (no consistent space-based word boundaries).
- **`strtolower()` / `strtoupper()`** change case (ASCII); **`ucfirst()`** capitalizes the first letter, **`ucwords()`** capitalizes every word.
- **`strrev()`** reverses bytes — breaks multibyte text; there's no safe direct multibyte reverse function.
- **`strpos()`** finds the first occurrence; **always check `=== false`**, never `== false` or `!$pos`, because a match at position `0` is falsy.
- **`str_replace()`** replaces text (case-sensitive); use **`str_ireplace()`** for case-insensitive replacement.
- **`substr()`** extracts by byte position — breaks multibyte text. **`mb_substr()`** is the character-safe version — **always use it for Burmese**.
- **`str_pad()`** pads a string to a fixed length — great for IDs like `INV-00042`.
- **`explode()`** splits a string into an array; **`implode()`** joins an array back into a string. They're a matched pair.
- **`trim()` / `ltrim()` / `rtrim()`** remove whitespace (or specified characters) from both/left/right ends — always trim form input.
- **`htmlentities()`** escapes ALL special + accented characters; **`htmlspecialchars()`** escapes only `< > & " '` and is the more common choice for security. Both prevent **XSS**.
- **`html_entity_decode()`** reverses `htmlentities()`/`htmlspecialchars()` encoding.
- **`md5()`** creates a one-way hash — useful for cache keys and file checksums, but **never for passwords**. Use `password_hash()`/`password_verify()` for passwords instead.
- **Golden Rule:** for any text that isn't plain English (Burmese, Chinese, Japanese, etc.), always use the **`mb_*`** version of string functions to avoid corrupting multibyte characters.