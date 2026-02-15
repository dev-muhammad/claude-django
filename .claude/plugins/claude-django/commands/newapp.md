---
name: django:newapp
description: Create a new Django app and add it to INSTALLED_APPS
arguments:
  - name: app_name
    description: Name of the app (e.g., 'blog', 'shop')
    required: true
  - name: add_to_installed
    description: Add to INSTALLED_APPS automatically (default: true)
    required: false
---

You are an expert Django developer. Create a new Django app using Django's startapp command.

## Task
Create a new Django app and optionally add it to INSTALLED_APPS.

## App Creation

### Basic Command
```bash
python manage.py startapp app_name
```

This creates:
```
app_name/
├── __init__.py
├── admin.py
├── apps.py
├── migrations/
│   └── __init__.py
├── models.py
├── tests.py
├── views.py
```

### With Templates Directory
```bash
python manage.py startapp app_name
mkdir -p app_name/templates/app_name
touch app_name/templates/app_name/base.html
```

## Adding to INSTALLED_APPS

Find the settings file and add the app:
```python
# settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    # ... other apps
    'app_name',  # Add this line
]
```

Or using the AppConfig class:
```python
INSTALLED_APPS = [
    # ...
    'app_name.apps.AppnameConfig',
]
```

## Best Practices

- App names should be lowercase with underscores
- Avoid Django built-in names (admin, auth, contenttypes)
- Add the app to INSTALLED_APPS immediately after creation
- Update URLs to include the new app's URLs

## Post-creation Tasks

After creating the app:
1. Add to INSTALLED_APPS in settings.py
2. Create migrations: `python manage.py makemigrations app_name`
3. Apply migrations: `python manage.py migrate`
4. Create URL patterns in app urls.py
5. Include app URLs in project urls.py

## Example Workflow

```bash
# Create the app
python manage.py startapp blog

# Add to INSTALLED_APPS (edit settings.py)

# Create initial migration
python manage.py makemigrations blog

# Apply migrations
python manage.py migrate
```

## Output

Display:
- Confirmation that app was created
- File structure of the new app
- Reminder to add to INSTALLED_APPS
- Next steps (makemigrations, migrate, URLs)

Ask the user for:
1. App name
2. Whether to add to INSTALLED_APPS automatically
3. Whether to create a templates directory
