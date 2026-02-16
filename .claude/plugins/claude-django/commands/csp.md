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
from django.utils.csp import CSP

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.middleware.csp.ContentSecurityPolicyMiddleware',  # Django 6.0 built-in
    # ... other middleware ...
]

TEMPLATES = [
    {
        # ...
        'OPTIONS': {
            'context_processors': [
                # ... existing processors ...
                'django.template.context_processors.csp',  # For nonce support
            ],
        },
    },
]
```

## CSP Levels

### Level 1: Relaxed (Good for starting)
```python
from django.utils.csp import CSP

SECURE_CSP = {
    "default-src": [CSP.SELF, CSP.UNSAFE_INLINE, CSP.UNSAFE_EVAL],
    "img-src": [CSP.SELF, "data:", "https:"],
    "script-src": [CSP.SELF, CSP.UNSAFE_INLINE, CSP.UNSAFE_EVAL],
    "style-src": [CSP.SELF, CSP.UNSAFE_INLINE],
}
```

### Level 2: Moderate (Recommended)
```python
from django.utils.csp import CSP

SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF],
    "style-src": [CSP.SELF, CSP.UNSAFE_INLINE],
    "img-src": [CSP.SELF, "data:"],
    "font-src": [CSP.SELF],
    "connect-src": [CSP.SELF],
    "frame-ancestors": [CSP.NONE],
}
```

### Level 3: Strict (Best security)
```python
from django.utils.csp import CSP

SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF],
    "style-src": [CSP.SELF],
    "img-src": [CSP.SELF, "data:"],
    "font-src": [CSP.SELF],
    "connect-src": [CSP.SELF],
    "frame-ancestors": [CSP.NONE],
    "base-uri": [CSP.NONE],
    "form-action": [CSP.SELF],
    "frame-src": [CSP.NONE],
    "object-src": [CSP.NONE],
}
```

## Common CSP Directives

```python
from django.utils.csp import CSP

SECURE_CSP = {
    # Scripts - critical for XSS prevention
    "script-src": [CSP.SELF, "https://cdn.example.com"],

    # Styles
    "style-src": [CSP.SELF, CSP.UNSAFE_INLINE],

    # Images - data URLs are common
    "img-src": [CSP.SELF, "data:", "https:"],

    # Fonts
    "font-src": [CSP.SELF, "https://fonts.gstatic.com"],

    # Connections (AJAX, fetch)
    "connect-src": [CSP.SELF, "https://api.example.com"],

    # Frames - prevent framing unless needed
    "frame-src": [CSP.NONE],

    # Prevent clickjacking
    "frame-ancestors": [CSP.NONE],
}
```

## Using Nonces for Inline Scripts

Django 6.0 has built-in nonce support via the `CSP.NONCE` constant and `csp` context processor:

```python
# settings.py
from django.utils.csp import CSP

SECURE_CSP = {
    "script-src": [CSP.SELF, CSP.NONCE],
    "style-src": [CSP.SELF, CSP.NONCE],
}
```

The `django.template.context_processors.csp` context processor provides the nonce automatically:

```django
<!-- template.html -->
<script nonce="{{ csp_nonce }}">
    // Your inline script — nonce is auto-generated per request
</script>
```

**Result header:** `script-src 'self' 'nonce-SECRET'; style-src 'self' 'nonce-SECRET'`

## CSP for Different Environments

```python
from django.utils.csp import CSP

# settings/base.py — Define the policy
SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE],
    "style-src": [CSP.SELF, CSP.NONCE],
    "img-src": [CSP.SELF, "data:"],
}

# settings/development.py — Report-only (don't block, just report)
SECURE_CSP_REPORT_ONLY = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.UNSAFE_INLINE],
    "report-uri": "/api/csp-report/",
}

# settings/production.py — Enforce (block violations)
# Uses SECURE_CSP from base.py (enforcing mode)
```

## CSP Report Endpoint

Create an endpoint to receive CSP violation reports:

```python
# views.py
import json
import logging
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_exempt

logger = logging.getLogger(__name__)

@csrf_exempt
def csp_report(request):
    if request.method == 'POST':
        report = json.loads(request.body)
        logger.warning(f'CSP Violation: {report}')
    return HttpResponse(status=204)

# urls.py
path('api/csp-report/', csp_report, name='csp_report')
```

## Handling Third-Party Services

### Google Analytics
```python
from django.utils.csp import CSP

SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, "https://www.googletagmanager.com", "https://www.google-analytics.com"],
    "img-src": [CSP.SELF, "https://www.google-analytics.com"],
    "connect-src": [CSP.SELF, "https://www.google-analytics.com"],
}
```

### Stripe
```python
from django.utils.csp import CSP

SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, "https://js.stripe.com"],
    "frame-src": [CSP.SELF, "https://js.stripe.com", "https://hooks.stripe.com"],
    "connect-src": [CSP.SELF, "https://api.stripe.com"],
}
```

### Cloudflare
```python
from django.utils.csp import CSP

SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, "https://ajax.cloudflare.com"],
    "img-src": [CSP.SELF, "https://cdnjs.cloudflare.com"],
}
```

## HTMX + CSP

When using HTMX with CSP, use the nonce approach:

```python
from django.utils.csp import CSP

SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE],
    "connect-src": [CSP.SELF],
}
```

```django
<script src="https://unpkg.com/htmx.org@2.0" nonce="{{ csp_nonce }}"></script>
```

## Testing CSP

1. **Start with report-only mode** - See what would break
2. **Check browser console** - CSP violations appear there
3. **Use CSP Evaluator** - Chrome extension to test policies
4. **Test common user flows** - Login, forms, file uploads

## Migration Strategy

```python
from django.utils.csp import CSP

# Step 1: Report only (2 weeks) — use SECURE_CSP_REPORT_ONLY
SECURE_CSP_REPORT_ONLY = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.UNSAFE_INLINE],
    "report-uri": "/api/csp-report/",
}

# Step 2: Enforce with unsafe-inline still allowed (1 month)
SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.UNSAFE_INLINE],
}

# Step 3: Switch to nonces (remove unsafe-inline)
SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE],
}

# Step 4: Full strict mode — no unsafe-*, minimal exceptions
```

Ask the user:
1. What CSP level do you need? (relaxed, moderate, strict)
2. Do you use any third-party services? (analytics, payment, CDNs)
3. Do you use HTMX or other JavaScript frameworks?
4. Any existing CSP violations to address?
