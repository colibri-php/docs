# Testing

Colibri uses [Pest](https://pestphp.com) for testing.

## Running Tests

```bash
composer test                        # Run all tests
composer test -- --filter=AuthTest   # Run specific test file
composer lint                        # Check formatting (Pint)
composer analyse                     # Static analysis (PHPStan)
composer check                       # All three: test + lint + analyse
```

## Project Structure

```
tests/                     # Your project tests
core/tests/                # Framework tests (in the framework package)
```

Project tests go in `tests/`. Framework tests live in the framework package and are not typically modified.

## Writing Tests

```php
<?php
// tests/HomepageTest.php

test('homepage returns 200', function () {
    // Your test here
});
```

## Testing with Database

Use SQLite in-memory for fast, isolated tests:

```php
use Colibri\App;
use Colibri\Config;
use Colibri\Database\DB;
use Medoo\Medoo;

beforeEach(function () {
    $appRef = new ReflectionClass(App::class);
    $appRef->setStaticPropertyValue('instance', null);

    $configRef = new ReflectionClass(Config::class);
    $configRef->setStaticPropertyValue('instance', null);

    $dbRef = new ReflectionClass(DB::class);
    $dbRef->setStaticPropertyValue('medoo', null);

    $_ENV['DB_PATH'] = ':memory:';
    App::boot(dirname(__DIR__));

    DB::initWith(new Medoo([
        'type' => 'sqlite',
        'database' => ':memory:',
    ]));

    // Create tables
    DB::exec('CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, email TEXT)');
});

test('can insert and query users', function () {
    DB::insert('users', ['name' => 'Alice', 'email' => 'alice@test.com']);

    $user = DB::get('users', '*', ['name' => 'Alice']);

    expect($user['email'])->toBe('alice@test.com');
});
```

## Testing Without Database

For file-based auth and cache:

```php
beforeEach(function () {
    // Reset singletons...
    $_ENV['AUTH_DRIVER'] = 'file';
    App::boot(dirname(__DIR__));
});

test('can register with file driver', function () {
    $user = Auth::register([
        'email' => 'alice@test.com',
        'password' => 'secret',
        'name' => 'Alice',
    ]);

    expect($user['email'])->toBe('alice@test.com');
});
```

## Dev Tools

| Tool | Command | Purpose |
|---|---|---|
| [Pest](https://pestphp.com) | `composer test` | Testing |
| [PHPStan](https://phpstan.org) | `composer analyse` | Static analysis (level 6) |
| [Pint](https://laravel.com/docs/pint) | `composer format` | Code formatting (PER preset) |
| [VarDumper](https://symfony.com/doc/current/components/var_dumper.html) | `dump()` / `dd()` | Debug output |
| [Whoops](https://github.com/filp/whoops) | Automatic in debug mode | Pretty error pages |
