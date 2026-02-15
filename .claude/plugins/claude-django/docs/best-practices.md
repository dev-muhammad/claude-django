# Django Best Practices

This document outlines best practices for Django development to build secure, maintainable, and performant applications.

## Project Setup

### Start with a Virtual Environment
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install django
```

### Use Django LTS Versions
- Use Django LTS (Long Term Support) for production
- Django 5.2 LTS (April 2025 - April 2028)
- Django 4.2 LTS (November 2022 - April 2026)

### Keep Requirements Updated
```
requirements/
├── base.txt       # All dependencies
├── development.txt # + dev tools (debug-toolbar, coverage, etc.)
└── production.txt  # + production tools (gunicorn, whitenoise, etc.)
```

## Database Best Practices

### Use PostgreSQL in Production
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST'),
        'PORT': os.environ.get('DB_PORT'),
        'CONN_MAX_AGE': 600,  # Persistent connections
    }
}
```

### Use SQLite for Development
```python
# settings/local.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

### Use Indexes Strategically
```python
class Post(models.Model):
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True, db_index=True)
    created_at = models.DateTimeField(db_index=True)

    class Meta:
        indexes = [
            models.Index(fields=['slug']),
            models.Index(fields=['-created_at', 'status']),
        ]
```

## Model Design

### Always Use Custom User Model
```python
# Create before first migration
# settings/base.py
AUTH_USER_MODEL = 'users.User'

# apps/users/models.py
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    email = models.EmailField(unique=True)
    bio = models.TextField(blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True)

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']
```

### Use Constraints Instead of unique_together
```python
class Enrollment(models.Model):
    student = models.ForeignKey('Student', on_delete=models.CASCADE)
    course = models.ForeignKey('Course', on_delete=models.CASCADE)

    class Meta:
        constraints = [
            models.UniqueConstraint(
                fields=['student', 'course'],
                name='unique_student_course'
            )
        ]
```

### Use Signals Sparingly
```python
# Prefer overriding save() when possible
class Post(models.Model):
    published_at = models.DateTimeField(null=True, blank=True)
    status = models.CharField(...)

    def save(self, *args, **kwargs):
        if self.status == 'published' and not self.published_at:
            self.published_at = timezone.now()
        super().save(*args, **kwargs)
```

## View Best Practices

### Prefer Class-Based Views for CRUD
```python
# Use generic views for standard operations
class PostListView(ListView):
    model = Post
    paginate_by = 20

# Use function-based views for custom logic
def custom_report(request):
    # Complex business logic
    pass
```

### Use Mixins for Reusable Behavior
```python
class AuthorRequiredMixin(LoginRequiredMixin, UserPassesTestMixin):
    def test_func(self):
        return self.get_object().author == self.request.user

class PostUpdateView(AuthorRequiredMixin, UpdateView):
    model = Post
    # ...
```

### Use get_queryset() for Filtering
```python
class PostListView(ListView):
    def get_queryset(self):
        queryset = Post.objects.filter(status='published')
        tag_slug = self.kwargs.get('tag_slug')
        if tag_slug:
            queryset = queryset.filter(tags__slug=tag_slug)
        return queryset
```

## Performance Optimization

### Use select_related() for ForeignKeys
```python
# One query instead of N queries
posts = Post.objects.select_related('author').all()

for post in posts:
    print(post.author.username)  # No additional query
```

### Use prefetch_related() for ManyToMany
```python
# One query for posts, one query for all tags
posts = Post.objects.prefetch_related('tags').all()

for post in posts:
    print([tag.name for tag in post.tags.all()])
```

### Use only() and defer() for Large Fields
```python
# Only fetch specific fields
posts = Post.objects.only('title', 'slug', 'status')

# Exclude large fields from query
posts = Post.objects.defer('content')
```

### Use QuerySet Methods Efficiently
```python
# Good - QuerySet is lazy
queryset = Post.objects.filter(status='published')
if some_condition:
    queryset = queryset.filter(author=request.user)

# Bad - Evaluates immediately
posts = list(Post.objects.filter(status='published'))
if some_condition:
    posts = [p for p in posts if p.author == request.user]
```

### Cache Expensive Operations
```python
from django.core.cache import cache

def get_popular_posts():
    cache_key = 'popular_posts'
    posts = cache.get(cache_key)
    if posts is None:
        posts = Post.objects.annotate(
            views_count=models.Count('views')
        ).order_by('-views_count')[:10]
        cache.set(cache_key, posts, 3600)  # 1 hour
    return posts
```

## Security Best Practices

### Never Commit Secrets
```python
# .env
SECRET_KEY=your-secret-key-here
DATABASE_URL=postgresql://...

# settings.py
import os
SECRET_KEY = os.environ.get('SECRET_KEY')
```

### Use django-environ for Config
```python
import environ

env = environ.Env()
environ.Env.read_env()

