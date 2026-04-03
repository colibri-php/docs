# Frontend & Assets

## Cascading CSS/JS

Place `_styles.css` and `_scripts.js` files in route directories. They cascade from parent to child and are automatically injected into the layout via `{styles}` and `{scripts}`.

```
routes/web/
├── _styles.css          ← global styles
├── _scripts.js          ← global scripts
├── index.latte
├── admin/
│   ├── _styles.css      ← admin-specific (added after global)
│   └── users.php
```

Visiting `/admin/users` loads both `routes/web/_styles.css` and `routes/web/admin/_styles.css`, in that order.

The content is injected inline as `<style>` and `<script>` tags. No separate HTTP requests.

## Asset Tags

### `{css}` — Include a CSS file

```latte
{css 'admin.css'}
```

Resolves from `public/assets/`. Generates a `<link>` tag pushed to the `{styles}` stack.

Full URLs are passed through:

```latte
{css 'https://cdn.example.com/lib.css'}
```

### `{js}` — Include a JS file

```latte
{js 'chart.js'}
{js 'analytics.js', defer}
{js 'tracker.js', async}
```

### Layout Tags

In your layout, use `{styles}` and `{scripts}` to output all collected assets:

```latte
<head>
    {styles}
</head>
<body>
    {block content}{/block}
    {scripts}
</body>
```

Everything converges into these two tags: cascading `_styles.css`/`_scripts.js`, `{css}` tags, and `{js}` tags.

## Image Processing

### `{image}` Tag

```latte
{image 'images/photo.jpg', alt: 'A photo'}
{image 'images/photo.jpg', resize: 400, alt: 'Resized to 400px wide'}
{image 'images/photo.jpg', resize: 400, resizeHeight: 300, alt: 'Exact dimensions'}
{image 'images/photo.jpg', thumbnail: 100, alt: 'Square thumbnail'}
{image 'images/photo.jpg', crop: 300, cropHeight: 200, alt: 'Cropped'}
```

Images without transforms resolve from `public/`. Processed images are cached in `public/cache/images/` and served directly — no reprocessing.

### PHP API

```php
use Colibri\Support\Image;

$url = Image::resize('images/photo.jpg', 400);       // width 400, proportional height
$url = Image::resize('images/photo.jpg', 400, 300);  // exact dimensions
$url = Image::thumbnail('images/photo.jpg', 100);     // square crop + resize
$url = Image::crop('images/photo.jpg', 300, 200);     // center crop
```

Returns a URL path like `/cache/images/abc123.jpg`. Returns `null` if the source file doesn't exist.

Source files are looked up in order: absolute path, `storage/uploads/`, `public/`.

### Configuration

```php
// config/image.php
return [
    'driver' => env('IMAGE_DRIVER', 'gd'),  // gd or imagick
];
```

## Vite Integration (Optional)

For projects using Tailwind, Vue, Svelte, or TypeScript.

### Template Tag

```latte
{vite 'resources/css/app.css'}
{vite 'resources/js/app.js'}
```

### Behavior

- **Dev mode** (`APP_DEBUG=true` + Vite dev server running): injects HMR client and dev server URLs
- **Production** (after `npm run build`): reads `public/build/manifest.json` and outputs hashed filenames

If no Vite dev server is running and no manifest exists, the tag outputs nothing silently.

### Configuration

Set the Vite dev server URL in `config/app.php`:

```php
'vite_host' => 'http://localhost:5173',  // default
```

## HTMX Support

Colibri has native HTMX support. No build step required — include HTMX via CDN:

```latte
{js 'https://unpkg.com/htmx.org@2.0.4', defer}
```

### Request Detection

```php
$request->isHtmx();      // true if HX-Request header present
$request->htmxTarget();   // HX-Target header value
$request->htmxTrigger();  // HX-Trigger header value
$request->htmxBoosted();  // true if HX-Boosted
```

### Partial Rendering

HTMX requests automatically render without the layout — only the `{block content}` is returned. The `$htmx` variable is available in templates:

```latte
{if $htmx}
    {* Partial content for HTMX swap *}
{/if}
```

### Response Helpers

```php
use Colibri\Http\Response;

Response::htmxRedirect('/dashboard');     // HX-Redirect header
Response::htmxRefresh();                  // HX-Refresh header
Response::htmxSwap('<p>Done!</p>');       // HTML fragment
Response::htmxSwap('<p>Saved</p>')->htmxTrigger('formSaved');  // trigger client event
```
