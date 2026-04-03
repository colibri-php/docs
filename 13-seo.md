# SEO

## Page Title

The `site_title()` function formats the page title with the site name:

```latte
<title>{site_title($page->title)}</title>
```

Produces: `About | MyApp` (configurable format).

### Configuration

```php
// config/app.php
'name' => env('APP_NAME', 'Colibri'),
'title_format' => ':title | :site',
```

Available placeholders: `:title` and `:site`. Examples:

- `:title | :site` → `About | MyApp`
- `:site - :title` → `MyApp - About`
- `:title` → `About`

When the page title equals the site name, only the site name is returned (no duplication).

## Meta Description

```latte
{page title: 'About', description: 'Learn about our company.'}
```

Or in PHP:

```php
$page->description = 'Learn about our company.';
```

## The `{meta}` Tag

Place `{meta}` in your layout to output all dynamic meta tags:

```latte
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{site_title($page->title)}</title>
    {meta}
    {styles}
</head>
```

`{meta}` generates:

- `<meta name="description">` — if `$page->description` is set
- `<meta property="og:*">` — OpenGraph tags
- `<meta name="twitter:*">` — Twitter Card tags
- `<link rel="canonical">` — canonical URL
- Any custom meta from `$page->meta`

## OpenGraph

```php
$page->og('title', 'Custom OG Title');
$page->og('image', 'https://example.com/hero.jpg');
$page->og('type', 'article');
```

If `og:title` or `og:description` are not explicitly set, they inherit from `$page->title` and `$page->description`.

## Twitter Cards

```php
$page->twitter('card', 'summary_large_image');
$page->twitter('site', '@mysite');
$page->twitter('creator', '@author');
```

Twitter tags only appear if at least one `og()` or `twitter()` call is made. The default card type is `summary`.

## Canonical URL

```php
$page->canonical('/about');
```

Generates:

```html
<link rel="canonical" href="/about">
```

## Custom Meta Tags

```php
$page->meta['robots'] = 'noindex, nofollow';
$page->meta['author'] = 'Alice';
```

## Manual Approach

If you need full control, skip `{meta}` and write the tags yourself:

```latte
<meta n:if="$page->description" name="description" content="{$page->description}">
{foreach $page->meta as $name => $content}
    <meta name="{$name}" content="{$content}">
{/foreach}
```

Both approaches can coexist — use `{meta}` for convenience, manual tags for edge cases.
