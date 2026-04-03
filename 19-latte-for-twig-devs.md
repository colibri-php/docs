# Latte for Twig Developers

If you're coming from Symfony's Twig, here's how Latte compares. They're actually very similar — both use `{` `}` delimiters.

## Output

| Twig | Latte |
|---|---|
| `{{ name }}` | `{$name}` |
| `{{ name\|raw }}` | `{$name\|noescape}` |
| `{{ name\|default('fallback') }}` | `{$name ?? 'fallback'}` |

Key difference: Latte uses PHP's `$` prefix for variables. Twig doesn't.

## Conditionals

**Twig:**
```twig
{% if user %}
    <p>{{ user.name }}</p>
{% elseif guest %}
    <p>Guest</p>
{% else %}
    <p>Anonymous</p>
{% endif %}
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

Key difference: Latte uses PHP arrays (`$user['name']`) instead of dot notation (`user.name`).

## Loops

**Twig:**
```twig
{% for post in posts %}
    <p>{{ post.title }}</p>
{% else %}
    <p>No posts.</p>
{% endfor %}
```

**Latte:**
```latte
{foreach $posts as $post}
    <p>{$post['title']}</p>
{else}
    <p>No posts.</p>
{/foreach}
```

### Loop Variable

**Twig:** `loop.index`, `loop.first`, `loop.last`

**Latte:** `$iterator->counter`, `$iterator->first`, `$iterator->last`

```latte
{foreach $items as $item}
    {$iterator->counter}. {$item}
    {sep}, {/sep}
{/foreach}
```

The `{sep}...{/sep}` tag renders content between items only — like Twig's `loop.last` check but cleaner.

## Filters

**Twig:**
```twig
{{ name|upper }}
{{ name|lower }}
{{ text|length }}
{{ date|date('Y-m-d') }}
```

**Latte:**
```latte
{$name|upper}
{$name|lower}
{$text|length}
{$date|date:'Y-m-d'}
```

Same concept, slightly different syntax: Latte uses `:` instead of `()` for filter arguments.

## Layouts & Blocks

**Twig:**
```twig
{# layouts/base.html.twig #}
<html>
<body>
    {% block content %}{% endblock %}
</body>
</html>

{# pages/about.html.twig #}
{% extends 'layouts/base.html.twig' %}

{% block content %}
    <h1>About</h1>
{% endblock %}
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

In Colibri, the layout is applied automatically — you don't need `{% extends %}`. Change it with `{page layout: 'admin'}`.

## Includes

**Twig:**
```twig
{% include 'partials/header.html.twig' with {'title': 'Home'} %}
```

**Latte:**
```latte
{include 'partials/header.latte', title: 'Home'}
```

## Macros vs Defines

**Twig:**
```twig
{% macro card(title, body) %}
    <div class="card">
        <h3>{{ title }}</h3>
        <p>{{ body }}</p>
    </div>
{% endmacro %}

{{ _self.card('Hello', 'World') }}
```

**Latte:**
```latte
{define card, $title, $body}
    <div class="card">
        <h3>{$title}</h3>
        <p>{$body}</p>
    </div>
{/define}

{include card, title: 'Hello', body: 'World'}
```

## HTML-Aware Attributes

This is where Latte shines — Twig doesn't have an equivalent:

```latte
<div n:if="$show">Conditional div</div>
<li n:foreach="$items as $item">{$item}</li>
<a href="/" n:class="$active ? active">Home</a>
<input n:attr="disabled: $disabled">
```

The `n:` prefix moves logic into HTML attributes — less nesting, cleaner templates.

## Expressions

**Twig** has its own expression language. **Latte** uses PHP directly:

```latte
{if count($items) > 0 && $user !== null}
    {$price * 1.15|number:2}
{/if}
```

This means any PHP function works in Latte without registering it first.

## Quick Reference

| Concept | Twig | Latte |
|---|---|---|
| Echo | `{{ }}` | `{ }` |
| Raw | `{{ \|raw }}` | `{\|noescape}` |
| Variables | `name` | `$name` |
| Properties | `user.name` | `$user['name']` |
| If | `{% if %}` | `{if}` |
| Loop | `{% for x in y %}` | `{foreach $y as $x}` |
| Layout | `{% extends %}` | `{page layout: '...'}` (auto) |
| Block | `{% block %}` | `{block}` |
| Include | `{% include %}` | `{include}` |
| Macro | `{% macro %}` | `{define}` |
| Filter args | `\|date('Y')` | `\|date:'Y'` |
| Attribute | — | `n:if`, `n:foreach`, `n:class` |
| Comments | `{# #}` | `{* *}` |
| Expressions | Twig language | PHP |

## Learning More

- [Latte documentation](https://latte.nette.org/en/)
- [Latte tags reference](https://latte.nette.org/en/tags)
- [Latte filters reference](https://latte.nette.org/en/filters)
