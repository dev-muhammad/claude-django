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

### 2. **Enqueuing Tasks**
Use `.enqueue()` to dispatch a task for background execution:

```python
from django.tasks import task

@task
def send_welcome_email(user_id):
    from django.contrib.auth import get_user_model
    User = get_user_model()
    user = User.objects.get(pk=user_id)
    send_mail('Welcome!', 'Thanks for signing up.', 'from@example.com', [user.email])

# Dispatch the task
result = send_welcome_email.enqueue(user_id=42)
```

### 3. **Task Results**
Check the status and result of enqueued tasks:

```python
result = send_welcome_email.enqueue(user_id=42)

# Check task status
print(result.status)        # PENDING, RUNNING, COMPLETE, FAILED
print(result.id)            # Unique task ID

# Get result when complete
if result.status == result.COMPLETE:
    value = result.result
```

### 4. **Task Sequencing**
Run tasks in sequence by enqueuing from within a task:

```python
@task
def process_order(order_id):
    validate_order(order_id)
    charge_payment.enqueue(order_id=order_id)

@task
def charge_payment(order_id):
    # After payment, enqueue confirmation
    send_confirmation.enqueue(order_id=order_id)
```

### 5. **Parallel Task Dispatch**
Enqueue multiple tasks concurrently:

```python
def notify_all_users(message):
    for user_id in user_ids:
        send_email.enqueue(user_id=user_id, message=message)
```

## Configuration

### settings/base.py
```python
# Task backend configuration
TASKS = {
    'default': {
        'BACKEND': 'django.tasks.backends.database.DatabaseBackend',
    }
}
```

> **Important:** Django handles task creation and queuing. External worker processes are required to execute enqueued tasks. See Django's deployment docs for worker setup.

### Migration
```bash
python manage.py migrate
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

2. **Use .enqueue() to dispatch**
   ```python
   # Dispatch the task for background execution
   result = send_welcome_email.enqueue(user_id=user_id)
   ```

3. **Handle task results**
   ```python
   result = my_task.enqueue(arg1=val1, arg2=val2)
   task_id = result.id

   # Check status
   print(result.status)  # PENDING, RUNNING, COMPLETE, FAILED
   if result.status == result.COMPLETE:
       value = result.result
   ```

4. **Use the @task decorator**
   ```python
   from django.tasks import task

   @task
   def my_task():
       pass
   ```

## Common Patterns

### Email Sending
```python
@task
def send_email_async(subject, message, to):
    from django.core.mail import send_mail
    send_mail(subject, message, 'from@example.com', [to])

# Usage
send_email_async.enqueue(subject="Hello", message="World", to="user@example.com")
```

### Image Processing
```python
@task
def generate_thumbnail(image_id, sizes):
    from myapp.models import UploadedImage
    from PIL import Image as PILImage
    image = UploadedImage.objects.get(pk=image_id)
    img = PILImage.open(image.file.path)
    # Generate thumbnails...

# Usage
generate_thumbnail.enqueue(image_id=42, sizes=[100, 200, 400])
```

### Webhook Calls
```python
@task
def send_webhook(url, payload):
    import requests
    response = requests.post(url, json=payload, timeout=10)
    response.raise_for_status()

# Usage
send_webhook.enqueue(url="https://api.example.com/hook", payload={"event": "order.created"})
```

Ask the user:
1. What type of task do you need? (email, processing, scheduled, webhook)
2. What arguments should the task accept?
3. Does it need to run on a schedule?
4. Should it retry on failure? How many times?
