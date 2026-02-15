---
name: setup-csp
description: Configure Django 6.0 Content Security Policy for security
---

# Django 6.0 CSP Setup Skill

You are an expert Django security developer. Guide the user through setting up Content Security Policy.

## Interactive Setup Process

### Step 1: Security Assessment
Ask the user:
- What's your security requirement level?
  - **Relaxed**: Getting started, internal app
  - **Moderate**: Public-facing app with some third-party services
  - **Strict**: High-security requirement (finance, healthcare)
- What third-party services do you use?
  - Analytics (Google Analytics, etc.)
  - Payment (Stripe, PayPal)
  - CDNs (Cloudflare, AWS CloudFront)
  - Fonts (Google Fonts)
  - Scripts (HTMX, Alpine.js, React)
- Do you use HTMX or similar JavaScript frameworks?

### Step 2: Installation

```python
# settings/base.py
INSTALLED_APPS = [
    # ...
    'django.csp',  # Django 6.0 built-in
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.middleware.csp.CSPMiddleware',  # Django 6.0 built-in
    # ... other middleware ...
]
```

### Step 3: Generate CSP Configuration

Based on user's security level and services, generate appropriate configuration.

#### Relaxed Configuration
```python
# Good for development or internal tools
CSP_DEFAULT_SRC = ("'self'", "'unsafe-inline'", "'unsafe-eval'")
CSP_IMG_SRC = ("'self'", "data:", "https:")
CSP_SCRIPT_SRC = ("'self'", "'unsafe-inline'", "'unsafe-eval'")
CSP_STYLE_SRC = ("'self'", "'unsafe-inline'")
CSP_REPORT_ONLY = True  # Start in report-only mode
```

#### Moderate Configuration
```python
# Recommended for most public-facing apps
CSP_DEFAULT_SRC = ("'self'",)
CSP_SCRIPT_SRC = ("'self'", "'unsafe-inline'")  # For inline scripts
CSP_STYLE_SRC = ("'self'", "'unsafe-inline'")  # For inline styles
CSP_IMG_SRC = ("'self'", "data:", "https:")
CSP_FONT_SRC = ("'self'", "https://fonts.gstatic.com")
CSP_CONNECT_SRC = ("'self'", "https://api.example.com")
CSP_FRAME_ANCESTORS = ("'none'",)
```

#### Strict Configuration
```python
# Best security, requires nonces
CSP_DEFAULT_SRC = ("'self'",)
CSP_SCRIPT_SRC = ("'self'",)
CSP_STYLE_SRC = ("'self'",)
CSP_IMG_SRC = ("'self'", "data:")
CSP_FONT_SRC = ("'self'",)
CSP_CONNECT_SRC = ("'self'",)
CSP_FRAME_ANCESTORS = ("'none'",)
CSP_BASE_URI = ("'none'",)
CSP_FORM_ACTION = ("'self'",)
```

### Step 4: Add Third-Party Services

Generate CSP directives for user's specific services:

#### Google Analytics
```python
CSP_SCRIPT_SRC += ("https://www.googletagmanager.com", "https://www.google-analytics.com")
CSP_IMG_SRC += ("https://www.google-analytics.com",)
CSP_CONNECT_SRC += ("https://www.google-analytics.com",)
```

#### Stripe
```python
CSP_SCRIPT_SRC += ("https://js.stripe.com",)
CSP_FRAME_SRC += ("https://js.stripe.com", "https://hooks.stripe.com")
CSP_CONNECT_SRC += ("https://api.stripe.com",)
```

#### HTMX
```python
CSP_SCRIPT_SRC += ("'unsafe-eval'",)  # HTMX may need eval
```

### Step 5: Set Up Reporting

```python
# settings/base.py
CSP_REPORT_URI = '/api/csp-report/'
CSP_REPORT_ONLY = True  # Start with report-only

# Create view for reports
```

```python
# views.py
import json
import logging
from django.http import HttpResponse

logger = logging.getLogger(__name__)

@csrf_exempt  # CSP reports don't have CSRF tokens
def csp_report_view(request):
    if request.method == 'POST':
        report = json.loads(request.body)
        logger.warning(f'CSP Violation: {report}')
    return HttpResponse(status=204)
```

```python
# urls.py
path('api/csp-report/', csp_report_view, name='csp_report'),
```

### Step 6: Implementation Strategy

#### Phase 1: Report-Only (1-2 weeks)
```python
CSP_REPORT_ONLY = True
# Monitor logs for violations
```

#### Phase 2: Moderate Enforcement (1 month)
```python
CSP_REPORT_ONLY = False
# Allow unsafe-inline temporarily
```

#### Phase 3: Remove unsafe-inline
```python
# Implement nonce system
CSP_SCRIPT_SRC = ("'self'", "'nonce-%s'")
```

#### Phase 4: Strict Mode
```python
# Remove all unsafe-* directives
# Use Content-Security-Policy-Report-Only header for testing
```

### Step 7: Nonce Implementation

```python
# context processor
def csp_nonce(request):
    import secrets
    return {'csp_nonce': secrets.token_urlsafe(16)}

# middleware
class CSPNonceMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        import secrets
        request.csp_nonce = secrets.token_urlsafe(16)
        response = self.get_response(request)
        response['Content-Security-Policy'] = (
            f"default-src 'self'; "
            f"script-src 'self' 'nonce-{request.csp_nonce}'"
        )
        return response
```

```django
<!-- template -->
<script nonce="{{ csp_nonce }}">
    // Inline script
</script>
```

### Step 8: Testing

Create a test checklist:
- [ ] All pages load without console errors
- [ ] Forms submit successfully
- [ ] Third-party scripts load
- [ ] AJAX/fetch requests work
- [ ] Images and fonts display
- [ ] Browser console shows no CSP violations

## Monitoring CSP Violations

Set up logging and alerts:
```python
LOGGING = {
    'handlers': {
        'csp_violations': {
            'level': 'WARNING',
            'class': 'logging.FileHandler',
            'filename': BASE_DIR / 'logs/csp_violations.log',
        },
    },
    'loggers': {
        'django.security.csp': {
            'handlers': ['csp_violations'],
            'level': 'WARNING',
            'propagate': False,
        },
    },
}
```

## Ask the user:

1. **What security level do you need?** (relaxed/moderate/strict)
2. **What third-party services do you use?**
3. **Do you use HTMX or other JS frameworks?**
4. **What's your deployment timeline?** (affects migration strategy)
