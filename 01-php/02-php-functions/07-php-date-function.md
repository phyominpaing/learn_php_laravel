# PHP Date & Time Functions

Working with dates and times is something every PHP application needs — displaying timestamps, calculating deadlines, formatting dates for databases, scheduling tasks, and handling different timezones. PHP has a rich set of built-in tools for all of this.

---

## Table of Contents

1. [How PHP Handles Time](#how-php-handles-time)
2. [Timezone Setup](#timezone-setup)
   - [`date_default_timezone_set()`](#date_default_timezone_set)
   - [`date_default_timezone_get()`](#date_default_timezone_get)
3. [`date()` — Formatting Dates](#date--formatting-dates)
   - [Date Format Characters](#date-format-characters)
4. [`time()` — Unix Timestamp](#time--unix-timestamp)
5. [`mktime()` — Create a Timestamp](#mktime--create-a-timestamp)
6. [`strtotime()` — Parse a Date String](#strtotime--parse-a-date-string)
7. [`checkdate()` — Validate a Date](#checkdate--validate-a-date)
8. [`getdate()` — Date as Array](#getdate--date-as-array)
9. [The `DateTime` Class](#the-datetime-class)
10. [The `DateTimeImmutable` Class](#the-datetimeimmutable-class)
11. [`DateInterval` — Working with Durations](#dateinterval--working-with-durations)
12. [`DateTimeZone` — Timezone Object](#datetimezone--timezone-object)
13. [Comparing Dates](#comparing-dates)
14. [Practical Real-World Examples](#practical-real-world-examples)
15. [Quick Revision](#quick-revision)

---

## How PHP Handles Time

- PHP measures time internally as a **Unix timestamp** — the number of seconds that have elapsed since **January 1, 1970, 00:00:00 UTC** (called the "Unix Epoch").
- This is just a big integer — easy to store, compare, and do math with.
- `time()` gives you the current Unix timestamp.

```php
<?php
echo time();   // Output: e.g. 1719561600  (seconds since Jan 1, 1970)

// To human-readable:
echo date("Y-m-d H:i:s", time());  // 2026-06-28 14:30:00

// Unix timestamp is just a number — easy math
$oneHourAgo   = time() - 3600;          // 3600 seconds = 1 hour
$oneDayLater  = time() + 86400;         // 86400 seconds = 1 day
$oneWeekLater = time() + (7 * 86400);   // 7 days
?>
```

> 💡 **Why Unix time?** Storing a date as a single integer (`1719561600`) is much simpler than a formatted string (`"2026-06-28 14:30:00"`) — it's easy to compare, sort, add, and subtract. Formatting is only needed when displaying to humans.

---

## Timezone Setup

**Before doing anything with dates**, always set your timezone. Without it, PHP uses the server's default timezone (often UTC), which may not match your users or your database.

---

### `date_default_timezone_set()`

- Sets the **default timezone** for all date/time functions in the current script.
- Call this **once at the top of your application** (or set it in `php.ini`).
- Must be called before any date functions are used.

```php
<?php
// Set timezone for Myanmar
date_default_timezone_set("Asia/Yangon");
echo date("Y-m-d H:i:s");  // Shows current time in Myanmar time (+06:30)

// Common timezones
date_default_timezone_set("UTC");               // Coordinated Universal Time
date_default_timezone_set("Asia/Yangon");       // Myanmar (+06:30)
date_default_timezone_set("Asia/Bangkok");      // Thailand (+07:00)
date_default_timezone_set("Asia/Singapore");    // Singapore (+08:00)
date_default_timezone_set("Asia/Tokyo");        // Japan (+09:00)
date_default_timezone_set("Asia/Kolkata");      // India (+05:30)
date_default_timezone_set("Europe/London");     // UK
date_default_timezone_set("America/New_York");  // US Eastern
date_default_timezone_set("America/Los_Angeles"); // US Pacific
?>
```

```ini
; Best practice: set in php.ini instead of in every script
[Date]
date.timezone = "Asia/Yangon"
```

> ⚠️ **Warning:** If you don't set a timezone, PHP raises a `Warning: date(): It is not safe to rely on the system's timezone settings` in some configurations. Always set it explicitly.

---

### `date_default_timezone_get()`

- Returns the **currently active timezone** as a string.

```php
<?php
date_default_timezone_set("Asia/Yangon");
echo date_default_timezone_get();  // Output: Asia/Yangon

// Get all available timezone identifiers
$timezones = DateTimeZone::listIdentifiers();
print_r(array_slice($timezones, 0, 5));
// Array ( [0] => Africa/Abidjan [1] => Africa/Accra ... )

// Filter timezones by region
$asiaTimezones = DateTimeZone::listIdentifiers(DateTimeZone::ASIA);
foreach ($asiaTimezones as $tz) {
    echo $tz . "\n";  // Asia/Aden, Asia/Almaty, Asia/Amman, Asia/Yangon...
}
?>
```

---

## `date()` — Formatting Dates

- The most commonly used date function — formats a Unix timestamp into a human-readable string.
- Syntax: `date(string $format, int $timestamp = time())`
- If `$timestamp` is omitted, it defaults to **now**.

```php
<?php
// Current date and time (defaults to now)
echo date("Y-m-d");           // 2026-06-28
echo date("d/m/Y");           // 28/06/2026
echo date("H:i:s");           // 14:30:00
echo date("Y-m-d H:i:s");     // 2026-06-28 14:30:00

// With a specific timestamp
$timestamp = mktime(14, 30, 0, 6, 28, 2026);  // June 28, 2026 14:30:00
echo date("Y-m-d H:i:s", $timestamp);          // 2026-06-28 14:30:00

// Human-friendly formats
echo date("l, F j, Y");       // Sunday, June 28, 2026
echo date("D, d M Y");        // Sun, 28 Jun 2026
echo date("g:i A");           // 2:30 PM  (12-hour with AM/PM)
echo date("G:i");             // 14:30    (24-hour without leading zero)

// Day of week, week number
echo date("N");   // 7  (ISO day: 1=Monday, 7=Sunday)
echo date("w");   // 0  (0=Sunday, 6=Saturday)
echo date("W");   // 26 (ISO week number of the year)

// Timestamps and offsets
echo date("U");   // Unix timestamp (same as time())
echo date("Z");   // Timezone offset in seconds (e.g. 23400 for +06:30)
echo date("P");   // Timezone offset as +HH:MM  e.g. +06:30
echo date("T");   // Timezone abbreviation e.g. MMT
?>
```

---

### Date Format Characters

This is the complete reference for `date()` format characters:

#### Year

| Format | Description | Example |
|---|---|---|
| `Y` | 4-digit year | `2026` |
| `y` | 2-digit year | `26` |
| `L` | Leap year? `1` if yes, `0` if no | `0` |
| `o` | ISO 8601 week-numbering year | `2026` |

#### Month

| Format | Description | Example |
|---|---|---|
| `m` | Month with leading zero | `06` |
| `n` | Month without leading zero | `6` |
| `M` | Month abbreviation (3 letters) | `Jun` |
| `F` | Full month name | `June` |
| `t` | Number of days in the month | `30` |

#### Day

| Format | Description | Example |
|---|---|---|
| `d` | Day with leading zero | `08` |
| `j` | Day without leading zero | `8` |
| `D` | Day abbreviation (3 letters) | `Sun` |
| `l` (lowercase L) | Full day name | `Sunday` |
| `N` | ISO day number (1=Mon, 7=Sun) | `7` |
| `w` | Day number (0=Sun, 6=Sat) | `0` |
| `z` | Day of the year (0–365) | `178` |
| `S` | Ordinal suffix (st, nd, rd, th) | `th` |

#### Hour

| Format | Description | Example |
|---|---|---|
| `H` | Hour (24h) with leading zero | `14` |
| `G` | Hour (24h) without leading zero | `14` |
| `h` | Hour (12h) with leading zero | `02` |
| `g` | Hour (12h) without leading zero | `2` |
| `A` | AM or PM (uppercase) | `PM` |
| `a` | am or pm (lowercase) | `pm` |

#### Minute, Second, Microsecond

| Format | Description | Example |
|---|---|---|
| `i` | Minutes with leading zero | `05` |
| `s` | Seconds with leading zero | `09` |
| `u` | Microseconds | `123456` |
| `v` | Milliseconds | `123` |

#### Timezone

| Format | Description | Example |
|---|---|---|
| `e` | Timezone identifier | `Asia/Yangon` |
| `T` | Timezone abbreviation | `MMT` |
| `Z` | Timezone offset in seconds | `23400` |
| `O` | UTC offset with no colon | `+0630` |
| `P` | UTC offset with colon | `+06:30` |

#### Special

| Format | Description | Example |
|---|---|---|
| `U` | Unix timestamp | `1719561600` |
| `c` | ISO 8601 full date+time | `2026-06-28T14:30:00+06:30` |
| `r` | RFC 2822 formatted date | `Sun, 28 Jun 2026 14:30:00 +0630` |

```php
<?php
date_default_timezone_set("Asia/Yangon");

// Common practical format combinations
echo date("Y-m-d");                // 2026-06-28       ← MySQL DATE column
echo date("Y-m-d H:i:s");         // 2026-06-28 14:30:00 ← MySQL DATETIME column
echo date("c");                    // 2026-06-28T14:30:00+06:30 ← ISO 8601
echo date("r");                    // Sun, 28 Jun 2026 14:30:00 +0630 ← email headers
echo date("U");                    // 1719561600      ← Unix timestamp
echo date("D, d M Y");             // Sun, 28 Jun 2026
echo date("l, F jS, Y");           // Sunday, June 28th, 2026
echo date("g:i A");                // 2:30 PM
echo date("d/m/Y H:i");            // 28/06/2026 14:30

// Escaping literal characters — backslash before letter
echo date("Y \Y\e\a\r m \M\o\n\t\h");   // 2026 Year 06 Month
// Or use a non-format character directly (non-letter chars are literal)
echo date("Y-m-d");  // Hyphens are not format chars — output literally
?>
```

---

## `time()` — Unix Timestamp

- Returns the **current Unix timestamp** as an integer.
- The foundation of all PHP date/time calculations.

```php
<?php
$now = time();
echo $now;   // e.g. 1719561600

// Date arithmetic using seconds
$oneMinute  = 60;
$oneHour    = 60 * 60;           // 3600
$oneDay     = 60 * 60 * 24;      // 86400
$oneWeek    = 60 * 60 * 24 * 7;  // 604800
$oneMonth   = 60 * 60 * 24 * 30; // Approximate! Use DateInterval for accuracy
$oneYear    = 60 * 60 * 24 * 365;

$yesterday  = $now - $oneDay;
$tomorrow   = $now + $oneDay;
$nextWeek   = $now + $oneWeek;
$lastYear   = $now - $oneYear;

echo date("Y-m-d", $yesterday); // Yesterday's date
echo date("Y-m-d", $tomorrow);  // Tomorrow's date
echo date("Y-m-d", $nextWeek);  // A week from now

// How long ago was something?
$eventTime   = 1719475200;  // Some past timestamp
$secondsAgo  = $now - $eventTime;
$hoursAgo    = floor($secondsAgo / $oneHour);
echo "Event was $hoursAgo hours ago";
?>
```

---

## `mktime()` — Create a Timestamp

- Creates a Unix timestamp for a **specific date and time** you specify.
- Syntax: `mktime(hour, minute, second, month, day, year)`
- Parameters you don't specify default to the current time.

```php
<?php
// Create timestamp for a specific date
$birthday    = mktime(0, 0, 0, 8, 15, 1995);  // Aug 15, 1995 00:00:00
echo date("Y-m-d", $birthday);  // 1995-08-15

// Christmas 2026
$christmas   = mktime(0, 0, 0, 12, 25, 2026);
echo date("l, F j, Y", $christmas);  // Friday, December 25, 2026

// How many days until Christmas?
$daysLeft = ceil(($christmas - time()) / 86400);
echo "$daysLeft days until Christmas";

// Start and end of today
$startOfDay  = mktime(0,  0,  0,  date("n"), date("j"), date("Y"));
$endOfDay    = mktime(23, 59, 59, date("n"), date("j"), date("Y"));
echo date("Y-m-d H:i:s", $startOfDay);  // 2026-06-28 00:00:00
echo date("Y-m-d H:i:s", $endOfDay);    // 2026-06-28 23:59:59

// First day of current month
$firstOfMonth = mktime(0, 0, 0, date("n"), 1, date("Y"));
echo date("Y-m-d", $firstOfMonth);  // 2026-06-01

// Last day of current month (day 0 of NEXT month)
$lastOfMonth  = mktime(0, 0, 0, date("n") + 1, 0, date("Y"));
echo date("Y-m-d", $lastOfMonth);   // 2026-06-30
?>
```

---

## `strtotime()` — Parse a Date String

- Converts a **human-readable date string** into a Unix timestamp.
- Incredibly flexible — understands natural language expressions.
- Syntax: `strtotime(string $datetime, int $baseTimestamp = time())`

```php
<?php
// Parse a date string
$ts = strtotime("2026-06-28");
echo date("Y-m-d", $ts);       // 2026-06-28

$ts = strtotime("June 28, 2026");
echo date("Y-m-d", $ts);       // 2026-06-28

$ts = strtotime("28-06-2026");
echo date("Y-m-d", $ts);       // 2026-06-28

// NATURAL LANGUAGE — this is where strtotime() shines
echo date("Y-m-d", strtotime("now"));             // Today
echo date("Y-m-d", strtotime("today"));           // Today at 00:00:00
echo date("Y-m-d", strtotime("yesterday"));       // Yesterday
echo date("Y-m-d", strtotime("tomorrow"));        // Tomorrow

echo date("Y-m-d", strtotime("+1 day"));          // Tomorrow
echo date("Y-m-d", strtotime("+7 days"));         // 7 days from now
echo date("Y-m-d", strtotime("+1 week"));         // 1 week from now
echo date("Y-m-d", strtotime("+1 month"));        // 1 month from now
echo date("Y-m-d", strtotime("+1 year"));         // 1 year from now
echo date("Y-m-d", strtotime("-3 days"));         // 3 days ago
echo date("Y-m-d", strtotime("-2 months"));       // 2 months ago

echo date("Y-m-d", strtotime("next monday"));     // Coming Monday
echo date("Y-m-d", strtotime("last friday"));     // Most recent Friday
echo date("Y-m-d", strtotime("first day of next month")); // Aug 1 (if July now)
echo date("Y-m-d", strtotime("last day of this month"));  // June 30

// Relative to a base timestamp (second argument)
$base = strtotime("2026-01-01");
echo date("Y-m-d", strtotime("+3 months", $base));  // 2026-04-01
echo date("Y-m-d", strtotime("next monday", $base)); // First Monday of 2026

// Returns false for invalid strings — always check!
$ts = strtotime("not a date");
if ($ts === false) {
    echo "Invalid date string";
}
?>
```

> ⚠️ **Warning:** `strtotime()` can misinterpret ambiguous formats. For example, `"01/02/03"` could be Jan 2, 2003 or Feb 1, 2003 depending on context. Always use unambiguous formats like `"Y-m-d"` for data, and check the return value — it returns `false` on failure.

---

## `checkdate()` — Validate a Date

- Checks if a given date is **actually valid** — correct month, day in range for the month, etc.
- Syntax: `checkdate(int $month, int $day, int $year)`
- Returns `true` if valid, `false` if not.

```php
<?php
var_dump(checkdate(6, 28, 2026));   // bool(true)  — June 28, 2026
var_dump(checkdate(2, 29, 2024));   // bool(true)  — Feb 29, 2024 (2024 is leap year)
var_dump(checkdate(2, 29, 2023));   // bool(false) — Feb 29, 2023 (2023 is NOT leap year)
var_dump(checkdate(13, 1, 2026));   // bool(false) — month 13 doesn't exist
var_dump(checkdate(6, 31, 2026));   // bool(false) — June has only 30 days
var_dump(checkdate(0, 1, 2026));    // bool(false) — month 0 doesn't exist

// Practical: validate a user-submitted date
function isValidDate(int $year, int $month, int $day): bool {
    return checkdate($month, $day, $year);
}

$year  = (int) ($_POST["year"]  ?? 0);
$month = (int) ($_POST["month"] ?? 0);
$day   = (int) ($_POST["day"]   ?? 0);

if (!isValidDate($year, $month, $day)) {
    echo "Invalid date submitted";
} else {
    echo "Date is valid: " . date("Y-m-d", mktime(0, 0, 0, $month, $day, $year));
}
?>
```

---

## `getdate()` — Date as Array

- Returns an **associative array** containing all components of a timestamp.
- Useful when you need multiple components from the same timestamp without calling `date()` many times.

```php
<?php
$info = getdate();  // Current time
print_r($info);
// Array (
//   [seconds] => 0
//   [minutes] => 30
//   [hours]   => 14
//   [mday]    => 28        ← day of the month
//   [wday]    => 0         ← day of the week (0=Sunday)
//   [mon]     => 6         ← month
//   [year]    => 2026
//   [yday]    => 178       ← day of the year (0-365)
//   [weekday] => Sunday
//   [month]   => June
//   [0]       => 1719561600  ← Unix timestamp
// )

// With a specific timestamp
$info = getdate(mktime(0, 0, 0, 12, 25, 2026));
echo $info["weekday"];  // Friday
echo $info["month"];    // December
echo $info["mday"];     // 25
echo $info["year"];     // 2026
?>
```

---

## The `DateTime` Class

- The **object-oriented way** to work with dates — more powerful and readable than procedural functions.
- Mutable — operations on it change the object itself.

```php
<?php
// Create a DateTime object
$now  = new DateTime();                          // Current time
$date = new DateTime("2026-06-28");              // Specific date
$date = new DateTime("2026-06-28 14:30:00");     // Specific date + time
$date = new DateTime("now", new DateTimeZone("Asia/Yangon"));

// Format the date
echo $now->format("Y-m-d H:i:s");      // 2026-06-28 14:30:00
echo $now->format("l, F j, Y");         // Sunday, June 28, 2026
echo $now->format("U");                 // Unix timestamp
echo $now->format("c");                 // ISO 8601

// Modify the date (MUTATES the object)
$date = new DateTime("2026-01-01");
$date->modify("+3 months");
echo $date->format("Y-m-d");  // 2026-04-01

$date->modify("+1 year");
echo $date->format("Y-m-d");  // 2027-04-01

$date->modify("last day of this month");
echo $date->format("Y-m-d");  // 2027-04-30

// Set specific components
$date = new DateTime();
$date->setDate(2026, 12, 25);       // Change only the date
$date->setTime(18, 30, 0);          // Change only the time
echo $date->format("Y-m-d H:i:s"); // 2026-12-25 18:30:00

// Add/subtract with DateInterval
$date = new DateTime("2026-06-28");
$date->add(new DateInterval("P1Y2M3D"));   // Add 1 year, 2 months, 3 days
echo $date->format("Y-m-d");  // 2027-08-31

$date->sub(new DateInterval("P10D"));      // Subtract 10 days
echo $date->format("Y-m-d");  // 2027-08-21

// Get the timestamp
echo $date->getTimestamp();   // Unix timestamp as integer

// Change timezone
$date = new DateTime("2026-06-28 14:30:00", new DateTimeZone("UTC"));
$date->setTimezone(new DateTimeZone("Asia/Yangon"));
echo $date->format("Y-m-d H:i:s P");  // 2026-06-28 21:00:00 +06:30

// Create from a format string (useful for non-standard input)
$date = DateTime::createFromFormat("d/m/Y", "28/06/2026");
echo $date->format("Y-m-d");   // 2026-06-28

$date = DateTime::createFromFormat("d-M-Y H:i", "28-Jun-2026 14:30");
echo $date->format("Y-m-d H:i:s");   // 2026-06-28 14:30:00
?>
```

---

## The `DateTimeImmutable` Class

- Identical API to `DateTime`, but **never modifies itself** — every operation returns a **new object**.
- The modern, preferred choice — avoids bugs caused by unexpected mutation.

```php
<?php
// DateTime MUTATES itself (dangerous)
$date    = new DateTime("2026-01-01");
$nextMonth = $date->modify("+1 month");  // Modifies $date AND returns it
echo $date->format("Y-m-d");      // 2026-02-01 ← $date was CHANGED!
echo $nextMonth->format("Y-m-d"); // 2026-02-01 ← same object

// DateTimeImmutable NEVER mutates (safe)
$date    = new DateTimeImmutable("2026-01-01");
$nextMonth = $date->modify("+1 month");  // Returns a NEW object, $date untouched
echo $date->format("Y-m-d");      // 2026-01-01 ← UNCHANGED ✅
echo $nextMonth->format("Y-m-d"); // 2026-02-01 ← new object

// All operations return new instances
$original = new DateTimeImmutable("2026-06-28");
$plus1Day  = $original->modify("+1 day");
$plus1Week = $original->modify("+1 week");
$lastYear  = $original->modify("-1 year");

echo $original->format("Y-m-d");  // 2026-06-28 (always unchanged)
echo $plus1Day->format("Y-m-d");  // 2026-06-29
echo $plus1Week->format("Y-m-d"); // 2026-07-05
echo $lastYear->format("Y-m-d");  // 2025-06-28

// Fluent chaining — safe because each step returns a new object
$result = (new DateTimeImmutable("2026-01-01"))
    ->modify("+3 months")
    ->modify("+15 days")
    ->setTime(12, 0, 0);
echo $result->format("Y-m-d H:i:s");  // 2026-04-16 12:00:00

// Creating from format
$date = DateTimeImmutable::createFromFormat("d/m/Y", "28/06/2026");
echo $date->format("Y-m-d");   // 2026-06-28
?>
```

> 💡 **Best Practice:** Always use `DateTimeImmutable` over `DateTime` in new code. The immutable version is safer — you can pass it to functions without worrying about them modifying your date. `DateTime` is mutable and can cause subtle bugs when the same object is shared.

---

## `DateInterval` — Working with Durations

- Represents a **duration of time** (e.g., 1 year 3 months 5 days).
- Used with `DateTime::add()`, `DateTime::sub()`, and `DateTime::diff()`.
- Syntax: `new DateInterval("PnYnMnDTnHnMnS")`

### DateInterval Format

```
P = Period (required prefix)
Y = Years
M = Months
D = Days
T = Time separator (required before time components)
H = Hours
M = Minutes (after T)
S = Seconds

Examples:
P1Y        = 1 year
P3M        = 3 months
P1Y6M      = 1 year 6 months
P1Y2M3D    = 1 year 2 months 3 days
PT1H       = 1 hour
PT30M      = 30 minutes
P1DT12H    = 1 day 12 hours
P1Y2M3DT4H5M6S = full specification
```

```php
<?php
$date = new DateTimeImmutable("2026-01-01");

// Add 1 year 6 months
$future = $date->add(new DateInterval("P1Y6M"));
echo $future->format("Y-m-d");  // 2027-07-01

// Add 2 hours 30 minutes
$future = $date->add(new DateInterval("PT2H30M"));
echo $future->format("Y-m-d H:i:s");  // 2026-01-01 02:30:00

// Subtract 10 days
$past = $date->sub(new DateInterval("P10D"));
echo $past->format("Y-m-d");  // 2025-12-22

// Calculate difference between two dates
$start = new DateTimeImmutable("2026-01-01");
$end   = new DateTimeImmutable("2026-06-28");
$diff  = $start->diff($end);

echo $diff->days;   // 178  — total days
echo $diff->m;      // 5    — months component
echo $diff->d;      // 27   — remaining days
echo $diff->y;      // 0    — years
echo $diff->invert; // 0    — 0 = positive (end is after start); 1 = negative

// Human-readable difference
echo $diff->format("%y years, %m months, %d days");
// 0 years, 5 months, 27 days

// Check if date is in the past or future
$diff  = (new DateTimeImmutable())->diff(new DateTimeImmutable("2027-01-01"));
if ($diff->invert === 0) {
    echo "Date is in the future — {$diff->days} days away";
} else {
    echo "Date is in the past — {$diff->days} days ago";
}
?>
```

---

## `DateTimeZone` — Timezone Object

- Represents a **timezone** as an object — used with `DateTime`/`DateTimeImmutable`.

```php
<?php
// Create timezone objects
$yangon    = new DateTimeZone("Asia/Yangon");
$utc       = new DateTimeZone("UTC");
$london    = new DateTimeZone("Europe/London");
$newYork   = new DateTimeZone("America/New_York");

// Create date in a specific timezone
$date = new DateTimeImmutable("2026-06-28 14:30:00", $yangon);
echo $date->format("Y-m-d H:i:s P");  // 2026-06-28 14:30:00 +06:30

// Convert to another timezone
$inUtc    = $date->setTimezone($utc);
$inLondon = $date->setTimezone($london);

echo $date->format("H:i T");    // 14:30 MMT    (Myanmar)
echo $inUtc->format("H:i T");   // 08:00 UTC    (same moment, different tz)
echo $inLondon->format("H:i T");// 09:00 BST    (same moment, different tz)

// Timezone info
echo $yangon->getName();  // Asia/Yangon
echo $yangon->getOffset(new DateTime());  // 23400 (seconds = +06:30)

// List timezones by region
$asianTz = DateTimeZone::listIdentifiers(DateTimeZone::ASIA);
foreach (array_slice($asianTz, 0, 5) as $tz) {
    echo $tz . "\n";
}
?>
```

---

## Comparing Dates

```php
<?php
// Method 1: Compare Unix timestamps (procedural)
$date1 = strtotime("2026-06-28");
$date2 = strtotime("2027-01-01");

if ($date1 < $date2) echo "Date1 is earlier";
if ($date1 > $date2) echo "Date1 is later";
if ($date1 === $date2) echo "Same date";

// Method 2: Compare DateTime objects (OOP) — PHP supports < > == on DateTime!
$a = new DateTimeImmutable("2026-06-28");
$b = new DateTimeImmutable("2027-01-01");

var_dump($a < $b);    // bool(true)
var_dump($a > $b);    // bool(false)
var_dump($a == $b);   // bool(false)

$c = new DateTimeImmutable("2026-06-28");
var_dump($a == $c);   // bool(true)  — same date
var_dump($a === $c);  // bool(false) — different objects (use == not ===)

// Method 3: diff() for the exact duration
$diff = $a->diff($b);
echo $diff->days;     // 187 days between the two dates
echo $diff->format("%m months %d days");

// Check if date is within a range
function isWithinRange(DateTimeImmutable $date, DateTimeImmutable $from, DateTimeImmutable $to): bool {
    return $date >= $from && $date <= $to;
}

$event = new DateTimeImmutable("2026-08-15");
$from  = new DateTimeImmutable("2026-08-01");
$to    = new DateTimeImmutable("2026-08-31");

var_dump(isWithinRange($event, $from, $to));  // bool(true)
?>
```

---

## Practical Real-World Examples

### Age Calculator

```php
<?php
function calculateAge(string $birthdate): int {
    $birth = new DateTimeImmutable($birthdate);
    $today = new DateTimeImmutable("today");
    return $birth->diff($today)->y;  // ->y = years component of the difference
}

echo calculateAge("1995-08-15");  // e.g. 30 (depends on current date)
echo calculateAge("2000-01-01");  // e.g. 26
?>
```

---

### Time Elapsed (Like "2 hours ago")

```php
<?php
function timeAgo(int|string|DateTimeInterface $time): string {
    $date = match(true) {
        is_int($time)                   => new DateTimeImmutable("@$time"),
        is_string($time)                => new DateTimeImmutable($time),
        $time instanceof DateTimeInterface => DateTimeImmutable::createFromInterface($time),
    };

    $diff    = $date->diff(new DateTimeImmutable());
    $seconds = abs((new DateTimeImmutable())->getTimestamp() - $date->getTimestamp());

    return match(true) {
        $seconds < 60         => "just now",
        $seconds < 3600       => floor($seconds / 60) . " minutes ago",
        $seconds < 86400      => floor($seconds / 3600) . " hours ago",
        $seconds < 604800     => floor($seconds / 86400) . " days ago",
        $diff->m < 1          => floor($seconds / 604800) . " weeks ago",
        $diff->y < 1          => $diff->m . " months ago",
        default               => $diff->y . " years ago",
    };
}

echo timeAgo(time() - 45);          // just now
echo timeAgo(time() - 3600);        // 1 hours ago
echo timeAgo(time() - 86400 * 5);   // 5 days ago
echo timeAgo("2025-01-01");         // X months ago
?>
```

---

### Next Business Day

```php
<?php
function nextBusinessDay(DateTimeImmutable $from): DateTimeImmutable {
    $holidays = [
        "2026-01-01",  // New Year
        "2026-12-25",  // Christmas
        // Add your public holidays here
    ];

    $day = $from->modify("+1 day");

    while (true) {
        $dayOfWeek = (int) $day->format("N");  // 1=Mon, 7=Sun
        $dateStr   = $day->format("Y-m-d");

        // Skip weekends and holidays
        if ($dayOfWeek <= 5 && !in_array($dateStr, $holidays, true)) {
            return $day;
        }

        $day = $day->modify("+1 day");
    }
}

$today    = new DateTimeImmutable("2026-06-26");  // Friday
$nextBiz  = nextBusinessDay($today);
echo $nextBiz->format("l, Y-m-d");   // Monday, 2026-06-29
?>
```

---

### Format Duration for Display

```php
<?php
function formatDuration(int $seconds): string {
    $interval = new DateInterval("PT{$seconds}S");
    // Normalize: convert to a proper interval
    $from   = new DateTimeImmutable("@0");
    $to     = new DateTimeImmutable("@$seconds");
    $diff   = $from->diff($to);

    $parts = [];
    if ($diff->y)  $parts[] = "{$diff->y} year" . ($diff->y > 1 ? "s" : "");
    if ($diff->m)  $parts[] = "{$diff->m} month" . ($diff->m > 1 ? "s" : "");
    if ($diff->d)  $parts[] = "{$diff->d} day" . ($diff->d > 1 ? "s" : "");
    if ($diff->h)  $parts[] = "{$diff->h} hour" . ($diff->h > 1 ? "s" : "");
    if ($diff->i)  $parts[] = "{$diff->i} minute" . ($diff->i > 1 ? "s" : "");
    if ($diff->s)  $parts[] = "{$diff->s} second" . ($diff->s > 1 ? "s" : "");

    return implode(", ", $parts) ?: "0 seconds";
}

echo formatDuration(3661);        // 1 hour, 1 minute, 1 second
echo formatDuration(90061);       // 1 day, 1 hour, 1 minute, 1 second
echo formatDuration(31536000);    // 1 year
?>
```

---

### Storing & Retrieving Dates with MySQL

```php
<?php
// Storing a date in MySQL
$date = new DateTimeImmutable("2026-06-28 14:30:00");

// MySQL DATE column format
$mysqlDate     = $date->format("Y-m-d");           // "2026-06-28"

// MySQL DATETIME column format
$mysqlDatetime = $date->format("Y-m-d H:i:s");     // "2026-06-28 14:30:00"

// MySQL TIMESTAMP — same format as DATETIME
$mysqlTimestamp = $date->format("Y-m-d H:i:s");

$stmt = $pdo->prepare("INSERT INTO events (title, event_date) VALUES (?, ?)");
$stmt->execute(["My Event", $mysqlDatetime]);

// Reading from MySQL and converting back to DateTimeImmutable
$row  = $pdo->query("SELECT event_date FROM events WHERE id = 1")->fetch();
$date = new DateTimeImmutable($row["event_date"]);
echo $date->format("l, F j, Y g:i A");  // Sunday, June 28, 2026 2:30 PM
?>
```

---

## Quick Revision

- PHP measures time as **Unix timestamps** — seconds since Jan 1, 1970 UTC. All date math is just adding/subtracting seconds.
- **Always set timezone** with `date_default_timezone_set("Asia/Yangon")` before any date operation, or set `date.timezone` in `php.ini`.
- **`date($format, $timestamp)`** — formats a timestamp as a string. Omit timestamp to use now. Key formats: `Y`(year), `m`(month), `d`(day), `H`(hour 24h), `i`(minute), `s`(second), `l`(weekday name), `D`(short weekday), `N`(ISO day number 1-7).
- **`time()`** — returns current Unix timestamp as integer.
- **`mktime(h, m, s, month, day, year)`** — builds a timestamp for a specific date.
- **`strtotime("string")`** — converts human-readable strings to timestamps. Supports `"+1 month"`, `"next monday"`, `"yesterday"`, `"last day of this month"`, etc. Returns `false` on failure — always check.
- **`checkdate($month, $day, $year)`** — validates a date (catches Feb 29 on non-leap years, month 13, etc.).
- **`getdate($timestamp)`** — returns all date components as an associative array.
- **`DateTime`** — OOP date class. Mutable — operations change the object in place.
- **`DateTimeImmutable`** — preferred over `DateTime`. Every operation returns a new object, original is never changed. Use this in all new code.
- **`DateTime::createFromFormat("d/m/Y", "28/06/2026")`** — parse non-standard date formats safely.
- **`DateInterval("P1Y2M3DT4H5M6S")`** — represents a duration. Use with `->add()`, `->sub()`, `->diff()`.
- **`$date->diff($otherDate)`** — returns a `DateInterval` with `->days` (total days), `->y` (years), `->m` (months), `->d` (days), `->invert` (1 if negative/past).
- **`DateTimeZone`** — timezone object. Convert between timezones with `->setTimezone()`.
- **Compare `DateTime` objects with `<` `>` `==`** — PHP supports this natively. Use `==` not `===` to compare equal dates.
- **MySQL date storage:** `"Y-m-d"` for DATE columns, `"Y-m-d H:i:s"` for DATETIME/TIMESTAMP columns.
- **Best practice combo:** `DateTimeImmutable` + `DateTimeZone` + `DateInterval` for all date work in modern PHP.