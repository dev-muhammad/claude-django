---
name: django:csp
description: Configure Content Security Policy using Django 6.0 built-in CSP middleware
arguments:
  - name: csp_level
    description: CSP strictness level (strict, moderate, relaxed)
    required: false
---

# Django 6.0 Content Security Policy (CSP)

You are an expert Django 6.0 security developer. Configure Content Security Policy using Django's built-in CSP middleware.

## What is CSP?

Content Security Policy (CSP) is a security layer that helps prevent Cross-Site Scripting (XSS) attacks by controlling which resources can be loaded.

**Django 6.0** includes **native CSP support** - no more third-party packages needed!

## Basic Configuration

### Installation
```python
# settings/base.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.middleware.csp.CSPMiddleware',  # Django 6.0 built-in
    # ... other middleware ...
]

INSTALLED_APPS = [
    # ...
    'django.csp',  # Django 6.0 built-in
]
```

## CSP Levels

### Level 1: Relaxed (Good for starting)
```python
CSP_DEFAULT_SRC = ("'self'", "'unsafe-inline'", "'unsafe-eval'")
CSP_IMG_SRC = ("'self'", "data:", "https:")
CSP_SCRIPT_SRC = ("'self'", "'unsafe-inline'", "'unsafe-eval'")
CSP_STYLE_SRC = ("'self'", "'unsafe-inline'")
```

### Level 2: Moderate (Recommended)
```python
CSP_DEFAULT_SRC = ("'self'",)
CSP_SCRIPT_SRC = ("'self'",)
CSP_STYLE_SRC = ("'self'", "'unsafe-inline'")  # For inline styles
CSP_IMG_SRC = ("'self'", "data:")
CSP_FONT_SRC = ("'self'",)
CSP_CONNECT_SRC = ("'self'",)
CSP_FRAME_ANCESTORS = ("'none'",)  # Prevent clickjacking
```

### Level 3: Strict (Best security)
```python
CSP_DEFAULT_SRC = ("'self'",)
CSP_SCRIPT_SRC = ("'self'",)
CSP_STYLE_SRC = ("'self'",)
CSP_IMG_SRC = ("'self'", "data:")
CSP_FONT_SRC = ("'self'",)
CSP_CONNECT_SRC = ("'self'",)
CSP_FRAME_ANCESTORS = ("'none'",)
CSP_BASE_URI = ("'none'",)
CSP_FORM_ACTION = ("'self'",)
CSP_FRAME_SRC = ("'none'",)
CSP_OBJECT_SRC = ("'none'",)
```

## Common CSP Directives

```python
# Scripts - critical for XSS prevention
CSP_SCRIPT_SRC = ("'self'", "https://cdn.example.com")

# Styles - allow inline with nonce
CSP_STYLE_SRC = ("'self'", "'nonce-%s'")

# Images - data URLs are common
CSP_IMG_SRC = ("'self'", "data:", "https:")

# Fonts
CSP_FONT_SRC = ("'self'", "https://fonts.gstatic.com")

# Connections (AJAX, fetch)
CSP_CONNECT_SRC = ("'self'", "https://api.example.com")

# Frames - prevent framing unless needed
CSP_FRAME_SRC = ("'none'",)

# Prevent clickjacking
CSP_FRAME_ANCESTORS = ("'none'",)
```

## Using Nonces for Inline Scripts

When you need inline scripts, use nonces:

```python
# settings.py
CSP_SCRIPT_SRC = ("'self'", "'nonce-%s'")

# In views
from django.views.decorators.csp import csp_update

@csp_update SCRIPT_SRC="'self' 'nonce-abc123'"
def my_view(request):
    # Inline script with nonce
    return render(request, 'template.html', {'csp_nonce': 'abc123'})
```

```django
<!-- template.html -->
<script nonce="{{ csp_nonce }}">
    // Your inline script
</script>
```

## CSP for Different Environments

```python
# settings/base.py
CSP_DEFAULT_SRC = ("'self'",)
CSP_REPORT_ONLY = False  # Block violations (True = report only)

# settings/development.py
CSP_REPORT_ONLY = True  # Don't block, just report
CSP_REPORT_URI = '/api/csp-report/'

# settings/production.py
CSP_REPORT_ONLY = False  # Actually block violations
CSP_REPORT_URI = 'https://csp-report.example.com/'
```

## CSP Report Endpoint

Create an endpoint to receive CSP violation reports:

```python
# views.py
import json
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def csp_report(request):
    if request.method == 'POST':
        report = json.loads(request.body)
        # Log the violation
        logger.warning(f'CSP Violation: {report}')
    return HttpResponse(status=204)

# urls.py
path('api/csp-report/', csp_report, name='csp_report')
```

## Handling Third-Party Services

### Google Analytics
```python
CSP_SCRIPT_SRC = ("'self'", "https://www.googletagmanager.com", "https://www.google-analytics.com")
CSP_IMG_SRC = ("'self'", "https://www.google-analytics.com")
CSP_CONNECT_SRC = ("'self'", "https://www.google-analytics.com")
```

### Stripe
```python
CSP_SCRIPT_SRC = ("'self'", "https://js.stripe.com")
CSP_FRAME_SRC = ("'self'", "https://js.stripe.com", "https://hooks.stripe.com")
```

### Cloudflare
```python
CSP_SCRIPT_SRC = ("'self'", "https://ajax.cloudflare.com")
CSP_IMG_SRC = ("'self'", "https://cdnjs.cloudflare.com")
```

## HTMX + CSP

When using HTMX with CSP:

```python
CSP_SCRIPT_SRC = ("'self'", "'unsafe-eval'")  # HTMX needs eval in some cases
CSP_HX_REQUEST = ("'self'",)
CSP_HX_ELE = ("'self'",)
```

Or use nonce approach:

```django
<script src="https://unpkg.com/htmx.org@1.9.10" nonce="{{ csp_nonce }}"></script>
```

## Testing CSP

1. **Start with report-only mode** - See what would break
2. **Check browser console** - CSP violations appear there
3. **Use CSP Evaluator** - Chrome extension to test policies
4. **Test common user flows** - Login, forms, file uploads

## Migration Strategy

```python
# Step 1: Report only (2 weeks)
CSP_REPORT_ONLY = True

# Step 2: Moderate enforcement (1 month)
CSP_REPORT_ONLY = False
CSP_SCRIPT_SRC = ("'self'", "'unsafe-inline'")

# Step 3: Remove unsafe-inline (use nonces)

# Step 4: Full strict mode
# No unsafe-*, minimal exceptions
```

Ask the user:
1. What CSP level do you need? (relaxed, moderate, strict)
2. Do you use any third-party services? (analytics, payment, CDNs)
3. Do you use HTMX or other JavaScript frameworks?
4. Any existing CSP violations to address?
