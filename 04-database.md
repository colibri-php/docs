# Database

Colibri provides two data storage systems: `DB::` for SQL databases via [Medoo](https://medoo.in), and `Data::` for flat-file JSON storage.

## DB:: — SQL Database

### Configuration

```env
DB_DRIVER=sqlite
DB_PATH=storage/database.sqlite
```

For MySQL or PostgreSQL:

```env
DB_DRIVER=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=myapp
DB_USER=root
DB_PASS=secret
```

### Querying

```php
use Colibri\Database\DB;

// Select all
$users = DB::select('users', '*');

// Select with conditions
$admins = DB::select('users', '*', ['role' => 'admin']);

// Select specific columns
$names = DB::select('users', ['name', 'email']);

// Get a single record
$user = DB::get('users', '*', ['id' => 1]);

// Count
$total = DB::count('users');
$adminCount = DB::count('users', ['role' => 'admin']);

// Check existence
$exists = DB::has('users', ['email' => 'alice@test.com']);
```

### Insert, Update, Delete

```php
// Insert
DB::insert('users', [
    'name' => 'Alice',
    'email' => 'alice@example.com',
    'password' => password_hash('secret', PASSWORD_BCRYPT),
]);

$id = DB::id(); // Last inserted ID

// Update
DB::update('users', ['name' => 'Alicia'], ['id' => 1]);

// Delete
DB::delete('users', ['id' => 1]);
```

### Pagination

```php
$page = (int) $request->query('page', 1);
$results = DB::paginate('users', '*', null, page: $page, perPage: 10);

// $results is a Pagination object
$results->items();       // Data for current page
$results->currentPage(); // Current page number
$results->totalPages();  // Total pages
$results->total();       // Total records
$results->hasPages();    // More than one page?
$results->hasPrev();     // Has previous page?
$results->hasNext();     // Has next page?
$results->prevPage();    // Previous page number
$results->nextPage();    // Next page number
$results->pages();       // Page numbers with ellipsis [1, 2, '...', 8, 9, 10]
```

In templates:

```latte
{foreach $results->items() as $user}
    <p>{$user['name']}</p>
{/foreach}

{pagination $results}
```

### Raw SQL

```php
DB::exec('CREATE INDEX idx_email ON users(email)');
$stmt = DB::query('SELECT * FROM users WHERE created_at > ?', ['2026-01-01']);
```

### Direct Medoo Access

For advanced queries, access the Medoo instance directly:

```php
$medoo = DB::medoo();
```

See [Medoo documentation](https://medoo.in/doc) for the full API.

## Migrations

Migrations are PHP files in `migrations/` that implement `MigrationInterface`:

```php
<?php
// migrations/001_create_users.php

use Colibri\Database\DB;
use Colibri\Database\Interfaces\MigrationInterface;

return new class implements MigrationInterface {
    public function up(): void
    {
        DB::exec('
            CREATE TABLE users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                email TEXT NOT NULL UNIQUE,
                password TEXT NOT NULL,
                name TEXT NOT NULL DEFAULT "",
                role TEXT NOT NULL DEFAULT "user",
                created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
            )
        ');
    }

    public function down(): void
    {
        DB::exec('DROP TABLE IF EXISTS users');
    }
};
```

### Commands

```bash
php colibri migrate              # Run pending migrations
php colibri migrate:status       # Show pending migrations
php colibri migrate:down         # Rollback the last batch
php colibri make:migration create_posts   # Create a new migration file
```

Migrations are tracked in a `_migrations` table (auto-created). They run in alphabetical order and are idempotent — running `migrate` twice has no effect.

> **Note:** Migrate commands require a database connection. If `AUTH_DRIVER=file` and no database is configured, the commands will display an error message.

## Data:: — Flat-File JSON Storage

For projects that don't need a database, `Data::` stores data as JSON files.

### Configuration

```php
// config/data.php
return [
    'path' => 'data',
];
```

### Structure

```
data/
├── posts/
│   ├── my-first-post.json
│   └── another-post.json
└── pages/
    └── about.json
```

### API

```php
use Colibri\Storage\Data;

// Create or update
Data::put('posts/my-first-post', [
    'title' => 'My First Post',
    'status' => 'published',
    'date' => '2026-04-01',
]);

// Read
$post = Data::get('posts/my-first-post');
// ['title' => 'My First Post', 'status' => 'published', ...]

// Check existence
Data::exists('posts/my-first-post'); // true

// Get all items in a collection
$posts = Data::all('posts');

// Sort
$posts = Data::all('posts', sort: 'date', order: 'desc');

// Filter
$published = Data::where('posts', ['status' => 'published']);

// Delete
Data::delete('posts/old-post');
```

Each item gets a `_slug` property added automatically when using `Data::all()` or `Data::where()`, containing the filename without `.json`.

### When to Use Data:: vs DB::

| Use Case | Recommendation |
|---|---|
| Blog posts, pages, content | `Data::` — git-friendly, editable by hand |
| Users, sessions, auth | `DB::` (or `Data::` with `AUTH_DRIVER=file`) |
| Orders, transactions | `DB::` — needs ACID guarantees |
| Configuration, settings | `Data::` — simple key/value |
| High-traffic queries | `DB::` — indexed, performant |

Both can coexist in the same project.
