---
name: setup-tasks
description: Set up Django 6.0 background tasks framework
---

# Django 6.0 Background Tasks Setup Skill

You are an expert Django 6.0 developer. Guide the user through setting up the built-in background tasks framework.

## Interactive Setup Process

### Step 1: Requirements Analysis
Ask the user:
- What type of background tasks do you need?
  - Email sending
  - Image/media processing
  - Webhook calls
  - Scheduled maintenance
  - Data processing/exports
- What's your expected task volume? (low/medium/high)
- Do you need scheduled (cron-like) tasks?

### Step 2: Installation
```bash
# Add to requirements/base.txt
django>=6.0

# Run migrations
python manage.py migrate
```

### Step 3: Configuration
```python
# settings/base.py
TASKS = {
    'default': {
        'BACKEND': 'django.tasks.backends.database.DatabaseBackend',
    },
}
```

> **Note:** Django handles task creation and queuing. External worker processes are required to execute enqueued tasks. Refer to Django 6.0 deployment docs for worker setup.

### Step 4: Worker Setup

Django 6.0 handles task creation and queuing. External worker processes must be set up to execute tasks. Refer to the Django 6.0 deployment documentation for your specific backend's worker requirements.

```bash
# Run migrations for the database backend
python manage.py migrate
```

### Step 5: Create Tasks Directory Structure
```
project/
├── app/
│   ├── tasks.py
│   └── tests/
│       └── test_tasks.py
```

### Step 6: Task Example Generation

Based on user needs, generate:
- Email tasks with retry logic
- Scheduled cleanup tasks
- Long-running processing tasks
- Webhook tasks with error handling

## Task Templates

### Email Task Template
```python
from django.tasks import task
from django.core.mail import send_mail
from django.contrib.auth import get_user_model
import logging

logger = logging.getLogger(__name__)

@task
def send_welcome_email(user_id):
    User = get_user_model()
    try:
        user = User.objects.get(pk=user_id)
        send_mail(
            'Welcome!',
            'Thanks for joining!',
            'noreply@example.com',
            [user.email],
        )
    except User.DoesNotExist:
        logger.warning(f'User {user_id} not found for welcome email')

# Usage: send_welcome_email.enqueue(user_id=42)
```

### Cleanup Task Template
```python
from django.tasks import task
from django.utils import timezone
import logging

logger = logging.getLogger(__name__)

@task
def daily_cleanup():
    from myapp.models import ExpiredToken
    count = ExpiredToken.objects.filter(
        expires_at__lt=timezone.now()
    ).delete()[0]
    logger.info(f'Cleaned up {count} expired tokens')

# Usage: daily_cleanup.enqueue()
```

## Monitoring

Refer to the Django 6.0 tasks documentation for backend-specific monitoring and result inspection capabilities. The database backend stores task results that can be queried for status tracking.

## Testing Tasks
```python
from django.test import TestCase, override_settings
from myapp.tasks import my_task

class TaskTests(TestCase):
    @override_settings(TASKS={'default': {'BACKEND': 'django.tasks.backends.immediate.ImmediateBackend'}})
    def test_task_execution(self):
        # ImmediateBackend executes tasks synchronously for testing
        result = my_task.enqueue(arg1=val1, arg2=val2)
        self.assertEqual(result.status, result.COMPLETE)
```

## Ask the user:

1. **What tasks do you need?** (email, processing, scheduled, etc.)
2. **Do you need scheduled/cron tasks?**
3. **Your deployment setup?** (Docker, systemd, supervisor, etc.)
4. **Expected task volume?** (determines worker count)
