# Templating

Colibri uses [Latte](https://latte.nette.org) as its templating engine, with custom tags and automatic layout injection.

## Layouts

Templates are automatically wrapped in `templates/layouts/default.latte`:

```latte
<!DOCTYPE html>
<html lang="{locale()}">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{site_title($page->title)}</title>
    {meta}
    {styles}
</head>
<body>
    {alerts}
    {block content}{/block}
    {scripts}
</body>
</html>
```

### Switching Layouts

Set the layout per page with `{page}`:

```latte
{page layout: 'admin'}

{block content}
<h1>Admin Panel</h1>
{/block}
```

This resolves to `templates/layouts/admin.latte`. Layouts are flat files in `templates/layouts/` — no subdirectories.

### No Layout

HTMX requests automatically skip the layout and render only the content block. The `$htmx` variable is available in templates:

```latte
{if $htmx}
    <p>This is a partial response.</p>
{else}
    <p>This is a full page.</p>
{/if}
```

## The `$page` Object

Every template receives a `$page` object for metadata:

```latte
{page title: 'About', description: 'Our story'}
```

Or in PHP:

```php
$page->title = 'About';
$page->description = 'Our story';
```

### Properties

| Property | Default | Description |
|---|---|---|
| `$page->title` | `APP_NAME` | Page title (falls back to app name) |
| `$page->description` | `''` | Meta description |
| `$page->layout` | `'default'` | Layout name |
| `$page->meta` | `[]` | Custom meta tags (key/value) |

### SEO Methods

```php
$page->og('title', 'Custom OG Title');
$page->og('image', '/images/hero.jpg');
$page->twitter('card', 'summary_large_image');
$page->twitter('site', '@mysite');
$page->canonical('/about');
```

OpenGraph and Twitter Card tags inherit `$page->title` and `$page->description` as defaults. Explicit calls override them.

## Custom Tags

### `{page}`

Declare page metadata:

```latte
{page title: 'About', description: 'Our story', layout: 'admin'}
```

### `{t}` — Translation

```latte
{t 'welcome'}
{t 'cart.items', count: 3}
```

Also available as a filter and function:

```latte
{'welcome'|t}
{t('welcome')}
```

### `{meta}`

Outputs all dynamic meta tags (description, OpenGraph, Twitter Cards, canonical, custom meta):

```latte
{meta}
```

### `{styles}` and `{scripts}`

Render all collected CSS and JS (from cascading `_styles.css`/`_scripts.js` and `{css}`/`{js}` tags):

```latte
{styles}   {* in <head> *}
{scripts}  {* before </body> *}
```

### `{alerts}`

Render flash message alerts using `templates/partials/alerts/default.latte`:

```latte
{alerts}
```

### `{css}` and `{js}`

Push a CSS or JS file to the stack:

```latte
{css 'admin.css'}
{js 'chart.js'}
{js 'analytics.js', defer}
```

Files are resolved from `public/assets/`. Full URLs are passed through:

```latte
{css 'https://cdn.example.com/lib.css'}
```

### `{image}`

Render an image with optional processing:

```latte
{image 'images/photo.jpg', alt: 'A photo'}
{image 'images/photo.jpg', resize: 400, alt: 'Resized'}
{image 'images/photo.jpg', thumbnail: 100, alt: 'Thumb'}
{image 'images/photo.jpg', crop: 300, cropHeight: 200, alt: 'Cropped'}
```

Processed images are cached in `public/cache/images/` and served directly.

### `{pagination}`

Render pagination controls:

```latte
{pagination $results}
{pagination $results, type: 'simple'}
{pagination $results, type: 'minimal'}
{pagination $results, type: 'load-more'}
```

Templates are in `templates/partials/pagination/`. Available types: `default`, `simple`, `minimal`, `load-more`.

### `{vite}`

Include Vite-processed assets (optional, for projects using Tailwind/Vue/Svelte):

```latte
{vite 'resources/css/app.css'}
{vite 'resources/js/app.js'}
```

In dev mode, connects to the Vite dev server for HMR. In production, reads `public/build/manifest.json`.

### `{redirect}`

```latte
{redirect '/dashboard'}
{redirect '/new-url', permanent}
```

### `{error}`

Trigger an error page:

```latte
{error 404}
{error 403}
```

### `{dump}` and `{dd}`

Debug helpers (only active when `APP_DEBUG=true`):

```latte
{dump $variable}
{dd $variable}
```

## Template Functions

Available in Latte expressions:

| Function | Description |
|---|---|
| `t('key')` | Translate a key |
| `url(path: 'about')` | Build a URL with locale support |
| `url(locale: 'fr')` | Current page in another locale |
| `locale()` | Active locale (`'en'`, `'fr'`) |
| `locales()` | All configured locales with prefixes |
| `flash('success')` | Get a flash message |
| `flash()` | Get all flash messages |
| `old('email')` | Get old input for form repopulation |
| `csrf_token()` | Get the CSRF token |
| `csrf_field()` | Generate a hidden CSRF input |
| `site_title($page->title)` | Formatted title with site name |

## Template Filters

| Filter | Description |
|---|---|
| `{$text\|t}` | Translate |
| `{$path\|resize:400}` | Resize an image |
| `{$path\|crop:300,200}` | Crop an image |
| `{$path\|thumbnail:100}` | Generate a thumbnail |

## Partials

Reusable template fragments live in `templates/partials/`:

```
templates/partials/
├── alerts/
│   └── default.latte
├── pagination/
│   ├── default.latte
│   ├── simple.latte
│   ├── minimal.latte
│   └── load-more.latte
```

You can customize these by editing the files directly.

## The `view()` Helper

Render a template programmatically from PHP:

```php
$html = view('emails/welcome', ['name' => 'Alice']);
```

Templates are resolved from `templates/` by default. Absolute paths are also accepted.
