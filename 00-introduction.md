# Introduction

## The Story

Colibri was born from a simple frustration: every time we needed to build a small site — a portfolio, a landing page, a client project — we reached for Laravel out of habit. And every time, we carried thousands of files, dozens of service providers, and an architecture designed for applications ten times the size of what we were building.

We wanted something that felt like Laravel — the elegance, the conventions, the developer experience — but without the weight. Something where creating a page means creating a file. Where a database is optional, not required. Where the entire framework fits in your head.

So we built Colibri. Named after the hummingbird — small, fast, agile.

## The Need

There's a gap in the PHP ecosystem. On one side, you have full-stack frameworks (Laravel, Symfony, CakePHP) designed for complex applications. On the other, you have micro-frameworks (Slim, Mezzio) that give you a router and nothing else.

Colibri sits in between. It's a micro-framework with opinions — just enough structure to be productive, just few enough features to stay simple.

It's for:
- Freelancers building client sites
- Developers prototyping fast
- Small teams shipping content-driven sites
- Anyone who wants PHP without the ceremony

## Philosophy

Most PHP frameworks solve enterprise problems. Colibri solves a simpler one: **how do I ship a small site without carrying an enterprise framework?**

If you need queues, WebSockets, and an ORM — use Laravel. It's excellent. But if you're building a portfolio, a blog, a landing page, or a small client site, Laravel is a Swiss Army knife when you only need the screwdriver.

Colibri is the screwdriver. Fewer features, less complexity, faster to learn, faster to ship.

## Opinionated by Design

Colibri makes choices so you don't have to. Every dependency was hand-picked for a reason.

### Dependencies (5 packages, zero bloat)

| Package | Why this one |
|---|---|
| [Latte](https://latte.nette.org) | Templating engine with contextual auto-escaping (XSS-safe by default), `n:` attribute syntax, and zero configuration. Created by David Grudl, maintained since 2008. Used by the Nette framework ecosystem. We chose Latte over Blade (Laravel-coupled) and Twig (heavier, Symfony-coupled) because it's framework-agnostic, lightweight, and has the best security model of any PHP template engine. |
| [Medoo](https://medoo.in) | Database abstraction in a single file. Supports SQLite, MySQL, PostgreSQL. No ORM, no entity mapping, no migrations magic — just a clean query builder. When you outgrow it, you'll know. |
| [Valitron](https://github.com/vlucas/valitron) | Input validation without dependencies. Simple rules, simple API. No form objects, no validation services — just `$request->validate([...])`. |
| [PHPMailer](https://github.com/PHPMailer/PHPMailer) | The most battle-tested email library in PHP. 20+ years of SMTP edge cases handled. We wrap it in a driver pattern so you can swap it for SendGrid or Postmark when needed. |
| [phpdotenv](https://github.com/vlucas/phpdotenv) | `.env` file loading. The standard. Nothing more to say. |

### Why not Twig?

Twig is excellent but coupled to the Symfony ecosystem. It uses its own expression language (not PHP), requires registering every function you want to use in templates, and adds complexity for features Colibri doesn't need. Latte uses PHP directly — any PHP function works without registration.

### Why not Blade?

Blade is tightly coupled to Laravel's service container and view system. It can't be extracted cleanly. Latte is framework-agnostic and works anywhere.

### Why not Eloquent?

Eloquent is powerful but heavy — it brings the entire Illuminate database package, events, and the service container. For a micro-framework, Medoo gives you 90% of what you need in a single file. And for projects that don't need a database at all, `Data::` provides flat-file JSON storage.

## Conventions Over Configuration

Colibri favors conventions that feel natural:

- **File = route.** Create `routes/web/about.latte`, visit `/about`. No registration.
- **Underscore = framework.** `_middleware.php`, `_styles.css`, `_scripts.js` are special. Everything else is yours.
- **Cascade = inheritance.** Put `_middleware.php` in a parent directory, it applies to all children. Like CSS cascading, but for server-side logic.
- **Twins = logic + template.** `profile.php` and `profile.latte` in the same directory? The PHP runs first, its variables are injected into the template automatically.

## Database is Optional

Most frameworks assume a database. Colibri doesn't. Set `AUTH_DRIVER=file` and your users are JSON files. Use `Data::` instead of `DB::` and your content is version-controlled Markdown or JSON. Deploy on a $5/month shared host without touching MySQL.

When you need a database, it's one `.env` change away.

## Driver Pattern

Every component that might need a different backend uses the same pattern:

```php
Mail::registerDriver('sendgrid', SendGridMailDriver::class);
Cache::registerDriver('redis', RedisCacheDriver::class);
Auth::registerDriver('workos', WorkOsAuthDriver::class);
```

Today, Colibri ships with simple built-in drivers. Tomorrow, the community can build packages for any backend — without changing the framework.

## Evolution

Colibri is young. It's opinionated today because making decisions early is better than making none. But opinions can change.

The framework will evolve based on:
- **Real-world usage** — features that nobody uses will be removed, features that everybody hacks around will be added
- **Community feedback** — if you have ideas, opinions, or complaints, they're welcome
- **PHP evolution** — Colibri requires PHP 8.4+ and uses property hooks, asymmetric visibility, and modern syntax. It will continue to adopt new PHP features as they land.

What won't change: file-based routing, convention over configuration, and the commitment to staying small.

## What's Next

Colibri is at version 1.0.0-alpha. Here's where it's headed:

- **Service providers** — a unified way for packages to register drivers, commands, and middleware automatically
- **Storage drivers** — S3, DigitalOcean Spaces, and other cloud storage backends via the existing driver pattern
- **Log drivers** — Datadog, PaperTrail, Sentry integration
- **Starter kits** — ready-to-use project templates (portfolio, blog, API, HTMX app)
- **Documentation site** — a dedicated site built with Colibri itself, serving the Markdown docs
- **Community packages** — the driver pattern is ready for third-party packages (`colibri-php/sendgrid-mail-driver`, etc.)

The goal is not to become Laravel. The goal is to be the best framework for small PHP projects — and to stay that way.
