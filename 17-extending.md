# Extending Colibri

## Driver Pattern

Colibri uses a consistent driver pattern for extensible components. Each supports built-in drivers and custom ones via `registerDriver()`.

### Available Drivers

| Component | Config Key | Built-in | Register |
|---|---|---|---|
| Auth | `AUTH_DRIVER` | `db`, `file` | `Auth::registerDriver()` |
| Cache | `CACHE_DRIVER` | `file`, `db` | `Cache::registerDriver()` |
| Mail | `MAIL_DRIVER` | `smtp`, `sendmail`, `log` | `Mail::registerDriver()` |

### Creating a Custom Driver

1. Implement the interface for the component:

```php
use Colibri\Mail\Interfaces\MailDriverInterface;
use Colibri\Mail\Message;

class SendGridMailDriver implements MailDriverInterface
{
    public function send(Message $message): bool
    {
        // SendGrid API implementation...
        return true;
    }
}
```

2. Register it:

```php
use Colibri\Mail\Mail;

Mail::registerDriver('sendgrid', SendGridMailDriver::class);
```

3. Set the config:

```env
MAIL_DRIVER=sendgrid
```

### Interfaces

| Component | Interface | Method(s) |
|---|---|---|
| Auth | `Colibri\Auth\Interfaces\AuthDriverInterface` | `create()`, `findByEmail()`, `findById()` |
| Cache | `Colibri\Cache\Interfaces\CacheDriverInterface` | `get()`, `put()`, `has()`, `forget()`, `clear()` |
| Mail | `Colibri\Mail\Interfaces\MailDriverInterface` | `send(Message)` |

## Custom CLI Commands

See [CLI — Custom Commands](14-cli.md#custom-commands).

## Custom Middleware

See [Middleware — Custom Middleware](06-middleware.md#custom-middleware).

## Custom Latte Tags

If you need a custom Latte tag beyond what Colibri provides, you can extend the Latte engine directly:

```php
// In a route or bootstrap
$engine = Colibri\View\View::engine();
$engine->addExtension(new MyLatteExtension());
```

See the [Latte documentation](https://latte.nette.org/en/extending-latte) for creating extensions.

## Publishing a Package

Convention for Colibri packages:

```
colibri-php/sendgrid-mail-driver    → SendGrid mail driver
colibri-php/s3-storage-driver       → S3 storage driver
colibri-php/redis-cache-driver      → Redis cache driver
```

A package provides:

1. A driver class implementing the appropriate interface
2. Installation instructions: `composer require` + `registerDriver()` + `.env` config

## Framework Architecture

```
src/
├── App.php, Config.php, CLI.php, helpers.php   ← Bootstrap
├── Auth/           ← Authentication (Auth, Drivers, Interfaces)
├── Cache/          ← Caching (Cache, Drivers, Interfaces, Commands)
├── CLI/            ← CLI commands (Commands, Interfaces)
├── Database/       ← Database (DB, Migration, Commands, Interfaces)
├── Exceptions/     ← Exceptions (HttpException)
├── Http/           ← HTTP (Request, Response, Router, Commands)
├── Mail/           ← Email (Mail, Message, Drivers, Interfaces)
├── Middleware/      ← Middleware (Pipeline, built-in, Commands, Interfaces)
├── Storage/        ← Files (File, Data)
├── Support/        ← Utilities (Str, Arr, Log, Flash, Image, Pagination)
└── View/           ← Templating (View, Page, Assets, Lang, Vite, Latte nodes)
```

Each domain owns its interfaces, drivers, and commands.
