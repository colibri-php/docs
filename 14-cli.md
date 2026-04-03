# CLI

## Usage

```bash
php colibri <command> [arguments]
```

Run without arguments to see all available commands:

```bash
php colibri
```

## Built-in Commands

### Server

| Command | Description |
|---|---|
| `serve [host] [port]` | Start dev server (default: `localhost:8000`) |

### Database

| Command | Description |
|---|---|
| `migrate` | Run pending migrations |
| `migrate:down` | Rollback the last batch |
| `migrate:status` | Show pending migrations |

> Migrate commands require a database connection. They display an error if no database is configured.

### Cache

| Command | Description |
|---|---|
| `cache:clear [target]` | Clear cache. Targets: `all` (default), `app`, `views`, `phpstan` |

In debug mode, `cache:clear all` also clears the PHPStan cache. In production, only `app` and `views` are cleared.

### Scaffolding

| Command | Description |
|---|---|
| `make:page <path> [--latte\|--php\|--both]` | Create a page in `routes/web/` |
| `make:api <path>` | Create an API endpoint in `routes/api/` |
| `make:middleware <name>` | Create a middleware in `middleware/` |
| `make:migration <name>` | Create a migration in `migrations/` |

### Maintenance

| Command | Description |
|---|---|
| `down [--secret=xxx]` | Enter maintenance mode (returns 503) |
| `up` | Exit maintenance mode |

The `--secret` flag allows bypassing maintenance via `/?secret=xxx` (sets a cookie).

### Info

| Command | Description |
|---|---|
| `routes` | List all registered web and API routes |

## Custom Commands

### Creating a Command

Implement `CommandInterface`:

```php
<?php
// app/Commands/SitemapCommand.php

namespace App\Commands;

use Colibri\CLI\Interfaces\CommandInterface;

class SitemapCommand implements CommandInterface
{
    public function signature(): string
    {
        return 'sitemap:generate';
    }

    public function description(): string
    {
        return 'Generate the sitemap.xml file';
    }

    public function handle(array $args): int
    {
        // Generate sitemap...
        echo "  ✓ Sitemap generated.\n";

        return 0;  // 0 = success, 1 = error
    }
}
```

### Registering

Add it to `config/commands.php`:

```php
return [
    App\Commands\SitemapCommand::class,
];
```

The command is now available:

```bash
php colibri sitemap:generate
```

### Command Groups

Commands are automatically grouped by prefix in the help screen. Use `:` to create groups:

- `sitemap:generate` — appears under the `sitemap` group
- `import:users` — appears under the `import` group
- `backup` — appears under `Commands` (no prefix)
