---
name: django:partial
description: Create template partials for modular templates (Django 6.0)
arguments:
  - name: partial_type
    description: Type of partial (component, section, layout)
    required: false
---

# Django 6.0 Template Partials

You are an expert Django 6.0 template developer. Create modular, reusable template partials.

## What are Template Partials?

Django 6.0 introduced **template partials** - small, named template fragments that can be rendered independently. Perfect for:
- HTMX partial updates
- Component-based architecture
- Reusable UI elements
- Dynamic content loading

## Basic Syntax

### Defining a Partial
Use `{% partialdef %}` to define a named fragment within a template:

```django
{% partialdef "card" %}
<div class="card">
    <h2>{{ title }}</h2>
    <p>{{ content }}</p>
</div>
{% endpartialdef %}
```

### Rendering a Partial Inline
Use `{% partial %}` to render a defined partial in the same template:

```django
{% partialdef "card" %}
<div class="card">
    <h2>{{ title }}</h2>
    <p>{{ content }}</p>
</div>
{% endpartialdef %}

<!-- Render the partial here -->
{% partial "card" %}
```

### Rendering from a View
Reference a partial using the `template_name#partial_name` syntax:

```python
from django.shortcuts import render

def card_fragment(request):
    return render(request, 'my_template.html#card', {
        'title': 'Hello',
        'content': 'World',
    })
```

### Using with include
```django
{% include "my_template.html#card" %}
```

### Using with get_template
```python
from django.template.loader import get_template

template = get_template('my_template.html#card')
html = template.render({'title': 'Hello', 'content': 'World'})
```

## Partial Types

### 1. Component Partials
Self-contained UI components:

**templates/components/buttons.html**
```django
{% partialdef "primary" %}
<button class="btn btn-primary" {{ attrs|safe }}>
    {% if icon %}{{ icon|safe }}{% endif %}
    {{ label }}
</button>
{% endpartialdef %}

{% partialdef "danger" %}
<button class="btn btn-danger" {{ attrs|safe }}>
    {{ label }}
</button>
{% endpartialdef %}
```

**Usage:**
```django
{% include "components/buttons.html#primary" with label="Click me" %}
{% include "components/buttons.html#danger" with label="Delete" %}
```

### 2. Section Partials
Page sections (headers, footers, sidebars):

**templates/partials/header.html**
```django
{% partialdef "header" %}
<header class="header">
    <div class="container">
        <h1>{{ site_name }}</h1>
        <nav>
            {% for link in nav_links %}
            <a href="{{ link.url }}">{{ link.label }}</a>
            {% endfor %}
        </nav>
    </div>
</header>
{% endpartialdef %}
```

### 3. Form Partials
Reusable form fields:

**templates/partials/forms.html**
```django
{% partialdef "field" %}
<div class="form-group {{ class|default:'' }}">
    <label for="{{ field.id_for_label }}">{{ label }}</label>
    {{ field }}
    {% if field.help_text %}
    <small class="help-text">{{ field.help_text }}</small>
    {% endif %}
    {% if field.errors %}
    <div class="errors">{{ field.errors }}</div>
    {% endif %}
</div>
{% endpartialdef %}
```

**Usage:**
```django
<form method="post">
    {% csrf_token %}
    {% for field in form %}
        {% include "partials/forms.html#field" with field=field label=field.label %}
    {% endfor %}
    <button type="submit">Submit</button>
</form>
```

## HTMX + Partials

Perfect combination for dynamic UIs. Define the partial inline and render only the fragment on HTMX requests:

### Template with inline partial
```django
<!-- items/list.html -->
{% partialdef "item-row" %}
<tr id="item-{{ item.pk }}">
    <td>{{ item.name }}</td>
    <td>{{ item.price }}</td>
    <td>
        <button hx-get="{% url 'item-detail' item.pk %}"
                hx-target="#item-{{ item.pk }}"
                hx-swap="outerHTML">
            Refresh
        </button>
    </td>
</tr>
{% endpartialdef %}

<table>
    {% for item in items %}
        {% partial "item-row" %}
    {% endfor %}
</table>
```

### View returns only the partial fragment
```python
def item_detail(request, pk):
    item = get_object_or_404(Item, pk=pk)

    # Return only the partial for HTMX requests
    if request.headers.get('HX-Request'):
        return render(request, 'items/list.html#item-row', {'item': item})

    # Return the full page otherwise
    return render(request, 'items/list.html', {'items': Item.objects.all()})
```

## Inline Rendering Control

By default, `{% partialdef %}` renders the content where it's defined AND makes it available as a named partial. Use `inline=False` to define without rendering:

```django
{# Define but don't render here #}
{% partialdef "modal" inline=False %}
<div class="modal" id="{{ id|default:'modal' }}">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <h3>{{ title }}</h3>
                <button class="close">&times;</button>
            </div>
            <div class="modal-body">
                {{ body }}
            </div>
        </div>
    </div>
</div>
{% endpartialdef %}

{# Render it later with specific context #}
{% partial "modal" with id="confirm-modal" title="Confirm Delete" body="Are you sure?" %}
```

## Organizing Partials

```
templates/
├── partials/
│   ├── components/
│   │   ├── buttons.html      # Multiple partialdef per file
│   │   ├── cards.html
│   │   └── modals.html
│   ├── forms/
│   │   ├── fields.html
│   │   └── errors.html
│   ├── layout/
│   │   ├── header.html
│   │   ├── footer.html
│   │   └── sidebar.html
│   └── tables/
│       ├── table.html
│       └── pagination.html
```

## Loading Partials from Apps

Create reusable partials in apps:

```python
# settings.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        # Automatically finds templates/ in each app
    }
]
```

Use in templates:
```django
{% include "blog/partials.html#post-card" with post=post %}
```

Or from views:
```python
return render(request, 'blog/partials.html#post-card', {'post': post})
```

## Best Practices

1. **Keep partials small** - Single responsibility
2. **Use clear names** - `{% partialdef "user-card" %}` not `{% partialdef "c1" %}`
3. **Default values** - Use `|default` for optional context variables
4. **Document context** - Comment what variables partials expect
5. **Use `inline=False`** - When defining reusable fragments that shouldn't render in place
6. **Use `#` syntax** - `template.html#partial` for HTMX and fragment rendering

Ask the user:
1. What type of partial do you need? (component, section, form)
2. What context variables does it need?
3. Will it be used with HTMX?
4. Should the partial render inline or be defined for later use?
