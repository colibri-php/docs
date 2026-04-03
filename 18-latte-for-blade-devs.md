# Latte for Blade Developers

If you're coming from Laravel's Blade, here's how Latte compares. The concepts are the same — the syntax is slightly different.

## Output

| Blade | Latte |
|---|---|
| `{{ $name }}` | `{$name}` |
| `{!! $html !!}` | `{$html\|noescape}` |
| `{{ $name ?? 'default' }}` | `{$name ?? 'default'}` |

Latte auto-escapes everything by default, just like Blade.

## Conditionals

**Blade:**
```blade
@if($user)
    <p>{{ $user['name'] }}</p>
@elseif($guest)
    <p>Guest</p>
@else
    <p>Anonymous</p>
@endif
```

**Latte:**
```latte
{if $user}
    <p>{$user['name']}</p>
{elseif $guest}
    <p>Guest</p>
{else}
    <p>Anonymous</p>
{/if}
```

## Loops

**Blade:**
```blade
@foreach($posts as $post)
    <p>{{ $post['title'] }}</p>
@endforeach

@forelse($posts as $post)
    <p>{{ $post['title'] }}</p>
@empty
    <p>No posts.</p>
@endforelse
```

**Latte:**
```latte
{foreach $posts as $post}
    <p>{$post['title']}</p>
{/foreach}

{foreach $posts as $post}
    <p>{$post['title']}</p>
{else}
    <p>No posts.</p>
{/foreach}
```

### Loop Variable

**Blade:** `$loop->index`, `$loop->first`, `$loop->last`

**Latte:** `$iterator->counter`, `$iterator->first`, `$iterator->last`

```latte
{foreach $items as $item}
    {$iterator->counter}. {$item}
    {sep}, {/sep}
{/foreach}
```

The `{sep}...{/sep}` tag renders content between items only (no trailing comma).

## Layouts & Blocks

**Blade:**
```blade
{{-- layouts/app.blade.php --}}
<html>
<body>
    @yield('content')
</body>
</html>

{{-- pages/about.blade.php --}}
@extends('layouts.app')

@section('content')
    <h1>About</h1>
@endsection
```

**Latte:**
```latte
{* templates/layouts/default.latte *}
<html>
<body>
    {block content}{/block}
</body>
</html>

{* routes/web/about.latte *}
{page title: 'About'}

{block content}
    <h1>About</h1>
{/block}
```

In Colibri, the layout is applied automatically — you don't need `@extends`. Change it with `{page layout: 'admin'}`.

## Includes

**Blade:**
```blade
@include('partials.header', ['title' => 'Home'])
```

**Latte:**
```latte
{include 'partials/header.latte', title: 'Home'}
```

## Components

Blade has `<x-component>`. Latte doesn't have components in the same way, but you can use `{include}` with parameters or `{block}` / `{define}` for reusable sections:

```latte
{define card, $title, $body}
    <div class="card">
        <h3>{$title}</h3>
        <p>{$body}</p>
    </div>
{/define}

{include card, title: 'Hello', body: 'World'}
{include card, title: 'Another', body: 'Card'}
```

## HTML-Aware Attributes

Latte has a killer feature Blade doesn't: the `n:` attribute syntax.

**Blade:**
```blade
@if($active)
    <a href="/" class="active">Home</a>
@else
    <a href="/">Home</a>
@endif
```

**Latte:**
```latte
<a href="/" n:class="$active ? active">Home</a>
```

More examples:

```latte
<div n:if="$show">Conditional div</div>
<li n:foreach="$items as $item">{$item}</li>
<input n:attr="disabled: $disabled">
```

The `n:` prefix moves the logic into the HTML tag — cleaner, less nesting.

## CSRF

**Blade:**
```blade
<form method="POST">
    @csrf
    ...
</form>
```

**Latte (Colibri):**
```latte
<form method="POST">
    {csrf}
    ...
</form>
```

## Quick Reference

| Concept | Blade | Latte |
|---|---|---|
| Echo | `{{ }}` | `{ }` |
| Raw | `{!! !!}` | `{\|noescape}` |
| If | `@if` / `@endif` | `{if}` / `{/if}` |
| Loop | `@foreach` / `@endforeach` | `{foreach}` / `{/foreach}` |
| Layout | `@extends('layout')` | `{page layout: 'name'}` (auto) |
| Section | `@section` / `@yield` | `{block}` |
| Include | `@include('partial')` | `{include 'partial.latte'}` |
| Escape | auto | auto |
| Attribute | — | `n:if`, `n:foreach`, `n:class` |
| Comments | `{{-- --}}` | `{* *}` |

## Learning More

- [Latte documentation](https://latte.nette.org/en/)
- [Latte tags reference](https://latte.nette.org/en/tags)
- [Latte filters reference](https://latte.nette.org/en/filters)