SECRET_KEY = env('SECRET_KEY')
DEBUG = env.bool('DEBUG', default=False)
```

### Always Use HTTPS in Production
```python
# settings/production.py
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_HSTS_SECONDS = 31536000
```

### Use django-axes for Rate Limiting
```python
INSTALLED_APPS = ['axes']

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'axes.middleware.AxesMiddleware',
    # ...
]

AXES_FAILURE_LIMIT = 5
AXES_COOLOFF_TIME = 30  # minutes
```

### Validate File Uploads
```python
from django.core.exceptions import ValidationError

def validate_file_extension(value):
    allowed_extensions = ['pdf', 'doc', 'docx']
    ext = value.name.split('.')[-1].lower()
    if ext not in allowed_extensions:
        raise ValidationError(f'File type {ext} is not allowed.')

class Document(models.Model):
    file = models.FileField(
        upload_to='documents/',
        validators=[validate_file_extension]
    )
```

### Sanitize User Input
```python
from django.utils.html import strip_tags

def clean_content(self):
    content = self.cleaned_data['content']
    return strip_tags(content)  # Remove HTML tags
```

## Form Best Practices

### Use ModelForm When Possible
```python
class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content', 'status']
```

### Add Custom Validation
```python
def clean_slug(self):
    slug = self.cleaned_data['slug']
    if not slug:
        raise ValidationError('Slug is required.')
    return slug
```

### Use Widgets for Better UX
```python
class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = '__all__'
        widgets = {
            'content': forms.Textarea(attrs={'rows': 10}),
            'published_at': forms.DateTimeInput(attrs={'type': 'datetime-local'}),
            'tags': forms.CheckboxSelectMultiple(),
        }
```

## Template Best Practices

### Use Template Inheritance
```django
<!-- base.html -->
{% load static %}
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
</head>
<body>
    {% block content %}{% endblock %}
</body>
</html>

<!-- post_list.html -->
{% extends "base.html" %}
{% block title %}Posts{% endblock %}
{% block content %}
<h1>Posts</h1>
{% endblock %}
```

### Use Template Partials
```django
<!-- partials/navbar.html -->
<nav>
    <a href="{% url 'home' %}">Home</a>
    {% if user.is_authenticated %}
        <a href="{% url 'logout' %}">Logout</a>
    {% endif %}
</nav>
```

### Load Static Files Properly
```django
{% load static %}
<link rel="stylesheet" href="{% static 'css/style.css' %}">
<script src="{% static 'js/app.js' %}"></script>
```

### Use Context Processors for Global Variables
```python
# context_processors.py
def site_info(request):
    return {
        'site_name': settings.SITE_NAME,
        'current_year': datetime.now().year,
    }

# settings.py
TEMPLATES = [{
    'OPTIONS': {
        'context_processors': [
            'myapp.context_processors.site_info',
        ],
    },
}]
```

## Testing Best Practices

### Test Models First
```python
class PostModelTests(TestCase):
    def test_str_representation(self):
        post = Post.objects.create(title='Test Post')
        self.assertEqual(str(post), 'Test Post')
```

### Test Views Properly
```python
class PostViewTests(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(username='test')
        self.client.force_login(self.user)

    def test_list_view_requires_login(self):
        self.client.logout()
        response = self.client.get('/posts/')
        self.assertEqual(response.status_code, 302)  # Redirect
```

### Use Factory Boy for Test Data
```python
import factory
from .models import Post

class PostFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = Post

    title = factory.Faker('sentence')
    content = factory.Faker('text')
```

### Test Edge Cases
```python
def test_post_with_no_content(self):
    post = Post.objects.create(title='No Content', content='')
    self.assertEqual(post.reading_time, 1)  # Minimum 1 minute
```

## Deployment Best Practices

### Use Gunicorn in Production
```bash
pip install gunicorn
gunicorn myproject.wsgi:application --bind 0.0.0.0:8000
```

### Use WhiteNoise for Static Files
```python
INSTALLED_APPS = ['whitenoise.runserver_nostatic', ...]
MIDDLEWARE.insert(1, 'whitenoise.middleware.WhiteNoiseMiddleware')
```

### Use Environment-Specific Settings
```python
# settings/production.py
DEBUG = False
ALLOWED_HOSTS = ['yourdomain.com']

# settings/local.py
DEBUG = True
ALLOWED_HOSTS = ['localhost', '127.0.0.1']
```

### Set Up Logging
```python
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
        'level': 'INFO',
    },
}
```

### Use Django Commands for Maintenance
```bash
# Collect static files
python manage.py collectstatic --noinput

# Create and apply migrations
python manage.py makemigrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser --noinput
```

## API Best Practices (DRF)

### Use ViewSets for CRUD Operations
```python
from rest_framework import viewsets

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
```

### Use Serializers Properly
```python
class PostSerializer(serializers.ModelSerializer):
    author = serializers.StringRelatedField(read_only=True)
    reading_time = serializers.ReadOnlyField()

    class Meta:
        model = Post
        fields = '__all__'
```

### Use Pagination
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
}
```

### Use Throttling for Rate Limiting
```python
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle',
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day',
    },
}
```
