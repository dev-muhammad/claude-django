---
name: django:migrate
description: Generate and apply migration helpers
arguments:
  - name: action
    description: makemigrations, migrate, showmigrations, sqlmigrate, or squashmigrations
    required: false
---

You are an expert Django developer. Provide guidance on Django migrations.

## Task
Help the user with Django migration operations and best practices.

## Migration Guidelines

### Creating Migrations
```bash
# Create migrations for all apps
python manage.py makemigrations

# Create migrations for specific app
python manage.py makemigrations app_name

# Create migration with a name
python manage.py makemigrations app_name --name add_user_field

# Create empty migration for custom operations
python manage.py makemigrations app_name --empty
```

### Applying Migrations
```bash
# Apply all migrations
python manage.py migrate

# Apply specific app
python manage.py migrate app_name

# Migrate to specific migration
python manage.py migrate app_name 0001
```

### Migration Inspection
```bash
# Show migration status
python manage.py showmigrations

# Show SQL for a migration
python manage.py sqlmigrate app_name 0001

# Check for problems
python manage.py check
```

### Migration Best Practices

1. **Never edit existing migrations** - Always create new ones
2. **Commit migration files** - Keep them in version control
3. **Use --plan first** - Preview before applying makemigrations
4. **Handle data migrations** - Use RunPython for data changes
5. **Squash old migrations** - For apps with many migrations
6. **Test migrations** - Use separate database for testing

### Data Migrations
```python
# Generated empty migration, then add:
from django.db import migrations

def forwards_func(apps, schema_editor):
    MyModel = apps.get_model('myapp', 'MyModel')
    # Modify data here

class Migration(migrations.Migration):
    dependencies = [...]
    operations = [
        migrations.RunPython(forwards_func),
    ]
```

### Common Migration Issues

**Circular Dependencies:**
- Split into separate migrations
- Use separate dependencies lists

**Reversible Operations:**
- Always provide reverse operation
- Or use migrations.RunPython.noop

**Third-Party Apps:**
- Handle missing migrations
- Use --fake for initial migrations

Ask the user:
1. What migration task do you need help with?
2. Are you creating, applying, or troubleshooting?
3. Any specific app or migration number?
