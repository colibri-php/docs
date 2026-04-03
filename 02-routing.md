# Routing

Colibri uses file-based routing. Create a file, get a route. No registration needed.

## Web Routes

Files in `routes/web/` serve HTML pages. Both `.php` and `.latte` files are supported.

| File | URL |
|---|---|
| `routes/web/index.latte` | `/` |
| `routes/web/about.latte` | `/about` |
| `routes/web/blog/index.latte` | `/blog` |
| `routes/web/contact.php` | `/contact` |

### Three Rendering Modes

**1. `.latte` only — static page**

```latte
{* routes/web/about.latte *}
{page title: 'About'}

{block content}
<h1>About Us</h1>
{/block}
```

**2. `.php` only — logic**

```php
<?php
// routes/web/dashboard.php

if (! Colibri\Auth\Auth::check()) {
    return Colibri\Http\Response::redirect('/login');
}

echo '<h1>Dashboard</h1>';
```

**3. `.php` + `.latte` twins — logic + template**

When both `profile.php` and `profile.latte` exist in the same directory, the PHP file runs first and its variables are automatically injected into the Latte template.

```php
<?php
// routes/web/profile.php
$user = Colibri\Auth\Auth::user();
$posts = Colibri\Database\DB::select('posts', '*', ['author_id' => $user['id']]);
```

```latte
{* routes/web/profile.latte *}
{page title: $user['name']}

{block content}
<h1>{$user['name']}</h1>
{foreach $posts as $post}
    <article>{$post['title']}</article>
{/foreach}
{/block}
```

### Available Variables

Every page file receives these variables:

| Variable | Type | Description |
|---|---|---|
| `$request` | `Colibri\Http\Request` | The current request |
| `$app` | `Colibri\App` | The application instance |
| `$page` | `Colibri\View\Page` | Page metadata object |
| `$params` | `array` | Dynamic segment parameters |
| `$htmx` | `bool` | Whether this is an HTMX request |

## API Routes

Files in `routes/api/` serve JSON automatically. Only `.php` files are supported.

```php
<?php
// routes/api/users/index.php

return [
    ['id' => 1, 'name' => 'Alice'],
    ['id' => 2, 'name' => 'Bob'],
];
```

`GET /api/users` returns:

```json
[{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]
```

API routes automatically set `Content-Type: application/json` and encode return values as JSON. Errors are also returned as JSON:

```json
{"error": "Not Found"}
```

## Dynamic Segments

Use `[param]` in filenames to capture URL segments:

```
routes/api/users/[id].php    → /api/users/42   → $params['id'] = '42'
routes/web/blog/[slug].latte → /blog/my-post   → $params['slug'] = 'my-post'
```

```php
<?php
// routes/api/users/[id].php

$user = Colibri\Database\DB::get('users', '*', ['id' => (int) $params['id']]);

if (! $user) {
    return Colibri\Http\Response::json(['error' => 'User not found'], 404);
}

return $user;
```

### Priority

Static routes always win over dynamic routes. If both `routes/api/users/me.php` and `routes/api/users/[id].php` exist, `/api/users/me` matches the static file.

### Multi-Segment

```
routes/api/posts/[postId]/comments/[commentId].php
→ /api/posts/5/comments/12
→ $params['postId'] = '5', $params['commentId'] = '12'
```

## HTTP Methods

Return an array of method handlers to handle different HTTP methods in a single file:

```php
<?php
// routes/api/posts/index.php

return [
    'GET' => function ($request, $params) {
        return Colibri\Database\DB::select('posts', '*');
    },
    'POST' => function ($request, $params) {
        $errors = $request->validate([
            'title' => ['required'],
            'body' => ['required'],
        ]);

        if ($errors) {
            return Colibri\Http\Response::json(['errors' => $errors], 422);
        }

        Colibri\Database\DB::insert('posts', $request->body());
        return Colibri\Http\Response::json(['created' => true], 201);
    },
];
```

If a request uses a method not defined in the array, Colibri returns 405 Method Not Allowed (JSON for API, HTML for web).

Files without method handlers are treated as GET by default.

## Cascading Conventions

Special files prefixed with `_` cascade from parent directories to children:

```
routes/web/
├── _middleware.php      ← applies to all web routes
├── _styles.css          ← CSS injected on all pages
├── _scripts.js          ← JS injected on all pages
├── index.latte
├── admin/
│   ├── _middleware.php  ← adds to parent middleware
│   ├── _styles.css      ← adds to parent styles
│   └── users.php
```

Visiting `/admin/users` loads middleware from both `routes/web/_middleware.php` and `routes/web/admin/_middleware.php`, and injects CSS from both `_styles.css` files (parent first, then child).

See [Middleware](06-middleware.md) and [Frontend](08-frontend.md) for details.

## Auto Layout

Web routes are automatically wrapped in `templates/layouts/default.latte`. Change the layout per page:

```latte
{page layout: 'admin'}
```

This resolves to `templates/layouts/admin.latte`.

## Generating URLs

Use the `php colibri routes` command to see all registered routes:

```bash
php colibri routes
```

Use the `php colibri make:page` and `php colibri make:api` commands to scaffold new routes:

```bash
php colibri make:page about --latte          # routes/web/about.latte
php colibri make:page dashboard --php        # routes/web/dashboard.php
php colibri make:page profile --both         # routes/web/profile.php + profile.latte
php colibri make:api users/stats             # routes/api/users/stats.php
```
