# Middleware

Middleware runs before (and optionally after) your route handler. Colibri uses file-based middleware with cascading from parent to child directories.

## Defining Middleware

Create a `_middleware.php` file in any route directory. It returns an array of middleware names:

```php
<?php
// routes/web/_middleware.php
return ['headers', 'csrf'];
```

```php
<?php
// routes/api/_middleware.php
return ['json', 'cors', 'rate:60,1'];
```

```php
<?php
// routes/web/admin/_middleware.php
return ['auth:admin'];
```

## Cascade

Middleware cascades from parent to child. For a request to `/admin/users`:

1. `routes/web/_middleware.php` → `['headers', 'csrf']`
2. `routes/web/admin/_middleware.php` → `['auth:admin']`

The full chain executed: `headers → csrf → auth:admin → route handler`.

## Built-in Middleware

### `auth` — Authentication

Without parameters: redirects to login if not authenticated.

```php
return ['auth'];
```

With role: returns 403 if the user doesn't have the required role.

```php
return ['auth:admin'];
return ['auth:admin,editor'];  // multiple roles
```

The auth middleware stores the intended URL in the session for redirect-back after login.

### `csrf` — CSRF Protection

Validates a `_token` field on POST, PUT, PATCH, DELETE requests. Returns 403 if missing or invalid.

```php
return ['csrf'];
```

In your forms, include the CSRF field:

```latte
<form method="POST">
    {csrf_field()|noescape}
    <input name="email" value="{old('email')}">
    <button>Submit</button>
</form>
```

The token can also be sent via the `X-CSRF-Token` header (for AJAX requests).

### `cors` — Cross-Origin Resource Sharing

Reads configuration from `config/cors.php`:

```php
// config/cors.php
return [
    'origins' => ['*'],
    'methods' => ['GET', 'POST', 'PUT', 'DELETE'],
    'headers' => ['Content-Type', 'Authorization'],
    'max_age' => 86400,
];
```

Handles OPTIONS preflight requests automatically.

### `rate` — Rate Limiting

```php
return ['rate:60,1'];  // 60 requests per 1 minute
return ['rate:5,1'];   // 5 requests per 1 minute
```

Returns 429 Too Many Requests when exceeded. Tracks requests per IP + route in a `_rate_limits` table (auto-created).

### `headers` — Security Headers

Adds standard security headers:

- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `Referrer-Policy: strict-origin-when-cross-origin`
- `Strict-Transport-Security` (when `APP_HTTPS=true`)

### `json` — API Mode

Forces `Content-Type: application/json` on the response.

## Custom Middleware

Create a file in `middleware/` that returns an anonymous class:

```php
<?php
// middleware/throttle.php

use Colibri\Middleware\Interfaces\MiddlewareInterface;
use Colibri\Http\Request;
use Colibri\Http\Response;

return new class implements MiddlewareInterface {
    public function handle(Request $request, callable $next, string ...$params): Response
    {
        // Before the request...

        $response = $next($request);

        // After the request...

        return $response;
    }
};
```

Use it by name in `_middleware.php`:

```php
return ['throttle'];
```

Generate a middleware scaffold:

```bash
php colibri make:middleware throttle
```

### Resolution Order

1. **Project middleware** (`middleware/`) — checked first
2. **Built-in middleware** (`Colibri\Middleware\`) — fallback

A project middleware with the same name as a built-in one will override it.

### Parameters

Middleware can receive parameters via the colon syntax:

```php
// _middleware.php
return ['rate:30,2'];  // 30 requests per 2 minutes
```

Parameters are passed as string arguments to the `handle` method:

```php
public function handle(Request $request, callable $next, string ...$params): Response
{
    $max = (int) ($params[0] ?? 60);
    $minutes = (int) ($params[1] ?? 1);
    // ...
}
```

## Suggested Setup

```
routes/web/_middleware.php   → ['headers', 'csrf']
routes/api/_middleware.php   → ['json', 'cors', 'rate:60,1']
routes/web/admin/_middleware.php → ['auth:admin']
```
