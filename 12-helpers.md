# Helpers

## Str — String Utilities

```php
use Colibri\Support\Str;

Str::slug('Mon Article');          // 'mon-article'
Str::slug('Hello World', '_');     // 'hello_world'
Str::contains('Hello', 'ell');     // true
Str::startsWith('/api/users', '/api');  // true
Str::endsWith('photo.jpg', '.jpg');    // true
Str::limit('A long text...', 10);      // 'A long tex...'
Str::limit('Short', 10);              // 'Short'
Str::random(32);                       // crypto-safe random string
Str::lower('HÉLLO');                   // 'héllo' (multibyte safe)
Str::upper('héllo');                   // 'HÉLLO' (multibyte safe)
```

## Arr — Array Utilities

```php
use Colibri\Support\Arr;

$data = ['user' => ['name' => 'Alice', 'email' => 'alice@test.com']];

Arr::get($data, 'user.name');              // 'Alice'
Arr::get($data, 'user.age', 25);           // 25 (default)
Arr::has($data, 'user.name');              // true
Arr::has($data, 'user.age');               // false

$filtered = Arr::only($data['user'], ['name']);
// ['name' => 'Alice']

$safe = Arr::except($data['user'], ['email']);
// ['name' => 'Alice']
```

## Log — Logging

```php
use Colibri\Support\Log;

Log::debug('Debug info', ['key' => 'value']);
Log::info('User logged in', ['user_id' => 42]);
Log::warning('Slow query', ['ms' => 500]);
Log::error('Payment failed', ['order_id' => 123]);
```

### Configuration

```php
// config/app.php
'log' => [
    'channel' => 'daily',   // 'single' (colibri.log) or 'daily' (2026-04-03.log)
    'level' => 'debug',     // debug, info, warning, error
    'max_files' => 30,      // auto-purge old daily logs
],
```

Logs are written to `storage/logs/`. The log level filters which messages are written — `'error'` only writes errors, `'debug'` writes everything.

Daily logs older than `max_files` days are automatically purged.

## Flash — Flash Messages

```php
use Colibri\Support\Flash;

// Set a flash message
Flash::set('success', 'Profile saved!');
Flash::set('error', 'Invalid email.');

// Get and remove
$msg = Flash::get('success');    // 'Profile saved!' (removed from session)

// Check without removing
Flash::has('error');              // true

// Get all and clear
$all = Flash::all();             // ['success' => '...', 'error' => '...']
```

### In Templates

```latte
{alerts}   {* renders all flash messages using templates/partials/alerts/default.latte *}
```

Or manually:

```latte
{foreach flash() as $type => $message}
    <div class="alert alert-{$type}">{$message}</div>
{/foreach}
```

### Old Input

After a failed form submission, repopulate fields:

```php
// In your route handler
return Colibri\Http\Response::back()
    ->with('error', 'Validation failed.')
    ->withInput();
```

```latte
<input name="email" value="{old('email')}">
```

## Cache

```php
use Colibri\Cache\Cache;

Cache::put('key', 'value', 60);     // store for 60 minutes
Cache::put('key', 'value');          // store forever (TTL = 0)
Cache::get('key');                   // 'value'
Cache::get('missing', 'default');    // 'default'
Cache::has('key');                   // true
Cache::forget('key');                // remove
Cache::clear();                     // clear all
```

### Drivers

```env
CACHE_DRIVER=file    # stores in storage/cache/data/ as serialized files
CACHE_DRIVER=db      # stores in _cache table (auto-created)
```

Register a custom driver:

```php
Cache::registerDriver('redis', RedisCacheDriver::class);
```

## Pagination

The `Pagination` object is returned by `DB::paginate()`:

```php
$results = Colibri\Database\DB::paginate('posts', '*', null, page: 2, perPage: 10);

$results->items();        // array of records
$results->currentPage();  // 2
$results->totalPages();   // 5
$results->total();        // 50
$results->perPage();      // 10
$results->hasPages();     // true
$results->hasPrev();      // true
$results->hasNext();      // true
$results->prevPage();     // 1
$results->nextPage();     // 3
$results->pages();        // [1, 2, 3, '...', 5]
```

### In Templates

```latte
{pagination $results}                    {* default style *}
{pagination $results, type: 'simple'}    {* prev/next only *}
{pagination $results, type: 'minimal'}   {* arrows + page count *}
{pagination $results, type: 'load-more'} {* HTMX load more button *}
```

Customize by editing templates in `templates/partials/pagination/`.

## Global Helper Functions

| Function | Description |
|---|---|
| `env('KEY', 'default')` | Get environment variable |
| `base_path('routes/web')` | Resolve path from project root |
| `view('template', $data)` | Render a Latte template |
| `t('key', ['param' => 'val'])` | Translate a key |
| `url(path: 'about')` | Build a locale-aware URL |
| `flash('success')` | Get a flash message |
| `flash()` | Get all flash messages |
| `old('email', '')` | Get old input value |
| `csrf_token()` | Get CSRF token |
| `csrf_field()` | Generate hidden CSRF input |
| `site_title($page->title)` | Formatted title with site name |
