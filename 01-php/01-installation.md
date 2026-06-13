# PHP Study Notes

## 01 — Installation & CLI

---

## Table of Contents

- [What is PHP?](#what-is-php)
- [Installation](#installation)
  - [Windows — Chocolatey](#windows--chocolatey)
  - [macOS — Homebrew](#macos--homebrew)
  - [Ubuntu/Debian — APT](#ubuntudebian--apt)
- [Verify Your Installation](#verify-your-installation)
- [PHP CLI Commands](#php-cli-commands)
- [Running a PHP File](#running-a-php-file)
- [Quick Revision](#quick-revision)

---

## What is PHP?

- **PHP** (PHP: Hypertext Preprocessor) is a server-side scripting language designed for web development.
- It can also be used as a **general-purpose language** via the command line (CLI).
- Files use the `.php` extension.
- PHP code lives inside `<?php ... ?>` tags.

---

## Installation

### Windows — Chocolatey

> **Prerequisite:** Install [Chocolatey](https://chocolatey.org/install) first (run PowerShell as Administrator).

```powershell
# Install PHP
choco install php

# Upgrade PHP later
choco upgrade php
```

> ⚠️ **Tip:** After installation, restart your terminal so the `php` command is recognized in PATH.

---

### macOS — Homebrew

> **Prerequisite:** Install [Homebrew](https://brew.sh/) first.

```bash
# Install PHP (latest stable version)
brew install php

# Install a specific version
brew install php@8.2

# Switch between versions
brew unlink php && brew link php@8.2

# Upgrade PHP later
brew upgrade php
```

> ⚠️ **Common Mistake:** After switching versions with `brew link`, you may need to restart your terminal or run `source ~/.zshrc` / `source ~/.bashrc`.

---

### Ubuntu/Debian — APT

```bash
# Update package list
sudo apt update

# Install PHP (default version from repo)
sudo apt install php

# Install a specific version (using Ondřej Surý PPA for newer versions)
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php8.3

# Install common extensions alongside PHP
sudo apt install php8.3 php8.3-cli php8.3-mbstring php8.3-xml php8.3-curl
```

> 💡 **Tip:** The default Ubuntu repo may have an older PHP version. Use the `ondrej/php` PPA for the latest releases.

---

## Verify Your Installation

```bash
php -v
```

Expected output:
```
PHP 8.3.x (cli) (built: ...)
Copyright (c) The PHP Group
```

---

## PHP CLI Commands

These are the most useful commands you'll run directly in your terminal.

| Command | Purpose |
|---|---|
| `php -v` | Show PHP version |
| `php -h` | Show help / list of CLI options |
| `php --ini` | Show which `php.ini` config file is loaded |
| `php -i` | Full PHP info dump (like `phpinfo()` in terminal) |
| `php -a` | Open interactive PHP shell (REPL) |
| `php -m` | List all loaded PHP modules/extensions |
| `php filename.php` | Run a PHP script file |
| `php -r "code"` | Run PHP code inline without a file |
| `php -l filename.php` | Syntax check a file (lint) without running it |
| `php -S host:port` | Start PHP's built-in development web server |

---

### Command Details & Examples

#### `php -v` — Version Check
```bash
php -v
# PHP 8.3.2 (cli) (built: Jan 20 2024 00:00:00)
```

---

#### `php -h` — Help
```bash
php -h
# Prints all available CLI flags and options
```

---

#### `php --ini` — Config File Location
```bash
php --ini
# Configuration File (php.ini) Path: /etc/php/8.3/cli
# Loaded Configuration File: /etc/php/8.3/cli/php.ini
```

> 💡 **Why this matters:** There are often *two* `php.ini` files — one for CLI and one for the web server (Apache/Nginx). Changes to one don't affect the other.

---

#### `php -i` — Full PHP Info
```bash
php -i
# Dumps ALL PHP configuration info (very long output)

# Tip: pipe it to grep to find something specific
php -i | grep "memory_limit"
php -i | grep "upload_max"
```

---

#### `php -a` — Interactive Shell (REPL)
```bash
php -a
# Interactive shell
# php > echo "Hello, World!";
# Hello, World!
# php > $x = 10 + 5;
# php > echo $x;
# 15
# php > exit
```

> 💡 **REPL** stands for Read-Eval-Print Loop — great for quickly testing expressions without creating a file.

---

#### `php -m` — List Loaded Modules
```bash
php -m
# [PHP Modules]
# Core
# curl
# mbstring
# PDO
# ...

# Grep for a specific extension
php -m | grep pdo
```

---

#### `php -r` — Run Code Inline
```bash
php -r "echo date('Y-m-d');"
# 2024-06-13

php -r "echo PHP_VERSION;"
# 8.3.2
```

> ⚠️ **Warning:** Do NOT add `<?php` tags when using `-r`. The code is already treated as PHP.

---

#### `php -l` — Syntax Check (Lint)
```bash
php -l myfile.php
# No syntax errors detected in myfile.php

# If there's an error:
# PHP Parse error: syntax error, unexpected token in myfile.php on line 5
```

> 💡 **Tip:** Run `php -l` before executing a file to catch typos early without running the code.

---

#### `php -S` — Built-in Dev Server
```bash
# Start a local server in the current directory
php -S localhost:8000

# Serve a specific folder
php -S localhost:8000 -t public/
```

> ⚠️ **Warning:** The built-in server is for **development only**. Never use it in production.

---

## Running a PHP File

**1. Create a PHP file:**

```php
<?php
// hello.php
echo "Hello, World!\n";
```

**2. Run it from the terminal:**

```bash
php hello.php
# Hello, World!
```

> 💡 **Tip:** The `\n` adds a newline in CLI output. In a browser, you'd use `<br>` instead.

---

## Quick Revision

| Topic | Key Takeaway |
|---|---|
| **PHP** | Server-side scripting language; files end in `.php`; code goes inside `<?php ?>` |
| **Windows install** | `choco install php` via Chocolatey (run as Administrator) |
| **macOS install** | `brew install php` via Homebrew; use `php@8.x` for specific versions |
| **Ubuntu install** | `sudo apt install php`; use `ondrej/php` PPA for latest versions |
| **`php -v`** | Check which PHP version is active |
| **`php --ini`** | Find which `php.ini` config file is being used |
| **`php -i`** | Full config dump; pipe to `grep` to find specific settings |
| **`php -a`** | Interactive REPL shell for quick testing |
| **`php -m`** | List all loaded extensions/modules |
| **`php -l`** | Lint/syntax-check a file without running it |
| **`php -r`** | Run one-liner PHP code directly in the terminal |
| **`php -S localhost:8000`** | Start built-in dev server (development only!) |
| **`php filename.php`** | Execute a PHP script from the terminal |
| **Two `php.ini` files** | CLI and web server each have their own config — changes to one don't affect the other |