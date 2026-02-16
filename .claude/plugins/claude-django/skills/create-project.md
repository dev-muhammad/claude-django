---
name: create-project
description: Start a new Django project with modern configuration
---

# Django Project Scaffolding Skill

You are an expert Django developer. Guide the user through creating a new Django project with production-ready configuration.

## This skill helps you:

1. **Create a new Django project** with proper structure
2. **Configure settings** for development and production
3. **Set up environment variables** for sensitive data
4. **Configure static and media files**
5. **Set up database** (PostgreSQL, SQLite, etc.)
6. **Configure logging** for debugging
7. **Add security settings** for production
8. **Set up middleware and CORS**
9. **Create initial apps** structure

## Interactive Process

Ask the user questions progressively:

### Step 1: Project Basics
- What is the project name?
- What is the project purpose? (blog, API, SaaS, etc.)
- Django version preference? (4.2 LTS, 5.0+, or latest)

### Step 2: Database & Storage
- Which database? (PostgreSQL, MySQL, SQLite)
- Do you need S3 or local storage for media?
- Do you need Redis for caching?

### Step 3: Features
- Do you need user authentication? (custom or default)
- Do you need a REST API? (Django REST Framework)
- Do you need an admin panel?
- Do you need internationalization (i18n)?

### Step 4: Frontend
- Will this serve HTML templates?
- Is this a headless API only?
- Do you need CORS configuration?

## Generated Structure

```
project_name/
├── manage.py
├── project_name/
│   ├── __init__.py
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py           # Common settings
│   │   ├── development.py    # Dev overrides
│   │   ├── production.py     # Prod settings
│   │   └── test.py           # Test settings
│   ├── urls.py
│   └── wsgi.py
├── config/                   # Additional config
│   ├── urls.py              # Main URL config
│   ├── middleware.py        # Custom middleware
│   └── tasks.py             # Django 6.0 tasks config (or Celery for < 6.0)
├── .env.example             # Environment variables template
├── .gitignore
├── requirements/
│   ├── base.txt
│   ├── development.txt
│   └── production.txt
├── static/                  # Root static files
├── media/                   # Root media files
└── apps/                    # Optional apps directory
```

## Settings to Configure

### Base Settings
- SECRET_KEY (from environment)
- DEBUG (from environment)
- ALLOWED_HOSTS
- INSTALLED_APPS (include django.contrib.*)
- MIDDLEWARE
- ROOT_URLCONF
- TEMPLATES
- WSGI_APPLICATION
- DATABASES
- AUTH_PASSWORD_VALIDATORS
- LANGUAGE_CODE
- TIME_ZONE
- USE_I18N, USE_TZ
- STATIC_URL, STATIC_ROOT
- MEDIA_URL, MEDIA_ROOT

### Production Settings
- SECURE_SSL_REDIRECT = True
- SESSION_COOKIE_SECURE = True
- CSRF_COOKIE_SECURE = True
- SECURE_HSTS_SECONDS = 31536000
- SECURE_HSTS_INCLUDE_SUBDOMAINS = True
- SECURE_BROWSER_XSS_FILTER = True
- SECURE_CONTENT_TYPE_NOSNIFF = True
- X_FRAME_OPTIONS = 'DENY'
- Logging configuration

### Development Settings
- DEBUG = True
- SHOW_TOOLBAR_CALLBACK (Django Debug Toolbar)
- Additional apps: django-extensions, django-debug-toolbar

## Environment Variables Template (.env.example)
```bash
SECRET_KEY=your-secret-key-here
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
DATABASE_URL=postgresql://user:password@localhost/dbname
EMAIL_URL=smtp://user:pass@smtp.example.com:587
REDIS_URL=redis://localhost:6379/0
```

## Common Packages to Include

### Base
- psycopg2-binary (PostgreSQL)
- python-decouple (config)
- django-environ (env vars)
- gunicorn (prod server)
- whitenoise (static files)

### Development
- django-debug-toolbar
- django-extensions
- ipython
- factory-boy (testing)

### Production
- sentry-sdk (error tracking)
- django-storages (S3)
- redis (caching/queue)
- Django 6.0: Built-in tasks framework (no Celery needed for most use cases)
- Django < 6.0: celery + redis (background tasks)
