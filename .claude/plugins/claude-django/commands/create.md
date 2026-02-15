---
name: django:create
description: Create a new Django project with modular settings and optional integrations
arguments:
  - name: project_name
    description: Name of the project (e.g., 'mysite', 'blog')
    required: true
---

You are an expert Django developer. Create a new Django project with modern best practices.

## Task
Create a new Django project with modular settings structure and optional integrations.

## Project Structure

```
project_name/
├── manage.py
├── project_name/
│   ├── __init__.py
│   ├── asgi.py
│   ├── wsgi.py
│   ├── urls.py
│   └── settings/
│       ├── __init__.py
│       ├── base.py          # Common settings
│       ├── local.py         # Development settings
│       ├── test.py          # Test settings
│       └── production.py    # Production settings
├── apps/
└── .gitignore
```

## Settings Architecture

### base.py (Common Settings)
```python
"""
Base Django settings - shared across all environments.
"""

import os
from pathlib import Path

# Build paths
BASE_DIR = Path(__file__).resolve().parent.parent.parent

# Security
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY', 'dev-secret-key-change-in-production')
DEBUG = False
ALLOWED_HOSTS = []

# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'project_name.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'project_name.wsgi.application'

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME', 'project_name'),
        'USER': os.environ.get('DB_USER', 'postgres'),
        'PASSWORD': os.environ.get('DB_PASSWORD', ''),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}

# Internationalization
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_TZ = True

# Static files
STATIC_URL = 'static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_DIRS = [BASE_DIR / 'static']

# Media files
MEDIA_URL = 'media/'
MEDIA_ROOT = BASE_DIR / 'media'

# Default primary key field type
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# Apps directory
sys.path.insert(0, str(BASE_DIR / 'apps'))
```

### local.py (Development Settings)
```python
"""
Local development settings.
"""

from .base import *

DEBUG = True
ALLOWED_HOSTS = ['localhost', '127.0.0.1', '[::1]']

# Use SQLite for local development
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# Email backend for development (console)
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

# Debug Toolbar (optional)
INSTALLED_APPS += ['debug_toolbar']
MIDDLEWARE += ['debug_toolbar.middleware.DebugToolbarMiddleware']

INTERNAL_IPS = ['127.0.0.1']

# CORS (for local API development)
CORS_ALLOW_ALL_ORIGINS = True
```

### test.py (Test Settings)
```python
"""
Test environment settings.
"""

from .base import *

DEBUG = False

# Use in-memory SQLite for faster tests
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': ':memory:',
    }
}

# Faster password hashing for tests
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.MD5PasswordHasher',
]

# Disable logging during tests
LOGGING = {}
```

### production.py (Production Settings)
```python
"""
Production environment settings.
"""

from .base import *

DEBUG = False
ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# Security settings
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True

# Email (configure for your provider)
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = os.environ.get('EMAIL_HOST')
EMAIL_PORT = int(os.environ.get('EMAIL_PORT', 587))
EMAIL_USE_TLS = True
EMAIL_HOST_USER = os.environ.get('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_HOST_PASSWORD')
DEFAULT_FROM_EMAIL = os.environ.get('DEFAULT_FROM_EMAIL')

# Logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': BASE_DIR / 'logs' / 'django.log',
        },
    },
    'root': {
        'handlers': ['file'],
    },
}

# Static files (use WhiteNoise for production)
INSTALLED_APPS += ['whitenoise.runserver_nostatic']
MIDDLEWARE.insert(1, 'whitenoise.middleware.WhiteNoiseMiddleware')
```

### settings/__init__.py
```python
from .local import *  # Default to local settings
```

## Optional Add-ons

### Authentication
Add to INSTALLED_APPS in base.py:
```python
'django.contrib.auth',
'allauth',  # django-allauth
'allauth.account',
'allauth.socialaccount',
```

Settings:
```python
AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
    'allauth.account.auth_backends.AuthenticationBackend',
]
```

### Django REST Framework
Add to INSTALLED_APPS in base.py:
```python
'rest_framework',
'rest_framework.authtoken',
```

Settings:
```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
}
```

### Default Admin Skin
Options: `django-grappelli`, `django-suit`, `django-admin-interface`, `django-unfold`

Example with django-unfold (modern, clean interface):
```python
INSTALLED_APPS = [
    'unfold',
    'unfold.contrib.filters',  # Optional
    'unfold.contrib.forms',   # Optional
    'django.contrib.admin',
    # ...
]

# Settings
UNFOLD = {
    'site': {
        'title': 'My Site',
        'header': True,
        'color_scheme': 'auto',  # light, dark, auto
    },
    'tabs': [
        ('User', ['auth_user', 'auth_group']),
        ('Content', ['blog_post', 'blog_comment']),
    ],
}

# Unfold requires these patterns in urls.py
from django.urls import path, include
from django.contrib import admin
from unfold.schemas import UnfoldAdminSiteAdapter
from django.contrib.admin.views.decorators import staff_member_required
from django.views.decorators.cache import never_cache

admin_site = admin.__class__.as_view()

urlpatterns = [
    path('admin/', staff_member_required(never_cache(admin_site))),
]
```

Example with django-admin-interface:
```python
INSTALLED_APPS = [
    'admin_interface',
    'colorfield',
    'django.contrib.admin',
    # ...
]

# Settings
ADMIN_INTERFACE = {
    'name': 'My Site',
    'language': 'en-us',
    'theme': 'django',
    'color_scheme': 'auto',
}
```

## Create Command Sequence

```bash
# Create project structure
mkdir -p project_name/{apps,static,media,templates,logs}
django-admin startproject project_name .
django-admin startapp core apps/core

# Create modular settings
mkdir -p project_name/settings
touch project_name/settings/__init__.py project_name/settings/base.py project_name/settings/local.py project_name/settings/test.py project_name/settings/production.py

# Update manage.py to use settings module
# sed or edit manage.py to use --settings flag

# Create .gitignore
cat > .gitignore << 'EOF'
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
.venv/
ENV/
build/
dist/
*.egg-info/
.pytest_cache/
.coverage
htmlcov/
db.sqlite3
db.sqlite3-journal
/staticfiles/
/media/
/logs/
*.log
.env
.DS_Store
EOF

# Create .env.example
cat > .env.example << 'EOF'
DJANGO_SETTINGS_MODULE=project_name.settings.local
DJANGO_SECRET_KEY=your-secret-key-here
DB_NAME=project_name
DB_USER=postgres
DB_PASSWORD=
DB_HOST=localhost
DB_PORT=5432
ALLOWED_HOSTS=localhost,127.0.0.1
EOF
```

## Output

Display:
- Project structure created
- Settings files created with modular architecture
- Next steps for each add-on selected
- Commands to run:
  - `python manage.py migrate`
  - `python manage.py createsuperuser`
  - `python manage.py runserver`

Ask the user for:
1. Project name
2. Include authentication (django-allauth)?
3. Include Django REST Framework?
4. Default admin skin (grappelli, suit, admin-interface, unfold, or none)?
5. Any additional apps to create?
