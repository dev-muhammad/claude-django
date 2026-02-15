---
name: django:task
description: Create background tasks using Django 6.0 built-in tasks framework
arguments:
  - name: task_type
    description: Type of task (email, processing, scheduled, cleanup)
    required: false
---

# Django 6.0 Background Tasks

You are an expert Django 6.0 developer. Generate background tasks using Django's built-in tasks framework (no Celery required!).

## Django 6.0 Tasks Framework

Django 6.0 introduced a **native background tasks framework** that eliminates the need for external packages like Celery for many use cases.

## Task Types

### 1. **Simple Background Tasks**
Execute code outside the request-response cycle:

```python
from django.tasks import task

@task
def send_welcome_email(user_id):
    from django.contrib.auth import get_user_model
    User = get_user_model()
    user = User.objects.get(pk=user_id)
    send_mail(
        'Welcome!',
        'Thanks for signing up.',
        'from@example.com',
        [user.email],
    )
```

### 2. **Scheduled Tasks**
Run tasks on a schedule:

```python
from django.tasks import task
from django.tasks.schedule import crontab

@task(schedule=crontab(minute='*/15'))
def cleanup_expired_sessions():
    from django.contrib.sessions.models import Session
    Session.objects.filter(expire_date__lt=timezone.now()).delete()
```

### 3. **Retry on Failure**
Automatic retry with exponential backoff:

```python
from django.tasks import task
from django.tasks.retry import Retry

@task(
    max_retries=3,
    retry_backoff=True,
    retry_backoff_max=300,
)
def process_payment(order_id):
    # Will retry up to 3 times if exception is raised
    pass
```

### 4. **Task Chaining**
Run tasks in sequence:

```python
from django.tasks import chain

def process_order(order_id):
    workflow = chain(
        validate_order.s(order_id),
        charge_payment.s(),
        send_confirmation.s(),
    )
    workflow()
```

### 5. **Task Groups**
Run tasks in parallel:

```python
from django.tasks import group

def notify_all_users(message):
    tasks = [send_email.s(user_id, message) for user_id in user_ids]
    group(tasks)()
```

## Configuration

### settings/base.py
```python
# Task backend configuration
TASKS = {
    'default': {
        'BACKEND': 'django.tasks.backends.database.DatabaseBackend',
        'RESULT_EXPIRES': 3600,  # Results cached for 1 hour
        'MAX_RETRIES': 3,
        'RETRY_BACKOFF': True,
    }
}

# Task worker process
# Run with: python manage.py task_worker
```

### Installation
```python
INSTALLED_APPS = [
    # ...
    'django.tasks',
]
```

### Migration
```bash
python manage.py migrate
```

### Starting Worker
```bash
# Development
python manage.py task_worker

# Production with multiple workers
python manage.py task_worker --concurrency 4
```

## Task Best Practices

1. **Always pass IDs, not objects**
   ```python
   # BAD
   @task
   def process_user(user):
       pass

   # GOOD
   @task
   def process_user(user_id):
       User = get_user_model()
       user = User.objects.get(pk=user_id)
   ```

2. **Use .delay() to execute**
   ```python
   # Trigger the task
   send_welcome_email.delay(user_id)
   ```

3. **Handle task results**
   ```python
   result = my_task.delay(arg1, arg2)
   task_id = result.id

   # Check status
   result = AsyncResult(task_id)
   if result.ready():
       value = result.get()
   ```

4. **Use task decorators properly**
   ```python
   from django.tasks import task, shared_task

   @task  # Only for this project
   def my_task():
       pass

   @shared_task  # Can be used in reusable apps
   def shared_task():
       pass
   ```

## Common Patterns

### Email Sending
```python
@task
def send_email_async(subject, message, to):
    from django.core.mail import send_mail
    send_mail(subject, message, 'from@example.com', [to])
```

### Image Processing
```python
@task
def generate_thumbnail(image_id, sizes):
    from PIL import Image
    image = Image.objects.get(pk=image_id)
    img = Image.open(image.file.path)
    # Generate thumbnails...
```

### Webhook Calls
```python
@task(max_retries=5)
def send_webhook(url, payload):
    import requests
    response = requests.post(url, json=payload, timeout=10)
    response.raise_for_status()
```

Ask the user:
1. What type of task do you need? (email, processing, scheduled, webhook)
2. What arguments should the task accept?
3. Does it need to run on a schedule?
4. Should it retry on failure? How many times?
