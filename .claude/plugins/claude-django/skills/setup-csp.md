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

### Step 3: Generate CSP Configuration

Based on user's security level and services, generate appropriate configuration.

#### Relaxed Configuration
```python
from django.utils.csp import CSP

# Good for development or internal tools
SECURE_CSP = {
    "default-src": [CSP.SELF, CSP.UNSAFE_INLINE, CSP.UNSAFE_EVAL],
    "img-src": [CSP.SELF, "data:", "https:"],
    "script-src": [CSP.SELF, CSP.UNSAFE_INLINE, CSP.UNSAFE_EVAL],
    "style-src": [CSP.SELF, CSP.UNSAFE_INLINE],
}
```

#### Moderate Configuration
```python
from django.utils.csp import CSP

# Recommended for most public-facing apps
SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE],
    "style-src": [CSP.SELF, CSP.UNSAFE_INLINE],
    "img-src": [CSP.SELF, "data:", "https:"],
    "font-src": [CSP.SELF, "https://fonts.gstatic.com"],
    "connect-src": [CSP.SELF],
    "frame-ancestors": [CSP.NONE],
}
```

#### Strict Configuration
```python
from django.utils.csp import CSP

# Best security, requires nonces for all inline code
SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE],
    "style-src": [CSP.SELF, CSP.NONCE],
    "img-src": [CSP.SELF, "data:"],
    "font-src": [CSP.SELF],
    "connect-src": [CSP.SELF],
    "frame-ancestors": [CSP.NONE],
    "base-uri": [CSP.NONE],
    "form-action": [CSP.SELF],
}
```

### Step 4: Add Third-Party Services

Generate CSP directives for user's specific services. Third-party domains are added as strings alongside CSP constants:

#### Google Analytics
```python
SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE, "https://www.googletagmanager.com", "https://www.google-analytics.com"],
    "img-src": [CSP.SELF, "https://www.google-analytics.com"],
    "connect-src": [CSP.SELF, "https://www.google-analytics.com"],
}
```

#### Stripe
```python
SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE, "https://js.stripe.com"],
    "frame-src": [CSP.SELF, "https://js.stripe.com", "https://hooks.stripe.com"],
    "connect-src": [CSP.SELF, "https://api.stripe.com"],
}
```

#### HTMX
```python
SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE],
    "connect-src": [CSP.SELF],
}
```

```django
<script src="https://unpkg.com/htmx.org@2.0" nonce="{{ csp_nonce }}"></script>
```

### Step 5: Set Up Reporting

```python
# settings/development.py — Report-only mode for testing
from django.utils.csp import CSP

SECURE_CSP_REPORT_ONLY = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.UNSAFE_INLINE],
    "report-uri": "/api/csp-report/",
}
```

```python
# views.py
import json
import logging
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_exempt

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
# Use SECURE_CSP_REPORT_ONLY to monitor without blocking
SECURE_CSP_REPORT_ONLY = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.UNSAFE_INLINE],
    "report-uri": "/api/csp-report/",
}
```

#### Phase 2: Enforce with unsafe-inline (1 month)
```python
# Switch to SECURE_CSP to start enforcing
SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.UNSAFE_INLINE],
}
```

#### Phase 3: Switch to nonces
```python
# Remove unsafe-inline, use nonces instead
SECURE_CSP = {
    "default-src": [CSP.SELF],
    "script-src": [CSP.SELF, CSP.NONCE],
    "style-src": [CSP.SELF, CSP.NONCE],
}
```

#### Phase 4: Strict Mode
```python
# No unsafe-*, minimal third-party exceptions
```

### Step 7: Using Nonces in Templates

The `django.template.context_processors.csp` context processor provides a `csp_nonce` variable:

```django
<!-- Inline scripts need the nonce -->
<script nonce="{{ csp_nonce }}">
    document.addEventListener('DOMContentLoaded', function() {
        // Your inline script
    });
</script>

<!-- External scripts loaded via nonce -->
<script src="https://unpkg.com/htmx.org@2.0" nonce="{{ csp_nonce }}"></script>
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
