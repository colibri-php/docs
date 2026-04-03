# Error Handling

## Error Pages

Custom error templates live in `templates/errors/`:

```
templates/errors/
├── 403.latte    ← Forbidden
├── 404.latte    ← Not Found
├── 405.latte    ← Method Not Allowed
├── 429.latte    ← Too Many Requests
├── 500.latte    ← Internal Server Error
└── 503.latte    ← Maintenance
```

If a template exists for the error code, it is rendered. Otherwise, Colibri falls back to a minimal inline HTML page.

### Web vs API

- **Web routes** (`routes/web/`): render the Latte error template
- **API routes** (`routes/api/`): return JSON: `{"error": "Not Found"}`

## Debug Mode

When `APP_DEBUG=true`:

### 500 Errors

[Whoops](https://github.com/filp/whoops) renders a detailed error page with:

- Exception message and class
- File and line number
- Full stack trace
- Source code context
- Request and server variables

Whoops is a dev dependency — it's only available when installed with `composer install` (not `--no-dev`).

### 404 Errors

A debug-friendly 404 page shows:

- The requested URL and HTTP method
- A table of all available routes (path, type, file)

This helps identify typos and missing routes during development.

### API Errors in Debug Mode

API 500 responses include additional detail:

```json
{
    "error": "Internal Server Error",
    "message": "Call to undefined method...",
    "file": "routes/api/users/index.php:12",
    "trace": ["#0 ...", "#1 ..."]
}
```

## Production Mode

When `APP_DEBUG=false`:

- 500 errors show the generic `templates/errors/500.latte` (no stack trace)
- API 500 responses return only `{"error": "Internal Server Error"}`
- All exceptions are logged to `storage/logs/`

## Maintenance Mode

```bash
php colibri down                    # Enter maintenance mode
php colibri down --secret=abc123    # With bypass secret
php colibri up                      # Exit maintenance mode
```

In maintenance mode, all web requests return 503 with `templates/errors/503.latte`.

The `--secret` flag allows bypassing maintenance by visiting `/?secret=abc123`. A cookie is set so subsequent requests are allowed through.

API routes are also blocked during maintenance.

## Throwing HTTP Errors

In your routes:

```php
use Colibri\Exceptions\HttpException;

throw new HttpException(404);
throw new HttpException(403, 'You cannot access this resource.');
```

In Latte templates:

```latte
{error 404}
{error 403}
```

## Logging

All uncaught exceptions are automatically logged before rendering the error page:

```
[2026-04-03 14:30:00] ERROR: Call to undefined method... {"file":"routes/api/...","line":12}
```

See [Helpers — Log](12-helpers.md#log--logging) for the logging API.
