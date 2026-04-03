# Authentication

Colibri provides session-based authentication with support for DB and file drivers.

## Configuration

```env
AUTH_DRIVER=db    # db or file
```

```php
// config/auth.php
return [
    'datasource' => env('AUTH_DRIVER', 'db'),
    'roles' => ['user', 'admin'],
    'login_path' => '/login',
];
```

## API

```php
use Colibri\Auth\Auth;

// Register a new user
$user = Auth::register([
    'email' => 'alice@example.com',
    'password' => 'secret123',  // automatically hashed with bcrypt
    'name' => 'Alice',
]);

// Login
if (Auth::attempt('alice@example.com', 'secret123')) {
    // Authenticated — session started, ID regenerated
}

// Get current user
$user = Auth::user();   // array or null

// Check if logged in
Auth::check();          // bool

// Check role
Auth::is('admin');      // bool

// Logout
Auth::logout();         // session cleared, ID regenerated
```

## Roles

The default role for new users is `'user'`. Register with a custom role:

```php
Auth::register([
    'email' => 'admin@example.com',
    'password' => 'secret',
    'name' => 'Admin',
    'role' => 'admin',
]);
```

Check roles in your routes:

```php
if (! Auth::is('admin')) {
    return Colibri\Http\Response::redirect('/');
}
```

Or use the `auth` middleware (see [Middleware](06-middleware.md)).

## Drivers

### DB Driver (default)

Users are stored in the `users` table. Run `php colibri migrate` to create it.

The table has columns: `id`, `email`, `password`, `name`, `role`, `created_at`.

### File Driver

Users are stored as JSON files in `data/users/`. No database required.

```env
AUTH_DRIVER=file
```

Each user is a JSON file with a random ID as filename:

```
data/users/
├── a1b2c3d4e5f6g7h8.json
└── i9j0k1l2m3n4o5p6.json
```

### Custom Driver

Create a class implementing `AuthDriverInterface`:

```php
use Colibri\Auth\Interfaces\AuthDriverInterface;

class MyAuthDriver implements AuthDriverInterface
{
    public function create(array $data): ?array { /* ... */ }
    public function findByEmail(string $email): ?array { /* ... */ }
    public function findById(int|string $id): ?array { /* ... */ }
}
```

Register it:

```php
Auth::registerDriver('custom', MyAuthDriver::class);
```

## Protecting Pages

### In PHP Routes

```php
// routes/web/dashboard.php
if (! Auth::check()) {
    return Colibri\Http\Response::redirect('/login');
}
```

### With Middleware

Create `routes/web/admin/_middleware.php`:

```php
<?php
return ['auth:admin'];
```

All routes in `routes/web/admin/` now require an authenticated user with the `admin` role. See [Middleware](06-middleware.md).

## Login Flow Example

```php
// routes/web/login.php
use Colibri\Auth\Auth;
use Colibri\Http\Response;

return [
    'GET' => function ($request, $params) {
        return view('login');
    },
    'POST' => function ($request, $params) {
        $errors = $request->validate([
            'email' => ['required', 'email'],
            'password' => ['required'],
        ]);

        if ($errors) {
            return Response::back()->with('error', 'Invalid credentials.')->withInput();
        }

        if (! Auth::attempt($request->input('email'), $request->input('password'))) {
            return Response::back()->with('error', 'Invalid credentials.')->withInput();
        }

        $intended = $_SESSION['_intended_url'] ?? '/';
        unset($_SESSION['_intended_url']);

        return Response::redirect($intended);
    },
];
```

The auth middleware stores the intended URL in `$_SESSION['_intended_url']` when redirecting to login, enabling redirect-back after authentication.
