# Storage

## File Uploads

### Configuration

```php
// config/upload.php
return [
    'max_size' => 10 * 1024 * 1024,  // 10 MB
    'allowed_types' => ['image/jpeg', 'image/png', 'image/gif', 'image/webp', 'application/pdf'],
    'blocked_extensions' => ['php', 'phtml', 'phar', 'exe', 'bat', 'sh'],
];
```

### Uploading

```php
use Colibri\Storage\File;

// Upload to default disk (private)
$path = File::upload($_FILES['photo']);

// Upload to a specific disk
$path = File::upload($_FILES['photo'], disk: 'public');

// Upload to a subdirectory
$path = File::upload($_FILES['photo'], 'avatars', disk: 'public');
```

Returns the relative path within the disk (e.g., `avatars/a1b2c3d4.jpg`), or `null` on failure.

### Validation

Uploads are automatically validated:

- **MIME type** checked via `finfo` (real content inspection, not just extension)
- **Extension** blocked if in `blocked_extensions`
- **Size** rejected if exceeding `max_size`
- **Filenames** replaced with unique hashes to prevent collisions and malicious names

### Security

- `storage/uploads/` is outside the web root — files are not directly accessible by URL
- `storage/.htaccess` blocks PHP execution in the storage directory
- Use `File::serve()` for controlled access (see below)

## Disks

### Configuration

```php
// config/storage.php
return [
    'default' => 'private',
    'disks' => [
        'private' => ['path' => 'storage/uploads'],
        'public' => ['path' => 'public/uploads'],
    ],
];
```

| Disk | Path | Web Access | Use Case |
|---|---|---|---|
| `private` | `storage/uploads/` | Via `File::serve()` | Documents, invoices, user files |
| `public` | `public/uploads/` | Direct URL | Images, avatars, public media |

### File Operations

```php
File::exists('photo.jpg');                    // default disk
File::exists('photo.jpg', disk: 'public');   // specific disk
File::delete('old-doc.pdf');
File::size('report.pdf');                     // bytes
File::extension('photo.jpg');                 // 'jpg'
File::mimeType('photo.jpg');                  // 'image/jpeg'
```

### URLs

```php
File::url('photo.jpg', 'public');     // /uploads/photo.jpg (direct)
File::url('doc.pdf', 'private');      // /files/doc.pdf (served via PHP)
File::url('doc.pdf');                  // /files/doc.pdf (default = private)
```

### Serving Private Files

```php
// In a route handler
File::serve('doc.pdf');  // sends headers + streams file content, then exits
```

This sends the file with correct `Content-Type`, `Content-Length`, and `Content-Disposition` headers.

## Data:: — Flat-File JSON

For structured data without a database. See [Database](04-database.md#data--flat-file-json-storage).

```php
use Colibri\Storage\Data;

Data::put('posts/my-post', ['title' => 'Hello', 'status' => 'published']);
$post = Data::get('posts/my-post');
$all = Data::all('posts', sort: 'date', order: 'desc');
$published = Data::where('posts', ['status' => 'published']);
Data::exists('posts/my-post');
Data::delete('posts/my-post');
```
