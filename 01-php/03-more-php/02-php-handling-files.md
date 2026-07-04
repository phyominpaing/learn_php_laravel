# PHP Files — Combining, File System & Uploads

PHP has deep built-in support for working with files — from combining scripts together, to reading and writing files on disk, to handling what users upload from a browser. This note covers all three areas in depth.

---

## Table of Contents

1. [Combining Files](#combining-files)
   - [`require()`](#require)
   - [`include()`](#include)
   - [`require_once()`](#require_once)
   - [`include_once()`](#include_once)
   - [Comparison Table](#combining-files-comparison)
2. [File System Functions](#file-system-functions)
   - [`touch()` / `mkdir()`](#touch--mkdir)
   - [`is_dir()` / `is_file()`](#is_dir--is_file)
   - [`file_exists()`](#file_exists)
   - [`scandir()`](#scandir)
   - [`unlink()` / `rmdir()`](#unlink--rmdir)
   - [`pathinfo()` / `dirname()`](#pathinfo--dirname)
   - [`copy()` / `rename()`](#copy--rename)
   - [`fopen()`](#fopen)
   - [`fread()` / `file()`](#fread--file)
   - [`fgetc()` / `fgets()` / `feof()`](#fgetc--fgets--feof)
   - [`fwrite()`](#fwrite)
   - [`fclose()`](#fclose)
3. [Handling File Uploads](#handling-file-uploads)
   - [POST Method & Enctype](#post-method--enctype)
   - [`$_FILES`](#_files)
   - [Multiple File Uploads](#multiple-file-uploads)
   - [`move_uploaded_file()`](#move_uploaded_file)
4. [Quick Revision](#quick-revision)

---

## Combining Files

PHP lets you split your code into multiple files and combine them on demand. This is the foundation of every PHP project structure — config in one file, database logic in another, functions in another, and so on. No more writing everything in one giant file.

---

### `require()`

- Loads and executes another PHP file into the current script.
- If the file **does not exist** or cannot be loaded → **Fatal Error** — the entire script stops immediately.
- Use `require()` when the file is **essential** — without it, the script cannot continue.

```php
<?php
// project structure:
// /config/database.php
// /index.php

// config/database.php
<?php
define("DB_HOST", "localhost");
define("DB_USER", "root");
define("DB_PASS", "secret");
define("DB_NAME", "myapp");
```

```php
<?php
// index.php
require("config/database.php");   // Loads database.php into this script

echo DB_HOST;  // Output: localhost  (constant from the required file is now available)

// If config/database.php doesn't exist:
// Fatal error: require(): Failed opening required 'config/database.php'
// → Script STOPS completely
?>
```

> 💡 **Best practice:** Always use the `__DIR__` magic constant for reliable paths. Without it, PHP resolves relative paths from wherever the script is *called* from — not where it *lives*.

```php
<?php
// ❌ Relative path — breaks if called from a different directory
require("config/database.php");

// ✅ Absolute path using __DIR__ — always resolves relative to THIS file's location
require(__DIR__ . "/config/database.php");
?>
```

---

### `include()`

- Exactly like `require()` — loads and executes a file.
- The difference: if the file **does not exist** → only a **Warning** — the script continues running.
- Use `include()` when the file is **optional** — the page can still work without it.

```php
<?php
// Optional sidebar component — page still works without it
include(__DIR__ . "/components/sidebar.php");

echo "This still runs even if sidebar.php is missing";

// If sidebar.php is missing:
// Warning: include(): Failed opening 'components/sidebar.php'
// → Script CONTINUES (just the sidebar is absent)
?>
```

```php
<?php
// A classic PHP template example using include
include(__DIR__ . "/partials/header.php");
?>

<main>
    <h1>Welcome to My Site</h1>
    <p>This is the page content.</p>
</main>

<?php
include(__DIR__ . "/partials/footer.php");
?>
```

---

### `require_once()`

- Same as `require()` but guarantees the file is **only loaded once**, no matter how many times `require_once` is called for the same file.
- Prevents "cannot redeclare function/class" errors when a file is included multiple times across a complex project.

```php
<?php
// functions.php
function greet(string $name): string {
    return "Hello, $name!";
}
```

```php
<?php
// Without _once — if two files both require functions.php, it gets loaded twice
require(__DIR__ . "/functions.php");
require(__DIR__ . "/functions.php");
// ❌ Fatal Error: Cannot redeclare greet() — function already exists!

// With _once — safe no matter how many times it appears
require_once(__DIR__ . "/functions.php");
require_once(__DIR__ . "/functions.php");  // ← Second call is ignored silently ✅
echo greet("Phyo");  // Output: Hello, Phyo!
?>
```

```php
<?php
// Real project example: multiple files all require the same config
// config.php
require_once(__DIR__ . "/config/database.php");   // Loaded once
require_once(__DIR__ . "/config/constants.php");   // Loaded once

// models/User.php
require_once(__DIR__ . "/../config/database.php"); // Already loaded — ignored ✅

// models/Product.php
require_once(__DIR__ . "/../config/database.php"); // Already loaded — ignored ✅
?>
```

---

### `include_once()`

- Same as `include()` but only loads the file **once**, like `require_once()`.
- If the file is missing → only a **Warning** (doesn't stop the script).

```php
<?php
include_once(__DIR__ . "/helpers/format.php");
include_once(__DIR__ . "/helpers/format.php");  // Ignored — already included

include_once(__DIR__ . "/helpers/optional.php");  // Missing → Warning, script continues
?>
```

---

### Combining Files Comparison

| Function | File Missing | Loads Multiple Times | Use When |
|---|---|---|---|
| `require()` | ❌ Fatal Error — stops | Yes — can load twice | File is essential, loaded once |
| `include()` | ⚠️ Warning — continues | Yes — can load twice | File is optional, loaded once |
| `require_once()` | ❌ Fatal Error — stops | No — loaded only once | File is essential, may be included from multiple places |
| `include_once()` | ⚠️ Warning — continues | No — loaded only once | File is optional, may be included from multiple places |

> 💡 **Rule of thumb:** Use `require_once()` for classes, functions, and config files (essential, must-not-duplicate). Use `include()` for template parts like headers and footers (optional, loaded once by definition).

---

## File System Functions

PHP has a comprehensive set of functions for creating, reading, moving, and deleting files and directories — no shell scripts needed.

---

### `touch()` / `mkdir()`

- `touch($path)` — creates an **empty file** if it doesn't exist; if it does exist, updates its last-modified timestamp.
- `mkdir($path, $permissions, $recursive)` — creates a **directory**.

```php
<?php
// touch() — create an empty file or update its timestamp
touch("logs/app.log");      // Creates logs/app.log if it doesn't exist
var_dump(file_exists("logs/app.log"));  // bool(true)

// Also works as a "timestamp" updater
touch("cache/data.cache");  // If the file exists, updates its modified time
echo date("Y-m-d H:i:s", filemtime("cache/data.cache"));  // Shows current time


// mkdir() — create a directory
mkdir("uploads");           // Creates single directory
mkdir("path/to/nested/dir", 0755, true);  // Creates all nested directories at once
//                           ↑ Permissions  ↑ recursive=true → creates parents too

// Check before creating to avoid a warning
if (!is_dir("storage/logs")) {
    mkdir("storage/logs", 0755, true);
    echo "Directory created";
}
?>
```

---

### `is_dir()` / `is_file()`

- `is_dir($path)` — returns `true` if the path is a **directory**.
- `is_file($path)` — returns `true` if the path is a **regular file** (not a directory).

```php
<?php
$path1 = "/var/www/html";
$path2 = "/var/www/html/index.php";
$path3 = "/var/www/html/does-not-exist";

var_dump(is_dir($path1));   // bool(true)   — it's a directory
var_dump(is_dir($path2));   // bool(false)  — it's a file, not a directory
var_dump(is_dir($path3));   // bool(false)  — doesn't exist

var_dump(is_file($path1));  // bool(false)  — it's a directory, not a file
var_dump(is_file($path2));  // bool(true)   — it's a file
var_dump(is_file($path3));  // bool(false)  — doesn't exist

// Practical use: handle differently based on type
function inspect(string $path): string {
    if (is_dir($path))  return "$path is a directory";
    if (is_file($path)) return "$path is a file";
    return "$path does not exist";
}

echo inspect("/var/www/html");          // /var/www/html is a directory
echo inspect("/var/www/html/index.php"); // /var/www/html/index.php is a file
?>
```

---

### `file_exists()`

- Returns `true` if a file **or** directory exists at the given path.
- Doesn't care whether it's a file or directory — just checks existence.

```php
<?php
// Check if a file exists before reading it
$configPath = __DIR__ . "/config/settings.php";

if (file_exists($configPath)) {
    require($configPath);
    echo "Config loaded";
} else {
    die("Config file missing — cannot continue");
}

// Difference: file_exists() vs is_file() vs is_dir()
$path = "/var/www";
var_dump(file_exists($path)); // true  — exists (it's a directory)
var_dump(is_file($path));     // false — it's not a file
var_dump(is_dir($path));      // true  — it's a directory

// file_exists() is also useful before writing to avoid accidental overwrites
if (!file_exists("backup.sql")) {
    file_put_contents("backup.sql", $databaseBackup);
}
?>
```

---

### `scandir()`

- Returns an **array** of all files and directories inside a given directory.
- Always includes `.` (current directory) and `..` (parent directory) — filter these out.

```php
<?php
// List everything in the uploads directory
$items = scandir(__DIR__ . "/uploads");
print_r($items);
// Array (
//   [0] => .
//   [1] => ..
//   [2] => photo.jpg
//   [3] => document.pdf
//   [4] => avatar.png
// )

// Filter out . and .. to get only real items
$files = array_diff(scandir(__DIR__ . "/uploads"), [".", ".."]);
print_r(array_values($files));
// Array ( [0] => photo.jpg [1] => document.pdf [2] => avatar.png )

// Practical use: list only files (no subdirectories)
$uploadDir = __DIR__ . "/uploads";
$onlyFiles = array_filter(
    array_diff(scandir($uploadDir), [".", ".."]),
    fn($item) => is_file("$uploadDir/$item")
);
print_r(array_values($onlyFiles));

// Sorted in reverse order (descending by name)
$reversed = scandir(__DIR__ . "/uploads", SCANDIR_SORT_DESCENDING);
?>
```

---

### `unlink()` / `rmdir()`

- `unlink($file)` — **deletes a file** from disk.
- `rmdir($dir)` — **removes a directory** — but only if it's **empty**!

```php
<?php
// Delete a single file
if (file_exists("old-data.csv")) {
    unlink("old-data.csv");
    echo "File deleted";
}

// Delete a directory (must be empty first)
rmdir("empty-folder");

// ❌ Cannot remove non-empty directory with rmdir()
// rmdir("uploads");  // Warning: Directory not empty

// ✅ Delete a directory and ALL its contents (recursive)
function deleteDirectory(string $dir): bool {
    if (!is_dir($dir)) return false;

    $items = array_diff(scandir($dir), [".", ".."]);

    foreach ($items as $item) {
        $path = "$dir/$item";
        is_dir($path) ? deleteDirectory($path) : unlink($path);
    }

    return rmdir($dir);  // Now empty — safe to remove
}

deleteDirectory(__DIR__ . "/temp/old-cache");
?>
```

> ⚠️ **Warning:** `unlink()` and `rmdir()` are **permanent** — there is no Recycle Bin. Double-check paths before deleting, especially when dealing with user-provided paths. Never use raw user input as a file path.

---

### `pathinfo()` / `dirname()`

- `pathinfo($path)` — splits a file path into its components (directory, filename, extension).
- `dirname($path)` — returns just the directory portion of a path.

```php
<?php
$path = "/var/www/html/uploads/photo.jpg";

// pathinfo() — returns an array with all path components
$info = pathinfo($path);
print_r($info);
// Array (
//   [dirname]   => /var/www/html/uploads
//   [basename]  => photo.jpg
//   [extension] => jpg
//   [filename]  => photo    ← filename without extension
// )

// Access individual components
echo pathinfo($path, PATHINFO_DIRNAME);   // /var/www/html/uploads
echo pathinfo($path, PATHINFO_BASENAME);  // photo.jpg
echo pathinfo($path, PATHINFO_EXTENSION); // jpg
echo pathinfo($path, PATHINFO_FILENAME);  // photo


// dirname() — get directory from path
echo dirname($path);                   // /var/www/html/uploads
echo dirname($path, 2);               // /var/www/html  (go 2 levels up)
echo dirname(__FILE__);               // Directory of the current file

// basename() — get filename from path
echo basename($path);                  // photo.jpg
echo basename($path, ".jpg");          // photo  (strips the .jpg extension)

// Practical use: getting a file's extension for validation
$uploadedFile = $_FILES["photo"]["name"] ?? "";
$extension    = strtolower(pathinfo($uploadedFile, PATHINFO_EXTENSION));
$allowed      = ["jpg", "jpeg", "png", "webp"];

if (!in_array($extension, $allowed, true)) {
    die("File type not allowed");
}
?>
```

---

### `copy()` / `rename()`

- `copy($source, $destination)` — **copies** a file (source remains intact).
- `rename($oldPath, $newPath)` — **moves** a file or renames it (original no longer exists at old path).

```php
<?php
// copy() — duplicate a file
copy("template.html", "backup/template-backup.html");

// Source still exists!
var_dump(file_exists("template.html"));  // bool(true)

// rename() — move or rename a file
rename("old-name.php", "new-name.php");   // Rename in same directory
rename("uploads/temp.jpg", "public/images/photo.jpg");  // Move to different directory

// Source is GONE after rename
var_dump(file_exists("old-name.php"));  // bool(false)

// Practical use: move a just-uploaded file to a processed folder
$source = "uploads/raw/image.jpg";
$dest   = "uploads/processed/image.jpg";

if (file_exists($source)) {
    rename($source, $dest);
    echo "File moved";
}

// Creating a dated backup
$original = "data/users.csv";
$backup   = "backups/users-" . date("Y-m-d") . ".csv";
copy($original, $backup);
echo "Backup created: $backup";
?>
```

---

### `fopen()`

- Opens a file (or URL) and returns a **file handle resource** — a pointer to the file that other file functions use.
- Think of it as "opening a door" — you open the file first, then read/write through the handle.
- The **mode** argument controls whether you open for reading, writing, appending, etc.

```php
<?php
$handle = fopen("data.txt", "r");  // Open for reading
// $handle is now a resource — use it with fread(), fgets(), fwrite(), etc.
fclose($handle);  // Close when done
```

### File Open Modes

| Mode | Meaning | File Must Exist? | Pointer |
|---|---|---|---|
| `"r"` | Read only | ✅ Yes | Beginning |
| `"r+"` | Read and write | ✅ Yes | Beginning |
| `"w"` | Write only — truncates file to 0 | ❌ Creates if needed | Beginning |
| `"w+"` | Read and write — truncates to 0 | ❌ Creates if needed | Beginning |
| `"a"` | Append — writes at end | ❌ Creates if needed | End |
| `"a+"` | Read and append | ❌ Creates if needed | End |
| `"x"` | Write — fails if file exists | ❌ Must NOT exist | Beginning |
| `"x+"` | Read/write — fails if file exists | ❌ Must NOT exist | Beginning |

```php
<?php
// Examples of different modes
$readHandle   = fopen("data.txt", "r");   // Open to read
$writeHandle  = fopen("log.txt", "w");    // Open to write (overwrites!)
$appendHandle = fopen("log.txt", "a");    // Open to append (adds to end)

// Opening a URL to read remote content
$urlHandle    = fopen("https://api.example.com/data.json", "r");
?>
```

---

### `fread()` / `file()`

- `fread($handle, $length)` — reads **a specified number of bytes** from the file handle.
- `file($path)` — reads an entire file into an **array** (one element per line). No `fopen()` needed.

```php
<?php
// fread() — read a specific chunk
$handle  = fopen("data.txt", "r");
$content = fread($handle, filesize("data.txt"));  // Read the whole file at once
fclose($handle);
echo $content;

// Read in chunks (useful for large files — doesn't load whole file into memory)
$handle = fopen("large-file.csv", "r");
while (!feof($handle)) {
    $chunk = fread($handle, 4096);  // Read 4KB at a time
    echo $chunk;
}
fclose($handle);


// file() — one-liner to get all lines as an array
$lines = file("data.txt");
print_r($lines);
// Array (
//   [0] => "Line 1\n"
//   [1] => "Line 2\n"
//   [2] => "Line 3\n"
// )

// Strip line endings with FILE_IGNORE_NEW_LINES
$lines = file("data.txt", FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
print_r($lines);
// Array ( [0] => Line 1  [1] => Line 2  [2] => Line 3 )

// file_get_contents() — simplest way to read a whole file as one string
$content = file_get_contents("data.txt");
echo $content;

// file_get_contents() also works with URLs (like curl, but simpler)
$json = file_get_contents("https://api.example.com/users");
$data = json_decode($json, true);
?>
```

---

### `fgetc()` / `fgets()` / `feof()`

- `fgetc($handle)` — reads **one character** at a time from the file handle.
- `fgets($handle)` — reads **one line** at a time from the file handle.
- `feof($handle)` — returns `true` when the **end of file** is reached (used in `while` loop conditions).

```php
<?php
// fgets() — most commonly used — reads line by line
$handle = fopen("users.csv", "r");

while (!feof($handle)) {
    $line = fgets($handle);  // Read one line including the \n
    if ($line !== false) {
        $parts = explode(",", trim($line));
        echo "Name: {$parts[0]}, Email: {$parts[1]}\n";
    }
}

fclose($handle);
// Output (for a CSV with: Phyo,phyo@example.com):
// Name: Phyo, Email: phyo@example.com


// fgetc() — read character by character (less common — very granular)
$handle = fopen("data.txt", "r");

while (!feof($handle)) {
    $char = fgetc($handle);
    if ($char !== false) {
        echo $char;  // Outputs one character at a time
    }
}

fclose($handle);
?>
```

---

### `fwrite()`

- Writes a string to a file handle opened for writing or appending.
- Returns the **number of bytes written** or `false` on failure.

```php
<?php
// Write to a file (overwrites if "w" mode)
$handle = fopen("output.txt", "w");
fwrite($handle, "Hello, World!\n");
fwrite($handle, "Second line\n");
fclose($handle);
// output.txt contains:
// Hello, World!
// Second line


// Append to a file (adds to end without erasing existing content)
$handle = fopen("log.txt", "a");
$entry  = date("[Y-m-d H:i:s]") . " User logged in\n";
fwrite($handle, $entry);
fclose($handle);


// file_put_contents() — one-liner alternative to fopen + fwrite + fclose
file_put_contents("output.txt", "Hello, World!\n");           // Overwrite
file_put_contents("log.txt", "New entry\n", FILE_APPEND);      // Append
file_put_contents("log.txt", "Another\n", FILE_APPEND | LOCK_EX);  // Append + lock

// Practical: writing a JSON cache file
$data = ["users" => [["name" => "Phyo"], ["name" => "Alice"]]];
file_put_contents("cache/users.json", json_encode($data, JSON_PRETTY_PRINT));
?>
```

---

### `fclose()`

- **Closes** the file handle, freeing the resource.
- Always close file handles when you're done — open file handles consume system resources, and on some systems can lock the file for other processes.

```php
<?php
$handle = fopen("data.txt", "r");
// ... read or write operations ...
fclose($handle);  // Always close!

// Complete file read/write cycle — the proper pattern
$handle = fopen("log.txt", "a");

if ($handle === false) {
    die("Could not open log file");
}

fwrite($handle, date("[Y-m-d H:i:s]") . " Error occurred\n");
fclose($handle);  // Free the resource


// The full low-level file reading pattern
function readFileLineByLine(string $path): array {
    $lines  = [];
    $handle = fopen($path, "r");

    if ($handle === false) {
        throw new \RuntimeException("Cannot open file: $path");
    }

    while (!feof($handle)) {
        $line = fgets($handle);
        if ($line !== false) {
            $lines[] = trim($line);
        }
    }

    fclose($handle);
    return array_filter($lines);  // Remove empty lines
}
?>
```

### `fopen` Functions Summary

| Function | Does | Needs `fopen()`? |
|---|---|---|
| `fopen($path, $mode)` | Opens a file, returns handle | — (this IS the opener) |
| `fread($handle, $len)` | Reads N bytes | ✅ Yes |
| `fgets($handle)` | Reads one line | ✅ Yes |
| `fgetc($handle)` | Reads one character | ✅ Yes |
| `fwrite($handle, $str)` | Writes a string | ✅ Yes |
| `feof($handle)` | Check if end of file | ✅ Yes |
| `fclose($handle)` | Close and free the handle | ✅ Yes |
| `file_get_contents($path)` | Read whole file as string | ❌ No (one-liner) |
| `file_put_contents($path, $str)` | Write whole file | ❌ No (one-liner) |
| `file($path)` | Read file into array of lines | ❌ No (one-liner) |

> 💡 **Tip:** For most everyday tasks, use the one-liner convenience functions: `file_get_contents()`, `file_put_contents()`, and `file()`. Use `fopen()`-based functions when you need more control — like reading a huge file in chunks, or streaming a response.

---

## Handling File Uploads

File uploads are one of the most common and most dangerous features in PHP. Done correctly they are powerful; done incorrectly they open your server to serious security vulnerabilities.

---

### POST Method & Enctype

- File uploads **must** use `method="POST"` on the form — `GET` cannot carry a file.
- The form **must** have `enctype="multipart/form-data"` — without this, PHP receives only the filename as a string, not the actual file content.

```html
<!-- ❌ Without enctype — file won't upload, only the filename is sent -->
<form method="POST" action="upload.php">
    <input type="file" name="photo">
    <button>Upload</button>
</form>

<!-- ✅ Correct — multipart/form-data allows binary file data -->
<form method="POST" action="upload.php" enctype="multipart/form-data">
    <input type="file" name="photo">
    <button>Upload</button>
</form>
```

> 💡 **What `multipart/form-data` does:** It splits the request body into multiple "parts" — one per form field — and can handle binary data (file bytes) alongside regular text fields. Without it, the form uses `application/x-www-form-urlencoded` which only handles text.

---

### `$_FILES`

- `$_FILES["fieldname"]` is a nested array containing all information about the uploaded file.
- Populated **automatically** by PHP when a file upload request arrives.

```php
<?php
print_r($_FILES["photo"]);
// Array (
//   [name]     => profile-picture.jpg       ← original filename from user's computer
//   [type]     => image/jpeg                 ← MIME type reported by BROWSER (do NOT trust!)
//   [tmp_name] => /tmp/php7f83AB             ← temporary location PHP saved it to
//   [error]    => 0                          ← 0 = UPLOAD_ERR_OK (no error)
//   [size]     => 524288                     ← size in bytes (512 KB)
// )
?>
```

### Error Codes in `$_FILES["fieldname"]["error"]`

| Code | Constant | Meaning |
|---|---|---|
| `0` | `UPLOAD_ERR_OK` | No error — upload succeeded |
| `1` | `UPLOAD_ERR_INI_SIZE` | File exceeds `upload_max_filesize` in `php.ini` |
| `2` | `UPLOAD_ERR_FORM_SIZE` | File exceeds `MAX_FILE_SIZE` in the HTML form |
| `3` | `UPLOAD_ERR_PARTIAL` | File was only partially uploaded |
| `4` | `UPLOAD_ERR_NO_FILE` | No file was selected |
| `6` | `UPLOAD_ERR_NO_TMP_DIR` | Missing temporary folder |
| `7` | `UPLOAD_ERR_CANT_WRITE` | Failed to write file to disk |
| `8` | `UPLOAD_ERR_EXTENSION` | Upload stopped by a PHP extension |

```php
<?php
// upload.php — a safe, complete single file upload handler
if ($_SERVER["REQUEST_METHOD"] !== "POST") {
    die("POST only");
}

$file     = $_FILES["photo"] ?? null;
$uploadTo = __DIR__ . "/uploads/";

// Step 1 — check a file was actually sent
if ($file === null || $file["error"] === UPLOAD_ERR_NO_FILE) {
    die("No file uploaded.");
}

// Step 2 — check for upload errors
if ($file["error"] !== UPLOAD_ERR_OK) {
    $errorMessages = [
        UPLOAD_ERR_INI_SIZE  => "File too large (exceeds server limit)",
        UPLOAD_ERR_FORM_SIZE => "File too large (exceeds form limit)",
        UPLOAD_ERR_PARTIAL   => "File only partially uploaded",
        UPLOAD_ERR_NO_TMP_DIR => "Server has no temp directory",
        UPLOAD_ERR_CANT_WRITE => "Server cannot write to disk",
    ];
    die($errorMessages[$file["error"]] ?? "Unknown upload error");
}

// Step 3 — validate file size (client-side is bypassable — always re-check server-side)
$maxBytes = 2 * 1024 * 1024;  // 2 MB
if ($file["size"] > $maxBytes) {
    die("File too large. Maximum size is 2 MB.");
}

// Step 4 — validate MIME type using finfo (DON'T trust $_FILES["type"] — it's client-supplied!)
$finfo    = finfo_open(FILEINFO_MIME_TYPE);
$mimeType = finfo_file($finfo, $file["tmp_name"]);
finfo_close($finfo);

$allowedMimes = ["image/jpeg", "image/png", "image/webp", "image/gif"];
if (!in_array($mimeType, $allowedMimes, true)) {
    die("File type not allowed. Only JPEG, PNG, WebP, and GIF.");
}

// Step 5 — generate a safe filename (NEVER use the original filename!)
$extension   = strtolower(pathinfo($file["name"], PATHINFO_EXTENSION));
$safeFilename = bin2hex(random_bytes(16)) . "." . $extension;
$destination  = $uploadTo . $safeFilename;

// Step 6 — move from temp location to permanent location
if (!move_uploaded_file($file["tmp_name"], $destination)) {
    die("Failed to save the uploaded file.");
}

echo "File uploaded successfully: $safeFilename";
?>
```

---

### Multiple File Uploads

- To upload multiple files in one form, use `name="files[]"` (with `[]` — square brackets make it an array).
- `$_FILES["files"]` becomes a nested array where each property (`name`, `type`, etc.) is itself an array of values — one per file.

```html
<!-- Multiple files from a single input (the user can select multiple) -->
<form method="POST" action="upload-multiple.php" enctype="multipart/form-data">
    <input type="file" name="photos[]" multiple>
    <button>Upload All</button>
</form>

<!-- OR: Multiple separate file inputs -->
<form method="POST" action="upload-multiple.php" enctype="multipart/form-data">
    <input type="file" name="photos[]">
    <input type="file" name="photos[]">
    <input type="file" name="photos[]">
    <button>Upload All</button>
</form>
```

```php
<?php
// upload-multiple.php
$files     = $_FILES["photos"] ?? null;
$uploadTo  = __DIR__ . "/uploads/";
$uploaded  = [];
$errors    = [];

// Restructure $_FILES into a more logical array of individual files
// (PHP's default structure is awkward for multiple files)
if ($files) {
    $count           = count($files["name"]);
    $restructured    = [];

    for ($i = 0; $i < $count; $i++) {
        $restructured[] = [
            "name"     => $files["name"][$i],
            "type"     => $files["type"][$i],
            "tmp_name" => $files["tmp_name"][$i],
            "error"    => $files["error"][$i],
            "size"     => $files["size"][$i],
        ];
    }

    // Now process each file individually
    foreach ($restructured as $index => $file) {
        if ($file["error"] !== UPLOAD_ERR_OK) {
            $errors[] = "File #" . ($index + 1) . " has an error: code " . $file["error"];
            continue;
        }

        // Validate MIME type
        $finfo = finfo_open(FILEINFO_MIME_TYPE);
        $mime  = finfo_file($finfo, $file["tmp_name"]);
        finfo_close($finfo);

        if (!in_array($mime, ["image/jpeg", "image/png", "image/webp"], true)) {
            $errors[] = "File #" . ($index + 1) . " has unsupported type: $mime";
            continue;
        }

        // Safe filename
        $ext      = strtolower(pathinfo($file["name"], PATHINFO_EXTENSION));
        $safeName = bin2hex(random_bytes(8)) . ".$ext";
        $dest     = $uploadTo . $safeName;

        if (move_uploaded_file($file["tmp_name"], $dest)) {
            $uploaded[] = $safeName;
        } else {
            $errors[] = "Failed to save file #" . ($index + 1);
        }
    }
}

if (!empty($uploaded)) {
    echo "Uploaded " . count($uploaded) . " file(s):\n";
    foreach ($uploaded as $name) {
        echo "- $name\n";
    }
}

if (!empty($errors)) {
    echo "\nErrors:\n";
    foreach ($errors as $error) {
        echo "- $error\n";
    }
}
?>
```

> 💡 **Why restructure `$_FILES`?** PHP's default multi-file structure is grouped by property, not by file — `$_FILES["photos"]["name"][0]`, `$_FILES["photos"]["name"][1]`, etc. Restructuring it into an array of files (one per index) makes it far easier to process each file in a loop.

---

### `move_uploaded_file()`

- Moves a file from its **temporary upload location** to your **permanent destination**.
- This is the **only safe way** to save uploaded files — never use `rename()` or `copy()` for uploads.
- Returns `true` on success, `false` on failure.

```php
<?php
$tmpPath     = $_FILES["document"]["tmp_name"];  // Temporary path
$destination = __DIR__ . "/uploads/my-file.pdf"; // Where you want to save it

if (move_uploaded_file($tmpPath, $destination)) {
    echo "Saved to: $destination";
} else {
    echo "Failed — check permissions on the uploads directory";
}
?>
```

> 💡 **Why use `move_uploaded_file()` and not `rename()`?** `move_uploaded_file()` performs an **additional security check** — it verifies that the file was actually uploaded via PHP's HTTP POST mechanism, not some other way. Using `rename()` or `copy()` skips this check, potentially allowing an attacker to trick your code into moving files that weren't actually uploaded.

> ⚠️ **Security checklist for file uploads:**
> - ✅ Check `$_FILES["file"]["error"] === UPLOAD_ERR_OK` first.
> - ✅ Validate MIME type with `finfo_file()` — never trust `$_FILES["type"]`.
> - ✅ Validate file size server-side — client restrictions can be bypassed.
> - ✅ Generate a safe random filename — never use `$_FILES["name"]` directly.
> - ✅ Use `move_uploaded_file()` — not `rename()` or `copy()`.
> - ✅ Store files in the `uploads/` directory with PHP execution disabled (Nginx: `location /uploads { ... php_admin_value engine off; }`).
> - ✅ Never serve uploaded files directly as PHP-executable paths.

---

## Quick Revision

- **`require()`** — fatal error if file missing, loads every time it's called. Use for essential files loaded once.
- **`include()`** — warning only if file missing, loads every time. Use for optional template parts.
- **`require_once()`** — fatal error if missing, but only loads once (ignores duplicate calls). Use for classes, functions, configs.
- **`include_once()`** — warning only if missing, loads once. Use for optional files that might be included from multiple places.
- **Always use `__DIR__ . "/relative/path"`** in include/require to get a path relative to the current file's location.
- **`touch()`** creates an empty file or updates its timestamp. **`mkdir($path, 0755, true)`** creates a directory (and all parents with `true`).
- **`is_file()`** checks it's a file; **`is_dir()`** checks it's a directory; **`file_exists()`** checks either exists.
- **`scandir()`** returns all directory entries as an array — always filter out `.` and `..`.
- **`unlink()`** deletes a file. **`rmdir()`** removes a directory (must be empty first).
- **`pathinfo($path, PATHINFO_EXTENSION)`** gets the file extension. **`dirname()`** gets the directory. **`basename()`** gets the filename.
- **`copy()`** duplicates a file (original stays). **`rename()`** moves/renames a file (original is gone).
- **`fopen()` workflow:** `fopen()` → `fread()`/`fgets()`/`fwrite()` → `fclose()`. Always `fclose()` when done.
- **`fgets()`** reads one line. **`fgetc()`** reads one character. **`feof()`** detects end of file.
- For most tasks use the one-liner shortcuts: **`file_get_contents()`** (read whole file), **`file_put_contents()`** (write whole file), **`file()`** (read into array of lines).
- File uploads need **`method="POST"`** and **`enctype="multipart/form-data"`** on the form.
- **`$_FILES["field"]["error"]`** must be `UPLOAD_ERR_OK (0)` — always check it first.
- Never trust **`$_FILES["type"]`** — validate MIME type with `finfo_file()` on the `tmp_name`.
- Always generate a **random safe filename** — never use the original `$_FILES["name"]`.
- Use **`move_uploaded_file()`** only — not `rename()` or `copy()` — it performs a built-in security check.
- Store uploads outside the web root or disable PHP execution in the upload directory.