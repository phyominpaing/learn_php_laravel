# Laravel 11 — File Storage & Uploads

Laravel's filesystem abstraction lets you read, write, and manage files using one consistent API, regardless of whether they're actually stored on your local disk, AWS S3, or another cloud provider.

## Table of Contents

- [The Filesystem Abstraction (Why It Matters)](#the-filesystem-abstraction-why-it-matters)
- [Understanding Disks](#understanding-disks)
- [The Public Disk & `storage:link`](#the-public-disk--storagelink)
- [Handling a File Upload — Step by Step](#handling-a-file-upload--step-by-step)
- [Validating Uploaded Files](#validating-uploaded-files)
- [Storing With Custom File Names](#storing-with-custom-file-names)
- [The `Storage` Facade — Full Reference](#the-storage-facade--full-reference)
- [Generating URLs to Stored Files](#generating-urls-to-stored-files)
- [Deleting & Replacing Files](#deleting--replacing-files)
- [Storing Files on Amazon S3](#storing-files-on-amazon-s3)
- [Image Handling Considerations](#image-handling-considerations)
- [Associating Uploads With a Model](#associating-uploads-with-a-model)
- [Quick Revision](#quick-revision)

---

## The Filesystem Abstraction (Why It Matters)

- **Filesystem abstraction**: a unified API that lets your application code stay identical, no matter where files physically live — local disk during development, AWS S3 in production — because Laravel handles the underlying differences internally (powered by a package called **Flysystem**).

```php
// This exact same line works whether "public" is local storage or S3
Storage::disk('public')->put('avatars/photo.jpg', $contents);
```

> 💡 **Tip:** This is a genuinely senior-relevant concept beyond just file storage: designing your own code behind a consistent interface (so the underlying implementation can be swapped later without touching business logic) is a core software design principle, and Laravel's filesystem is a great real example of it in action.

---

## Understanding Disks

- **Disk**: a named, pre-configured storage location — defined in `config/filesystems.php` — specifying both a **driver** (local, S3, FTP, etc.) and connection details for that driver.

```php
// config/filesystems.php
'disks' => [
    'local' => [
        'driver' => 'local',
        'root' => storage_path('app/private'),
    ],

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],

    's3' => [
        'driver' => 's3',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION'),
        'bucket' => env('AWS_BUCKET'),
    ],
],
```

| Disk | Driver | Files Are... | Typical Use |
|---|---|---|---|
| `local` | local | Private, never web-accessible directly | Sensitive documents (invoices, private uploads) |
| `public` | local | Web-accessible (once linked, below) | Avatars, product images |
| `s3` | s3 | Stored in an AWS S3 bucket | Production file storage at scale |

> 💡 **Tip:** Switching your entire app's storage from local disk to S3 for production is often just a single `.env` change (`FILESYSTEM_DISK=s3`) — this is the direct payoff of the abstraction described above.

---

## The Public Disk & `storage:link`

By default, Laravel stores **all** files (even ones on the "public" disk) inside `storage/app/`, which is **outside** the `public/` folder — and recall from note 01, only `public/` is actually exposed to the web by your server.

This creates a problem: files on the `public` disk need to be reachable by a browser URL, but they're not physically inside `public/`.

**The fix — a symbolic link:**

```bash
php artisan storage:link
```

- This creates a **symbolic link** — essentially a shortcut/alias on the filesystem — from `public/storage` pointing to `storage/app/public`. Now, anything you store on the `public` disk is instantly reachable at a real, browsable URL, without Laravel physically duplicating the files.

```php
echo asset('storage/avatars/photo.jpg');
// → http://yourapp.test/storage/avatars/photo.jpg
```

> ⚠️ **Warning:** `storage:link` must be run **manually once per environment** (local, staging, production) — it's not something that happens automatically on deploy. Forgetting this on a fresh production server is a classic "why are my images all broken 404s" bug.

---

## Handling a File Upload — Step by Step

**The HTML form** (note the required `enctype`):

```blade
<form action="/upload" method="POST" enctype="multipart/form-data">
    @csrf
    <input type="file" name="avatar">
    <button type="submit">Upload</button>
</form>
```

- **`enctype="multipart/form-data"`**: without this exact attribute, the browser won't actually include the file's binary data in the request — one of the most common beginner mistakes with upload forms.

**The controller:**

```php
use Illuminate\Http\Request;

public function upload(Request $request)
{
    $request->validate([
        'avatar' => 'required|file|image|max:2048',
    ]);

    $path = $request->file('avatar')->store('avatars', 'public');

    return back()->with('status', "Stored at: {$path}");
}
```

- **`$request->file('avatar')`**: returns an `UploadedFile` object representing the incoming file — not just a string path.
- **`->store('avatars', 'public')`**: saves the file into an `avatars/` subfolder, on the `public` disk, and Laravel **automatically generates a random, collision-proof filename** for you.
- The returned `$path` (e.g., `"avatars/a1b2c3d4e5f6.jpg"`) is what you'd typically save into your database, so you can reconstruct the file's URL later.

---

## Validating Uploaded Files

Recall note 08's validation rules — files have their own dedicated set:

| Rule | Meaning |
|---|---|
| `file` | Must be an uploaded file |
| `image` | Must specifically be an image (jpg, png, gif, webp, etc.) |
| `mimes:jpg,png,pdf` | Restrict to specific file extensions |
| `mimetypes:image/jpeg` | Restrict by actual MIME type (more reliable than extension alone) |
| `max:2048` | Max file size in **kilobytes** (2048 = 2MB) |
| `dimensions:min_width=100,min_height=100` | Restrict image dimensions |

```php
$request->validate([
    'avatar' => 'required|image|mimes:jpg,png,webp|max:2048',
    'document' => 'required|file|mimes:pdf|max:5120',
]);
```

> ⚠️ **Warning:** Never trust a file's claimed extension or the browser-reported MIME type as your only security check — a malicious file can be renamed to look like an image. Laravel's `mimes`/`mimetypes` rules inspect the file's actual content signature, not just its name, which is exactly why you should always use these validation rules rather than skipping validation "because the input has `accept="image/*"`" (that HTML attribute is only a UI hint, easily bypassed).

---

## Storing With Custom File Names

Sometimes you want control over the filename instead of Laravel's auto-generated random one.

```php
$file = $request->file('avatar');
$filename = time().'_'.$file->getClientOriginalName();

$path = $file->storeAs('avatars', $filename, 'public');
```

- **`getClientOriginalName()`**: the filename as it was on the user's own computer — useful for display, but **never** safe to trust blindly for the actual stored filename (users can name files anything, including path-traversal attempts like `../../etc/passwd`).
- **`storeAs()`**: like `store()`, but lets you specify the exact filename instead of a random one.

> 💡 **Tip:** Prefixing with `time()` (or better, a UUID) avoids overwriting an existing file if two users happen to upload files with the same original name.

---

## The `Storage` Facade — Full Reference

Beyond handling a direct upload, the `Storage` facade lets you manipulate files on any disk directly:

```php
use Illuminate\Support\Facades\Storage;

Storage::disk('public')->put('notes/hello.txt', 'File contents here');
Storage::disk('public')->get('notes/hello.txt');
Storage::disk('public')->exists('notes/hello.txt');
Storage::disk('public')->size('notes/hello.txt');
Storage::disk('public')->delete('notes/hello.txt');
Storage::disk('public')->copy('notes/hello.txt', 'notes/backup.txt');
Storage::disk('public')->move('notes/hello.txt', 'archive/hello.txt');
Storage::disk('public')->files('notes');       // List files in a directory
Storage::disk('public')->allFiles('notes');    // List files, including subdirectories
```

| Method | Purpose |
|---|---|
| `put($path, $contents)` | Write raw content to a file |
| `get($path)` | Read a file's contents |
| `exists($path)` | Check if a file exists |
| `size($path)` | Get file size in bytes |
| `delete($path)` | Delete a file |
| `copy($from, $to)` | Duplicate a file |
| `move($from, $to)` | Move/rename a file |
| `files($dir)` | List files directly in a directory |

> 💡 **Tip:** `Storage::disk('local')` (without specifying — Laravel defaults to whatever `FILESYSTEM_DISK` is set to in `.env`) is genuinely useful for storing things like generated PDF invoices or exports that should never be publicly browsable.

---

## Generating URLs to Stored Files

```php
Storage::disk('public')->url('avatars/photo.jpg');
// → http://yourapp.test/storage/avatars/photo.jpg

// Or, using the global asset() helper directly:
asset('storage/avatars/photo.jpg');
```

**Temporary, expiring URLs** (available for local files since Laravel 9, and standard for S3):

```php
$url = Storage::disk('s3')->temporaryUrl(
    'documents/invoice.pdf',
    now()->addMinutes(5)
);
```

- **Temporary URL**: a signed link that only works for a limited time window — ideal for private files (invoices, sensitive documents) you want to let a specific user download briefly, without making the file permanently public.

---

## Deleting & Replacing Files

A very common real pattern — replacing a user's existing avatar:

```php
public function updateAvatar(Request $request)
{
    $request->validate(['avatar' => 'required|image|max:2048']);

    $user = $request->user();

    // Delete the OLD avatar first, if one exists
    if ($user->avatar_path) {
        Storage::disk('public')->delete($user->avatar_path);
    }

    $path = $request->file('avatar')->store('avatars', 'public');

    $user->update(['avatar_path' => $path]);

    return back();
}
```

> ⚠️ **Warning:** Forgetting to delete the old file when replacing it is a common real-world bug — over time, this silently accumulates orphaned files nobody references anymore, wasting storage space and (on S3) costing real money.

---

## Storing Files on Amazon S3

For production apps, S3 (or an S3-compatible service) is the standard choice — it scales independently of your app servers and survives server restarts/redeploys (local disk storage does not, on most modern hosting).

**Install the S3 driver package:**

```bash
composer require league/flysystem-aws-s3-v3
```

**Configure `.env`:**

```env
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=your-key
AWS_SECRET_ACCESS_KEY=your-secret
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=your-bucket-name
```

Once configured, **your application code doesn't change at all** — `$request->file('avatar')->store('avatars', 's3')` (or just `'avatars'` if `s3` is now your default disk) just works, uploading directly to S3 instead of local disk.

> 💡 **Tip:** This is the payoff of the filesystem abstraction in full: you can develop entirely with local storage, then flip a few `.env` values for production S3 storage, with zero code changes required in your controllers or Models.

> ⚠️ **Warning:** If you're deploying to a platform with an ephemeral/stateless filesystem (common on many modern hosting platforms, including serverless setups), local disk storage will silently **lose all uploaded files** on every redeploy or restart. Always use S3 (or similar) for user-uploaded content in any production environment where the filesystem isn't guaranteed to persist.

---

## Image Handling Considerations

Beyond just storing an image file as-is, real-world apps often need to process it first:

- **Resizing/thumbnails**: generating smaller versions for different display contexts (avatar thumbnail vs. full profile photo) — commonly done with the **Intervention Image** package, a popular third-party library for Laravel image manipulation.
- **Optimizing file size**: compressing images on upload so page load times stay fast.

```php
use Intervention\Image\Laravel\Facades\Image;

$image = Image::read($request->file('avatar'))
    ->cover(200, 200);

Storage::disk('public')->put('avatars/thumb.jpg', (string) $image->encode());
```

> 💡 **Tip:** You won't need image processing for every project, but it's worth knowing Intervention Image exists — reaching for a well-maintained package here is the right senior-level call rather than hand-writing image resizing logic with raw PHP's GD library.

---

## Associating Uploads With a Model

The standard real-world pattern: store the returned path as a plain string column on your model.

**Migration:**

```php
$table->string('avatar_path')->nullable();
```

**Model — add an accessor (from note 06) for convenience:**

```php
use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Support\Facades\Storage;

protected function avatarUrl(): Attribute
{
    return Attribute::make(
        get: fn () => $this->avatar_path
            ? Storage::disk('public')->url($this->avatar_path)
            : null,
    );
}
```

Now anywhere in your app: `$user->avatar_url` gives you a ready-to-use URL, without repeating the `Storage::disk('public')->url(...)` call everywhere you display it.

> 💡 **Tip:** For an app with many uploads per record (e.g., a product with multiple gallery images), you'd instead create a separate `images` table with a One-to-Many relationship (note 06) back to `Product` — one row per image, rather than cramming multiple paths into a single column.

---

## Quick Revision

- Laravel's filesystem abstraction (powered by Flysystem) lets identical code work across different storage backends (local disk, S3) via **disks**, configured in `config/filesystems.php`.
- The `public` disk stores files outside the web-accessible `public/` folder by default — run `php artisan storage:link` (once per environment) to create the symbolic link that makes them browsable.
- Upload forms need `enctype="multipart/form-data"`; access the file via `$request->file('field')`, then `->store('folder', 'disk')` (random filename) or `->storeAs('folder', 'name', 'disk')` (custom filename).
- Always validate uploads with `file`, `image`, `mimes:`, and `max:` rules — never trust a file's extension or browser-reported type alone.
- The `Storage` facade (`put`, `get`, `exists`, `delete`, `copy`, `move`) works identically across every configured disk.
- Generate URLs with `Storage::disk(...)->url()` or the `asset()` helper; use `temporaryUrl()` for time-limited, signed links to private files.
- Always delete the old file when replacing an upload (e.g., swapping a user's avatar) — forgetting this silently wastes storage over time.
- Production apps typically use **S3** rather than local disk, since local storage doesn't reliably persist across redeploys on most modern hosting — switching is usually just an `.env` change thanks to the filesystem abstraction.
- For image resizing/optimization, reach for the **Intervention Image** package rather than writing raw GD logic by hand.
- Store a file's returned path as a plain string column on your Model, and consider an accessor to conveniently expose its full URL.
- Next up (note 12): **API Development** — building a real JSON API with Sanctum authentication, API Resources, and proper REST conventions, the foundation for your end-of-series project.