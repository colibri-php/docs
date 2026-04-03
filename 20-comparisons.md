# Colibri vs. Other Frameworks

Colibri is heavily inspired by Laravel — its elegance, conventions, and developer experience. But Laravel is a Swiss Army knife. If you only need the screwdriver, carrying the whole knife is unnecessary weight.

Colibri is the screwdriver. It does fewer things, but it does them simply and well. No service container, no Eloquent, no queues — just file-based routing, templates, and the essentials to ship a site.

This is an honest comparison. Colibri is not a replacement for Laravel or Symfony — it's a different tool for different needs.

## At a Glance

| Feature | Colibri | Laravel | Symfony | Slim | CakePHP |
|---|---|---|---|---|---|
| Routing | File-based (core) | Registered (Folio for file-based) | Registered | Registered | Convention |
| Templating | Latte | Blade | Twig | Any | PHP/Twig |
| ORM | None (Medoo DBAL) | Eloquent | Doctrine | None | Built-in |
| Auth | Built-in (simple) | Full (Sanctum, Passport) | Security Bundle | None | Built-in |
| CLI | Built-in (simple) | Artisan (powerful) | Console (powerful) | None | Bake |
| Queue/Jobs | No | Yes | Messenger | No | No |
| Events | No | Yes | EventDispatcher | No | Yes |
| IoC Container | No | Yes | Yes | Yes | No |
| Middleware | Yes | Yes | Yes | Yes | Yes |
| i18n | Built-in | Built-in | Translation | No | Built-in |
| Cache | File/DB | Redis/Memcached/File | Redis/Memcached/File | No | File/Redis |
| Mail | PHPMailer | SwiftMailer/Symfony | Symfony Mailer | No | Built-in |
| DB optional by design | Yes (auth, cache, content) | DB expected | DB expected | Yes (no auth/cache) | DB expected |
| File count (core) | ~70 | ~1,400+ | ~2,000+ | ~30 | ~800+ |
| Learning curve | Low | Medium | High | Low | Medium |
| PHP version | 8.4+ | 8.2+ | 8.2+ | 8.1+ | 8.1+ |

## Colibri vs. Laravel

### Choose Colibri when:
- You want file-based routing as the core paradigm (not a bolt-on package)
- You need a site that works without a database
- You want something you can understand completely in a day
- You're building a portfolio, blog, landing page, or micro CMS
- You want i18n built-in without packages
- You prefer convention over configuration

> **Note:** Laravel offers file-based routing via [Folio](https://laravel.com/docs/folio), a first-party package. Colibri's file-based routing is built into the core — it's the only way to define routes, not an optional add-on. Colibri also extends the file-based convention to middleware (`_middleware.php`) and assets (`_styles.css`, `_scripts.js`), which Folio doesn't.

### Choose Laravel when:
- You need queues, jobs, event broadcasting
- You need a full ORM with relationships, migrations, seeders
- You need built-in API authentication (Sanctum, Passport)
- You're building a SaaS, marketplace, or large application
- You need the massive ecosystem (Nova, Forge, Vapor, Horizon)
- You need WebSocket support
- Your team already knows Laravel

### What Colibri doesn't have:
- No Eloquent ORM (Colibri uses Medoo for raw queries or flat-file JSON)
- No queue system
- No event system
- No service container / dependency injection
- No scheduled tasks
- No API rate limiting per-user (only per-IP)
- No built-in testing helpers for HTTP requests

## Colibri vs. Symfony

### Choose Colibri when:
- You want to ship in hours, not days
- You don't need enterprise architecture
- You want zero XML/YAML configuration
- You want a single `_styles.css` instead of Webpack Encore

### Choose Symfony when:
- You need a modular, component-based architecture
- You need Doctrine ORM with complex entity relationships
- You're building an enterprise application
- You need the Symfony ecosystem (bundles, Flex recipes)
- You need a full security system (voters, firewalls, LDAP)
- Your team is trained on Symfony

### What Colibri doesn't have:
- No dependency injection container
- No form builder component
- No entity/repository pattern
- No Doctrine
- No Symfony Console (Colibri has its own simpler CLI)
- No Messenger (queues/async)

## Colibri vs. Slim

### Choose Colibri when:
- You want file-based routing instead of registered routes
- You want templating, auth, mail, i18n built-in
- You want a project skeleton with opinions (layout, config, structure)

### Choose Slim when:
- You want absolute minimal overhead
- You want to choose every component yourself
- You're building a pure API with no views
- You need PSR-7/PSR-15 compliance

### What Slim has that Colibri doesn't:
- PSR-7 request/response
- PSR-15 middleware standard
- Smaller footprint

### What Colibri has that Slim doesn't:
- Templating engine
- Authentication
- i18n
- Mail
- CLI tools
- File-based routing
- Cascading assets and middleware

## Colibri vs. CakePHP

### Choose Colibri when:
- You want file-based routing
- You want to work without a database
- You want something lighter and more modern (PHP 8.4)

### Choose CakePHP when:
- You want convention-over-configuration with a full ORM
- You need bake (code generation)
- You want built-in CRUD scaffolding
- You're maintaining a legacy CakePHP project

## Colibri vs. Static Site Generators (Hugo, Jekyll, Astro)

### Choose Colibri when:
- You need server-side logic (forms, auth, database)
- You want PHP on shared hosting
- You need dynamic content without a build step
- You want i18n with URL prefixes

### Choose a static site generator when:
- Your site is 100% static content
- You want to deploy on Netlify/Vercel/GitHub Pages
- You don't need forms or authentication
- You want maximum performance (pre-built HTML)

## The Colibri Sweet Spot

Colibri is designed for:

- **Freelancers** building client sites (portfolios, landing pages, small business sites)
- **Side projects** that need a backend but not an enterprise framework
- **Prototypes** that need to ship fast
- **Learning** PHP — the entire framework fits in your head
- **Micro CMS** — content-driven sites with optional database

Colibri is NOT designed for:

- SaaS platforms with complex business logic
- Applications that need queues, WebSockets, or real-time features
- Teams of 10+ developers working on the same codebase
- Projects that require strict PSR compliance
- High-traffic APIs serving millions of requests
