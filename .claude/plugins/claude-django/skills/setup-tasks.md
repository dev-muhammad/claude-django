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
INSTALLED_APPS = [
    # ...
    'django.tasks',
]

MIDDLEWARE = [
    # ...
    'django.tasks.middleware.TaskMiddleware',  # Optional, for request-bound tasks
]

TASKS = {
    'default': {
        'BACKEND': 'django.tasks.backends.database.DatabaseBackend',
        'RESULT_EXPIRES': 3600,
        'MAX_RETRIES': 3,
        'RETRY_BACKOFF': True,
    },
    'high_priority': {
        'BACKEND': 'django.tasks.backends.database.DatabaseBackend',
        'MAX_RETRIES': 5,
    },
}

# Task routing (optional)
TASK_ROUTES = {
    'app.tasks.send_email': 'high_priority',
    'app.tasks.process_image': 'default',
}
```

### Step 4: Worker Setup

Create `procfile.sh` or supervisor config:
```bash
#!/bin/bash
# Start task workers
python manage.py task_worker --concurrency=4 --loglevel=info
```

For development:
```bash
# Single worker, verbose
python manage.py task_worker --concurrency=1 --loglevel=debug
```

For production (supervisor):
```ini
[program:django-tasks]
command=/path/to/venv/bin/python manage.py task_worker --concurrency=4
directory=/path/to/project
user=www-data
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/var/log/django-tasks.log
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
from django.tasks import shared_task
from django.core.mail import send_mail
from django.contrib.auth import get_user_model

@shared_task(
    max_retries=3,
    retry_backoff=True,
)
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
        # Don't retry if user doesn't exist
        logger.warning(f'User {user_id} not found for welcome email')
```

### Scheduled Task Template
```python
from django.tasks import shared_task
from django.tasks.schedule import crontab
from django.utils import timezone

@shared_task(schedule=crontab(hour=0, minute=0))  # Daily at midnight
def daily_cleanup():
    from myapp.models import ExpiredToken
    count = ExpiredToken.objects.filter(
        expires_at__lt=timezone.now()
    ).delete()[0]
    logger.info(f'Cleaned up {count} expired tokens')
```

## Monitoring

### Admin Integration
```python
# admin.py
from django.tasks import admin
admin.site.register(django.tasks.models.TaskResult)
```

### Monitoring Views
```python
def task_stats(request):
    from django.tasks.models import TaskResult
    stats = {
        'pending': TaskResult.objects.filter(status='pending').count(),
        'started': TaskResult.objects.filter(status='started').count(),
        'success': TaskResult.objects.filter(status='success').count(),
        'failure': TaskResult.objects.filter(status='failure').count(),
    }
    return JsonResponse(stats)
```

## Testing Tasks
```python
from django.test import TestCase
from myapp.tasks import my_task

class TaskTests(TestCase):
    def test_task_execution(self):
        result = my_task.delay(arg1, arg2)
        self.assertTrue(result.ready())
        self.assertEqual(result.get(), expected_value)
```

## Ask the user:

1. **What tasks do you need?** (email, processing, scheduled, etc.)
2. **Do you need scheduled/cron tasks?**
3. **Your deployment setup?** (Docker, systemd, supervisor, etc.)
4. **Expected task volume?** (determines worker count)
