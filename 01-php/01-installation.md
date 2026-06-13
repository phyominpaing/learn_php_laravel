# PHP Installation & CLI

A **PHP environment** is the setup required to run PHP code on your machine. Before writing any PHP, you need to install PHP and understand how to use it from the command line.

---

## Table of Contents

1. [What is PHP?](#what-is-php)
2. [Installing PHP](#installing-php)
3. [PHP CLI Commands](#php-cli-commands)
4. [Running a PHP File](#running-a-php-file)
5. [Quick Revision](#quick-revision)

---

## What is PHP?

- **PHP** (PHP: Hypertext Preprocessor) is a server-side scripting language mainly used for web development.
- It can also run directly in the terminal using the **PHP CLI** (Command Line Interface).
- PHP files use the `.php` extension and code lives inside `<?php ?>` tags.

```php
<?php
echo "Hello, World!";
?>
```

---

## Installing PHP

### Windows — Chocolatey

- **Chocolatey** is a package manager for Windows (like `brew` for Mac).
- Run these commands in **PowerShell as Administrator**.

```powershell
# Install PHP
choco install php

# Upgrade PHP later
choco upgrade php
```

> ⚠️ **Warning:** Restart your terminal after installing so Windows recognizes the `php` command in PATH.

---

### macOS — Homebrew

- **Homebrew** is a package manager for macOS.

```bash
# Install latest PHP
brew install php

# Install a specific version
brew install php@8.2

# Switch between versions
brew unlink php && brew link php@8.2

# Upgrade PHP later
brew upgrade php
```

> ⚠️ **Common Mistake:** After switching versions with `brew link`, restart your terminal or run `source ~/.zshrc` for the change to take effect.

---

### Ubuntu — APT

- **APT** is the default package manager for Ubuntu/Debian.

```bash
# Update package list first
sudo apt update

# Install PHP
sudo apt install php

# Install a specific version (add PPA for newer versions)
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php8.3

# Install common extensions alongside PHP
sudo apt install php8.3 php8.3-cli php8.3-mbstring php8.3-xml php8.3-curl
```

> 💡 **Tip:** Ubuntu's default repo may have an older PHP version. Use the `ondrej/php` PPA to get the latest release.

---

### Installation Methods Comparison

| Platform | Package Manager | Install Command |
|---|---|---|
| Windows | Chocolatey | `choco install php` |
| macOS | Homebrew | `brew install php` |
| Ubuntu | APT | `sudo apt install php` |

---

## PHP CLI Commands

**PHP CLI** (Command Line Interface) lets you run and inspect PHP directly from the terminal — no browser or web server needed.

### Verify Installation

```bash
php -v
```

- Shows the currently installed PHP version.
- Always run this first to confirm PHP installed correctly.

```
PHP 8.3.2 (cli) (built: Jan 20 2024)
Copyright (c) The PHP Group
```

---

### Get Help

```bash
php -h
```

- Displays a list of all available CLI options and flags.
- Example output:

```
Usage: php [options] [-f] <file> [--] [args...]
  -v  Version number
  -i  PHP information
  -a  Run interactively
  ...
```

---

### Find Your Config File

```bash
php --ini
```

- Shows which `php.ini` configuration file PHP is currently loading.
- Example output:

```
Configuration File (php.ini) Path: /etc/php/8.3/cli
Loaded Configuration File:         /etc/php/8.3/cli/php.ini
```

> 💡 **Important:** There are usually **two** `php.ini` files — one for the CLI and one for the web server (Apache/Nginx). Editing one does **not** affect the other.

---

### Full PHP Info Dump

```bash
php -i
```

- Dumps **all** PHP configuration details to the terminal (equivalent to `phpinfo()` in a browser).
- The output is very long — pipe it with `grep` to find something specific.

```bash
# Find a specific setting
php -i | grep memory_limit
php -i | grep upload_max_filesize
```

---

### Interactive Shell (REPL)

```bash
php -a
```

- Opens an **interactive PHP shell** where you can type and run PHP code line by line.
- Great for quickly testing expressions without creating a file.

```
Interactive shell

php > echo "Hello!";
Hello!
php > $x = 10 + 5;
php > echo $x;
15
php > exit
```

> 💡 **REPL** stands for Read-Eval-Print Loop.

---

### List Loaded Modules/Extensions

```bash
php -m
```

- Lists all PHP **extensions/modules** currently loaded.
- Use `grep` to check if a specific extension is installed.

```bash
php -m | grep pdo
php -m | grep curl
```

Example output:
```
[PHP Modules]
Core
curl
mbstring
PDO
pdo_mysql
...
```

---

### Run Code Inline

```bash
php -r "echo date('Y-m-d');"
# Output: 2024-06-13

php -r "echo PHP_VERSION;"
# Output: 8.3.2
```

> ⚠️ **Warning:** Do **NOT** add `<?php` tags when using `-r`. The code is already treated as PHP.

---

### Syntax Check (Lint)

```bash
php -l filename.php
```

- Checks a file for **syntax errors** without actually running it.

```bash
# No errors
php -l hello.php
# No syntax errors detected in hello.php

# With an error
# PHP Parse error: syntax error, unexpected token in hello.php on line 3
```

> 💡 **Tip:** Always run `php -l` before executing a new file to catch typos early.

---

### Built-in Development Server

```bash
# Start server in current directory
php -S localhost:8000

# Serve a specific folder
php -S localhost:8000 -t public/
```

> ⚠️ **Warning:** The built-in server is for **development only**. Never use it in production.

---

### PHP CLI Commands Summary

| Command | Purpose |
|---|---|
| `php -v` | Show PHP version |
| `php -h` | Show help and available options |
| `php --ini` | Show which `php.ini` config is loaded |
| `php -i` | Full PHP info dump (pipe with `grep`) |
| `php -a` | Open interactive PHP shell (REPL) |
| `php -m` | List all loaded modules/extensions |
| `php -r "code"` | Run PHP code inline without a file |
| `php -l filename.php` | Syntax check a file without running it |
| `php -S localhost:8000` | Start built-in development web server |
| `php filename.php` | Run a PHP script file |

---

## Running a PHP File

- Create a file with the `.php` extension, write your code inside `<?php ?>`, then run it with `php filename.php`.

```php
<?php
// hello.php
echo "Hello, World!\n";
```

```bash
php hello.php
# Output: Hello, World!
```

> 💡 **Tip:** `\n` adds a newline in terminal output. In a browser, you'd use `<br>` instead.

---

## Quick Revision

- PHP needs to be **installed** before you can run any PHP code — use Chocolatey (Windows), Homebrew (macOS), or APT (Ubuntu).
- The **PHP CLI** lets you run PHP directly in the terminal without a browser or web server.
- `php -v` confirms your installation; `php --ini` tells you which config file is active.
- There are **two separate `php.ini` files** — one for CLI, one for web server. They don't affect each other.
- `php -a` opens an interactive shell for quick testing without creating a file.
- `php -l` checks for syntax errors **before** running a file — use it as a habit.
- `php -i | grep <setting>` is the fastest way to find a specific configuration value.
- `php -S localhost:8000` starts a quick dev server — **development only**, never production.
- Run any PHP script with `php filename.php`.