# PHP Basics — What is PHP, What Can It Do & Basic Syntax

A **PHP script** is a file containing PHP code that the server processes before sending the result to the browser. Understanding what PHP is and how its syntax works is the foundation of everything else you'll write.

---

## Table of Contents

1. [What is PHP?](#what-is-php)
2. [What Can PHP Do?](#what-can-php-do)
3. [Basic Syntax](#basic-syntax)
   - [PHP Tags](#php-tags)
   - [Escaping from HTML](#escaping-from-html)
   - [Statements & Semicolons](#statements--semicolons)
   - [Whitespace & Line Breaks](#whitespace--line-breaks)
   - [Comments](#comments)
   - [Case Sensitivity](#case-sensitivity)
4. [Quick Revision](#quick-revision)

---

## What is PHP?

- **PHP** stands for **PHP: Hypertext Preprocessor** (a recursive acronym).
- It is a **server-side scripting language** — the code runs on the server, not in the user's browser.
- PHP generates **HTML output** that is then sent to the browser.
- It is **open-source** and **free** to use.
- PHP is embedded directly inside HTML files using special tags.

### How PHP Works (Request Flow)

```
Browser Request → Web Server → PHP Engine processes .php file → HTML Output → Browser
```

- The browser **never sees** the raw PHP code — only the final HTML result.
- This is different from JavaScript, which runs **in the browser**.

---

## What Can PHP Do?

- **Generate dynamic HTML pages** — content changes based on user, time, data, etc.
- **Handle forms** — collect and validate user input from HTML forms.
- **Work with databases** — read, write, update, and delete data (MySQL, PostgreSQL, etc.).
- **Manage files** — create, read, write, and delete files on the server.
- **Handle cookies & sessions** — track users across multiple pages.
- **Send emails** — trigger emails from a server using `mail()` or libraries like PHPMailer.
- **Authenticate users** — manage login systems, passwords, and access control.
- **Communicate with APIs** — send and receive JSON data from external services.
- **Generate non-HTML content** — output PDFs, images, CSV files, and more.

> 💡 **PHP powers a huge portion of the web** — WordPress, Facebook (originally), Wikipedia, and Laravel apps all run on PHP.

---

## Basic Syntax

### PHP Tags

- PHP code must be wrapped inside **PHP tags** so the server knows what to process.
- Everything outside PHP tags is treated as plain HTML and sent to the browser as-is.

```php
<?php
// Your PHP code goes here
echo "Hello, World!";
?>
```

#### Tag Types

| Tag | Name | Use |
|---|---|---|
| `<?php ?>` | Standard tag | Always use this — works everywhere ✅ |
| `<?= ?>` | Short echo tag | Shortcut for `<?php echo ?>` ✅ |
| `<? ?>` | Short open tag | Avoid — disabled on many servers ❌ |

```php
<!-- Standard tag -->
<?php echo "Hello"; ?>

<!-- Short echo tag (shortcut for echo) -->
<?= "Hello" ?>

<!-- These two lines produce the same output -->
```

> ⚠️ **Warning:** Always use `<?php ?>` or `<?= ?>`. Short open tags `<? ?>` can be disabled in `php.ini` and will break your code on some servers.

---

### Escaping from HTML

- **Escaping from HTML** means switching from HTML mode into PHP mode using `<?php` and back with `?>`.
- You can jump in and out of PHP mode as many times as you need inside one file.
- Everything between the PHP tags is executed; everything outside is output directly.

```php
<!DOCTYPE html>
<html>
<body>

  <h1>Welcome</h1>

  <?php
    $name = "Phyo";
    echo "<p>Hello, $name!</p>";
  ?>

  <p>This is plain HTML again.</p>

  <?php echo "<p>Back in PHP!</p>"; ?>

</body>
</html>
```

Output in browser:
```
Welcome
Hello, Phyo!
This is plain HTML again.
Back in PHP!
```

> 💡 **Tip:** The PHP engine only processes what's inside `<?php ?>`. Everything else passes through untouched.

---

### Statements & Semicolons

- Every PHP **statement** must end with a **semicolon** `;`.
- A statement is one complete instruction — like printing text or assigning a variable.
- Missing a semicolon is one of the most common beginner errors.

```php
<?php
echo "Hello";       // ✅ Correct
echo "World";       // ✅ Correct

echo "Hello"        // ❌ Missing semicolon — causes a parse error
?>
```

> ⚠️ **Common Mistake:** Forgetting the semicolon at the end of a statement causes a `Parse error` and stops the entire script from running.

> 💡 **Exception:** The very last statement before `?>` doesn't technically require a semicolon, but always add one anyway for consistency.

---

### Whitespace & Line Breaks

- PHP **ignores extra whitespace** (spaces, tabs, newlines) between statements.
- You can format your code however you like for readability — it won't affect the output.

```php
<?php
// These are all equivalent
echo "Hello";
echo     "Hello";
echo
  "Hello";
?>
```

> 💡 **Tip:** Use consistent indentation (2 or 4 spaces) to keep your code readable. PHP doesn't care, but future-you will.

---

### Comments

- **Comments** are notes in your code that PHP ignores completely — they never appear in the browser output.
- Use them to explain your logic or temporarily disable code.

```php
<?php
// This is a single-line comment

# This is also a single-line comment (less common)

/*
  This is a
  multi-line comment
*/

echo "Hello"; // You can also comment at the end of a line
?>
```

| Style | Syntax | Use case |
|---|---|---|
| Single-line | `// comment` | Short notes, inline explanations |
| Single-line (alt) | `# comment` | Same as `//`, less commonly used |
| Multi-line | `/* comment */` | Longer explanations, disabling blocks of code |

> 💡 **Tip:** Comment your code as you write it — not after. It's much faster when the logic is fresh in your mind.

---

### Case Sensitivity

- PHP is **partially case-sensitive** — the rules differ depending on what you're writing.

| What | Case Sensitive? | Example |
|---|---|---|
| Variables | ✅ Yes | `$name` and `$Name` are **different** variables |
| Functions | ❌ No | `echo()`, `ECHO()`, `Echo()` all work the same |
| Keywords | ❌ No | `if`, `IF`, `If` are all valid |
| Classes | ❌ No | `new Car()` and `new car()` refer to the same class |

```php
<?php
$name = "Phyo";
$Name = "Min";

echo $name; // Output: Phyo
echo $Name; // Output: Min  ← completely different variable!

ECHO "Hello"; // ✅ Works fine — functions are case-insensitive
?>
```

> ⚠️ **Warning:** Variable names are case-sensitive. `$user`, `$User`, and `$USER` are three separate variables. This is a very common source of bugs.

---

## Quick Revision

- PHP is a **server-side** language — it runs on the server and sends HTML to the browser. The browser never sees PHP code.
- PHP can handle **forms, databases, files, sessions, emails, APIs**, and generate non-HTML content like PDFs.
- Always wrap PHP code in `<?php ?>` tags — avoid short open tags `<? ?>`.
- Use `<?= ?>` as a shorthand for `<?php echo ?>`.
- **Escaping from HTML** means switching in and out of PHP mode inside a single file — everything outside PHP tags is sent as plain HTML.
- Every PHP statement ends with a **semicolon** `;` — missing one causes a `Parse error`.
- PHP **ignores whitespace** — use it freely for clean, readable formatting.
- Use `//` for single-line comments and `/* */` for multi-line comments.
- **Variables are case-sensitive** (`$name` ≠ `$Name`), but **functions and keywords are not** (`echo` = `ECHO`).