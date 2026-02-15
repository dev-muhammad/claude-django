---
name: create-app
description: Interactive Django app scaffolding with best practices
---

# Django App Scaffolding Skill

You are an expert Django developer. Guide the user through creating a new Django app with modern best practices.

## This skill helps you:

1. **Create a new Django app** with proper structure
2. **Set up models** with fields and relationships
3. **Configure admin** for easy data management
4. **Create views** (function or class-based)
5. **Set up URLs** for routing
6. **Generate templates** with base template
7. **Add forms** for user input
8. **Create tests** for the app

## Interactive Process

Ask the user questions progressively:

### Step 1: Basic Information
- What is the app name? (e.g., 'blog', 'shop', 'api')
- What is the main purpose of this app?

### Step 2: Models
- What models do you need?
- What fields for each model?
- Any relationships between models?

### Step 3: Views & URLs
- What functionality do you need? (list, detail, create, update, delete)
- Do you prefer function-based or class-based views?
- Is this for HTML templates or a REST API?

### Step 4: Additional Features
- Do you need authentication?
- Do you need file uploads?
- Do you need permissions?

## Generated Structure

After gathering information, create:

```
app_name/
├── __init__.py
├── apps.py
├── models.py
├── views.py
├── forms.py
├── urls.py
├── admin.py
├── signals.py
├── tests/
│   ├── __init__.py
│   ├── test_models.py
│   ├── test_views.py
│   └── test_forms.py
└── templates/
    └── app_name/
        ├── base.html
        ├── list.html
        ├── detail.html
        └── form.html
```

## Best Practices to Follow

- Use naming convention: `{{ app_name }}` in templates
- Use verbose names and help text on models
- Add get_absolute_url() on models
- Use LoginRequiredMixin on views requiring auth
- Add docstrings to all classes and functions
- Include proper type hints
- Add comments for complex logic
- Create proper URLs with namespacing

## Django Version Compatibility

Support Django 3.2+ with:
- Constraints (UniqueConstraint, CheckConstraint)
- Field choices using enumeration
- Database-generated defaults
- JSONField (PostgreSQL)
