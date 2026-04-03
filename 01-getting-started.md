# Getting Started

## Requirements

- PHP >= 8.4
- Extensions: `pdo`, `intl`, `mbstring`, `fileinfo`

## Installation

```bash
composer create-project colibri-php/colibri my-project --stability=alpha
cd my-project
cp .env.example .env
php colibri serve
```

Visit `http://localhost:8000`.

## Project Structure

```
my-project/
├── colibri               # CLI entry point
├── config/               # Configuration files
│   ├── app.php           # App name, debug, timezone, HTTPS, logging, title format
│   ├── auth.php          # Auth driver, roles, login path
│   ├── cache.php         # Cache driver and path
│   ├── commands.php      # Custom CLI commands
│   ├── cors.php          # CORS settings
│   ├── data.php          # Flat-file storage path
│   ├── database.php      # Database driver and credentials
│   ├── i18n.php          # Default locale, fallback, URL prefixes
│   ├── image.php         # Image processing driver (GD/Imagick)
│   ├── mail.php          # Mail driver and SMTP settings
│   ├── storage.php       # Storage disks (private/public)
│   └── upload.php        # Upload limits and allowed types
├── data/                 # Flat-file JSON storage (Data::)
├── locales/              # Translation files
│   ├── en.json
│   └── fr.json
├── middleware/            # Custom middleware (anonymous classes)
├── migrations/            # Database migrations
│   └── 001_create_users.php
├── public/               # Web root
│   ├── index.php         # Entry point
│   └── .htaccess         # Apache rewrite rules
├── routes/
│   ├── web/              # HTML pages (.php, .latte)
│   │   ├── _styles.css   # Cascading CSS (auto-injected)
│   │   ├── _scripts.js   # Cascading JS (auto-injected)
│   │   └── index.latte   # Homepage
│   └── api/              # JSON endpoints (.php only)
│       └── users/
│           └── index.php
├── storage/
│   ├── cache/            # Cache files, compiled views, PHPStan
│   ├── logs/             # Log files
│   └── uploads/          # Uploaded files (private disk)
├── templates/
│   ├── layouts/          # Latte layouts
│   │   └── default.latte
│   ├── partials/         # Reusable partials
│   │   ├── alerts/
│   │   └── pagination/
│   ├── emails/           # Email templates
│   └── errors/           # Error pages (404, 500, 503...)
├── tests/                # Project tests
├── .env                  # Environment variables (not committed)
├── .env.example          # Environment template
└── composer.json
```

## Environment

Copy `.env.example` to `.env` and adjust values. Key settings:

```env
APP_NAME=MyApp            # Used in site_title() and page title fallback
APP_DEBUG=true            # Enables Whoops error pages and debug logging
APP_TIMEZONE=America/Montreal

DB_DRIVER=sqlite          # sqlite, mysql, or pgsql
DB_PATH=storage/database.sqlite

AUTH_DRIVER=db            # db or file (file = no database needed)
CACHE_DRIVER=file         # file or db
MAIL_DRIVER=log           # log, smtp, or sendmail
```

## First Page

Create `routes/web/about.latte`:

```latte
{page title: 'About'}

{block content}
<h1>About Us</h1>
<p>Welcome to our site.</p>
{/block}
```

Visit `http://localhost:8000/about` — Colibri automatically wraps it in the default layout.

## First API Endpoint

Create `routes/api/status/index.php`:

```php
<?php

return ['status' => 'ok', 'time' => date('c')];
```

Visit `http://localhost:8000/api/status` — returns JSON automatically.

## CLI

```bash
php colibri                  # Show all available commands
php colibri serve            # Start development server
php colibri migrate          # Run database migrations
php colibri make:page about  # Create a new page
php colibri make:api health  # Create a new API endpoint
php colibri cache:clear      # Clear all caches
```

## No Database Mode

Colibri works without a database. Set `AUTH_DRIVER=file` in `.env` and use `Data::` for storage instead of `DB::`. Users are stored as JSON files in `data/users/`.

## Next Steps

- [Routing](02-routing.md) — How file-based routing works
- [Templating](03-templating.md) — Latte templates, layouts, custom tags
- [Database](04-database.md) — DB::, migrations, flat-file storage
