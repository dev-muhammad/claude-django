---
name: setup-drf
description: Configure Django REST Framework for your project
---

# Django REST Framework Setup Skill

You are an expert Django REST Framework developer. Guide the user through setting up DRF with best practices.

## This skill helps you:

1. **Install and configure DRF** in an existing Django project
2. **Set up authentication** (JWT, Session, Token)
3. **Configure permissions** properly
4. **Create serializers** for models
5. **Set up ViewSets** and routers
6. **Add pagination** for list views
7. **Configure filtering** and searching
8. **Set up versioning** for your API
9. **Add documentation** (drf-spectacular or drf-yasg)
10. **Add rate limiting** for API protection

## Interactive Process

Ask the user questions progressively:

### Step 1: Basic Configuration
- Is this a new project or existing project?
- What Django version are you using?
- What DRF version? (latest stable or specific)

### Step 2: Authentication
- What authentication method?
  - Session authentication (same site)
  - Token authentication (simple)
  - JWT authentication (recommended for SPAs/mobile)
  - OAuth2 (for third-party access)

### Step 3: Features Needed
- Do you need filtering? (django-filter)
- Do you need search functionality?
- Do you need ordering?
- Do you need throttling/rate limiting?
- Do you need API versioning?
- Do you need API documentation?

### Step 4: Additional Packages
- drf-spectacular (OpenAPI 3.0, recommended)
- drf-yasg (Swagger UI alternative)
- django-filter (advanced filtering)
- django-rest-framework-simplejwt (JWT)
- django-rest-auth (auth endpoints)

## Installation Steps

```bash
# Install DRF
pip install djangorestframework

# Optional but recommended packages
pip install django-filter
pip install djangorestframework-simplejwt
pip install drf-spectacular
```

## Settings Configuration

```python
# settings/base.py
INSTALLED_APPS = [
    # Django apps...
    'rest_framework',
    'rest_framework.authtoken',  # If using token auth
    'django_filters',              # If using django-filter
    'drf_spectacular',             # If using drf-spectacular
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
        # 'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
        'rest_framework.filters.SearchFilter',
        'rest_framework.filters.OrderingFilter',
    ],
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}

# JWT Settings (if using simplejwt)
from datetime import timedelta

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
}

# Spectacular Settings
SPECTACULAR_SETTINGS = {
    'TITLE': 'Your API',
    'DESCRIPTION': 'Your API description',
    'VERSION': '1.0.0',
    'SERVE_INCLUDE_SCHEMA': False,
}
```

## URL Configuration

```python
from django.urls import path, include
from rest_framework import permissions
from drf_spectacular.views import SpectacularAPIView, SpectacularRedocView, SpectacularSwaggerView

urlpatterns = [
    # API endpoints
    path('api/v1/', include('your_app.urls')),

    # API authentication (if using token auth)
    path('api-token-auth/', rest_framework.authtoken.views.obtain_auth_token),

    # JWT token obtain/refresh (if using simplejwt)
    path('api/token/', rest_framework_simplejwt.views.TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', rest_framework_simplejwt.views.TokenRefreshView.as_view(), name='token_refresh'),

    # API Documentation
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    path('api/schema/swagger-ui/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
    path('api/schema/redoc/', SpectacularRedocView.as_view(url_name='schema'), name='redoc'),
]
```

## API Structure

```
api/
├── v1/
│   ├── __init__.py
│   ├── serializers.py      # All serializers
│   ├── views.py            # API views/viewsets
│   ├── urls.py             # URL patterns with router
│   ├── filters.py          # Custom filter sets
│   ├── permissions.py      # Custom permissions
│   └── pagination.py       # Custom pagination classes
```

## Best Practices

1. **Use ViewSets** for standard CRUD operations
2. **Use APIView** for custom non-model endpoints
3. **Use @action decorator** for custom ViewSet actions
4. **Separate concerns** - views, serializers, filters, permissions
5. **Always use pagination** for list views
6. **Validate input** with serializer validation
7. **Use appropriate HTTP methods** (GET, POST, PATCH, DELETE)
8. **Return proper status codes** (200, 201, 204, 400, 401, 403, 404)
9. **Add API versioning** from the start (/api/v1/, /api/v2/)
10. **Document your API** with drf-spectacular
