# Configuration

## Environment Variables

Colibri uses [phpdotenv](https://github.com/vlucas/phpdotenv) to load `.env` files. Copy `.env.example` to `.env` and adjust values.

```env
APP_NAME=MyApp
APP_DEBUG=true
APP_TIMEZONE=America/Montreal
APP_HTTPS=false
```

Access environment variables with the `env()` helper:

```php
$debug = env('APP_DEBUG', false);
$name = env('APP_NAME', 'Colibri');
```

The helper automatically casts string values:

| `.env` value | PHP value |
|---|---|
| `true`, `(true)` | `true` |
| `false`, `(false)` | `false` |
| `null`, `(null)` | `null` |
| `empty`, `(empty)` | `''` |

## Config Files

Configuration files live in `config/` and return PHP arrays. Values can use `env()` for environment-specific overrides.

Access with dot notation via `Config::get()`:

```php
use Colibri\Config;

Config::get('app.name');                    // reads config/app.php → 'name'
Config::get('database.driver');             // reads config/database.php → 'driver'
Config::get('app.log.channel');             // nested access
Config::get('nonexistent', 'default');      // default value
```

Config files are lazy-loaded — only read from disk on first access.

## Available Config Files

### `config/app.php`

```php
return [
    'name' => env('APP_NAME', 'Colibri'),
    'debug' => env('APP_DEBUG', false),
    'timezone' => env('APP_TIMEZONE', 'America/Montreal'),
    'https' => env('APP_HTTPS', false),
    'title_format' => ':title | :site',  // format for site_title()

    'log' => [
        'channel' => env('LOG_CHANNEL', 'daily'),  // 'single' or 'daily'
        'level' => env('LOG_LEVEL', 'debug'),       // debug, info, warning, error
        'max_files' => 30,                           // days to keep (daily only)
    ],
];
```

### `config/database.php`

```php
return [
    'driver' => env('DB_DRIVER', 'sqlite'),
    'path' => env('DB_PATH', 'storage/database.sqlite'),
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '3306'),
    'name' => env('DB_NAME', 'colibri'),
    'user' => env('DB_USER', 'root'),
    'pass' => env('DB_PASS', ''),
];
```

### `config/auth.php`

```php
return [
    'datasource' => env('AUTH_DRIVER', 'db'),  // db or file
    'roles' => ['user', 'admin'],
    'login_path' => '/login',
];
```

### `config/cache.php`

```php
return [
    'datasource' => env('CACHE_DRIVER', 'file'),  // file or db
    'path' => 'storage/cache/data',
];
```

### `config/i18n.php`

```php
return [
    'default' => env('I18N_LOCALE', 'en'),
    'fallback' => env('I18N_FALLBACK', 'en'),
    'prefixes' => [
        'en' => '/',
        'fr' => '/fr',
    ],
];
```

### `config/mail.php`

```php
return [
    'driver' => env('MAIL_DRIVER', 'log'),
    'host' => env('MAIL_HOST', 'localhost'),
    'port' => (int) env('MAIL_PORT', 587),
    'username' => env('MAIL_USER', ''),
    'password' => env('MAIL_PASS', ''),
    'encryption' => env('MAIL_ENCRYPTION', 'tls'),
    'from' => [
        'address' => env('MAIL_FROM_ADDRESS', 'noreply@example.com'),
        'name' => env('MAIL_FROM_NAME', 'Colibri'),
    ],
];
```

### `config/cors.php`

```php
return [
    'origins' => ['*'],
    'methods' => ['GET', 'POST', 'PUT', 'DELETE'],
    'headers' => ['Content-Type', 'Authorization'],
    'max_age' => 86400,
];
```

### `config/storage.php`

```php
return [
    'default' => 'private',
    'disks' => [
        'private' => ['path' => 'storage/uploads'],
        'public' => ['path' => 'public/uploads'],
    ],
];
```

### `config/upload.php`

```php
return [
    'max_size' => 10 * 1024 * 1024,  // 10 MB
    'allowed_types' => ['image/jpeg', 'image/png', 'image/gif', 'image/webp', 'application/pdf'],
    'blocked_extensions' => ['php', 'phtml', 'phar', 'exe', 'bat', 'sh'],
];
```

### `config/image.php`

```php
return [
    'driver' => env('IMAGE_DRIVER', 'gd'),  // gd or imagick
];
```

### `config/data.php`

```php
return [
    'path' => 'data',
];
```

### `config/commands.php`

```php
return [
    // Register custom CLI commands:
    // App\Commands\MyCommand::class,
];
```

## Environment-Specific Behavior

Use `APP_DEBUG` to control behavior:

| Feature | Debug (`true`) | Production (`false`) |
|---|---|---|
| Error pages | Whoops with stack trace | Generic error page |
| 404 pages | Shows available routes | Standard 404 template |
| `{dump}` / `{dd}` | Outputs debug info | Silently ignored |
| `cache:clear all` | Clears app + views + PHPStan | Clears app + views |
| Log level | `debug` (logs everything) | `error` (errors only) |
