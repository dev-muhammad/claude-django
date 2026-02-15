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
```django
{% partial "card" %}
<div class="card">
    <h2>{{ title }}</h2>
    <p>{{ content }}</p>
</div>
{% endpartial %}
```

### Rendering a Partial
```django
<!-- Render the partial with context -->
{% partial "card" title="Hello" content="World" %}
```

### Or from a View
```python
from django.views import View
from django.shortcuts import render

class MyView(View):
    def get(self, request):
        return render(request, 'my_template.html', {
            'partial': 'card',
            'partial_context': {'title': 'Hello', 'content': 'World'}
        })
```

## Partial Types

### 1. Component Partials
Self-contained UI components:

**templates/partials/button.html**
```django
{% partial "button" %}
<button class="btn {{ class|default:'btn-primary' }}" {{ attrs|safe }}>
    {% if icon %}{{ icon|safe }}{% endif %}
    {{ label }}
</button>
{% endpartial %}
```

**Usage:**
```django
{% partial "button" label="Click me" class="btn-large" %}
{% partial "button" label="Delete" class="btn-danger" icon="&times;" %}
```

### 2. Section Partials
Page sections (headers, footers, sidebars):

**templates/partials/header.html**
```django
{% partial "header" %}
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
{% endpartial %}
```

### 3. Form Partials
Reusable form fields:

**templates/partials/field.html**
```django
{% partial "field" %}
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
{% endpartial %}
```

**Usage:**
```django
<form method="post">
    {% csrf_token %}
    {% for field in form %}
        {% partial "field" field=field label=field.label %}
    {% endfor %}
    <button type="submit">Submit</button>
</form>
```

## HTMX + Partials

Perfect combination for dynamic UIs:

### Button that loads content
```django
<button hx-get="/items/123/partial/"
        hx-swap="outerHTML"
        hx-target="#item-123">
    Refresh
</button>

<div id="item-123">
    {% partial "item-detail" item=item %}
</div>
```

### View returns partial
```python
def item_detail_partial(request, pk):
    item = get_object_or_404(Item, pk=pk)
    response = render(request, 'partials/item-detail.html', {'item': item})
    response.headers['HX-Trigger'] = 'itemUpdated'
    return response
```

## Partial with Slots

Create flexible partials with slots:

```django
{% partial "modal" %}
<div class="modal {{ class|default:'' }}" id="{{ id|default:'modal' }}">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <h3>{{ title }}</h3>
                <button class="close">&times;</button>
            </div>
            <div class="modal-body">
                {% slot body %}Default body{% endslot %}
            </div>
            {% if footer %}
            <div class="modal-footer">
                {% slot footer %}Default footer{% endslot %}
            </div>
            {% endif %}
        </div>
    </div>
</div>
{% endpartial %}
```

**Usage:**
```django
{% partial "modal" id="confirm-modal" title="Confirm" %}
    {% fill body %}
        <p>Are you sure you want to delete this?</p>
    {% endfill %}
    {% fill footer %}
        <button class="btn-cancel">Cancel</button>
        <button class="btn-danger">Delete</button>
    {% endfill %}
{% endpartial %}
```

## Partial Context Inheritance

Partials inherit parent context:

```django
<!-- parent.html -->
{% partial "child" %}
<!-- Can access parent_var -->
{% endpartial %}
```

### Isolate context when needed
```django
{% partial "child" only %}
<!-- Only has explicitly passed variables -->
{% endpartial %}
```

### Merge contexts
```django
{% partial "child" with foo="bar" %}
<!-- Has parent context plus foo -->
{% endpartial %}
```

## Organizing Partials

```
templates/
├── partials/
│   ├── components/
│   │   ├── button.html
│   │   ├── card.html
│   │   └── modal.html
│   ├── forms/
│   │   ├── field.html
│   │   ├── errors.html
│   │   └── widgets/
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
{% load partials %}
{% partial "blog:post-card" post=post %}
```

## Best Practices

1. **Keep partials small** - Single responsibility
2. **Use clear names** - `components/card.html` not `card.html`
3. **Default values** - Use `|default` for optional args
4. **Document context** - Comment what variables partials expect
5. **Test independently** - Partials should render standalone
6. **Version controlled** - Track partial changes like code

Ask the user:
1. What type of partial do you need? (component, section, form)
2. What context variables does it need?
3. Should it have slots for flexible content?
4. Will it be used with HTMX?
