# Django Standards & Conventions

This document outlines the coding standards and conventions to follow when working with Django projects.

## Project Structure

### Apps Directory Pattern
```
project_name/
├── apps/
│   ├── __init__.py
│   ├── core/          # Core utilities, base classes
│   ├── users/         # Custom user model
│   └── blog/          # Feature-specific apps
```

Add apps directory to Python path in `settings/base.py`:
```python
import sys
from pathlib import Path

sys.path.insert(0, str(BASE_DIR / 'apps'))
```

### Modular Settings Structure
```
project_name/settings/
├── __init__.py        # Default to local settings
├── base.py            # Common settings (all environments)
├── local.py           # Development settings (DEBUG=True, SQLite)
├── test.py            # Test settings (in-memory SQLite)
└── production.py      # Production settings (security, PostgreSQL, logging)
```

## Naming Conventions

| Type | Convention | Examples |
|------|------------|----------|
| Models | PascalCase | `User`, `BlogPost`, `OrderItem` |
| Views | PascalCase | `PostListView`, `PostDetailView` |
| Template tags | snake_case | `post_list.html`, `post_detail.html` |
| URLs | kebab-case | `/blog-posts/`, `/create-post/` |
| Functions | snake_case | `get_absolute_url`, `increment_views` |
| Variables | snake_case | `user_id`, `is_active`, `created_at` |
| Constants | UPPER_SNAKE_CASE | `MAX_UPLOAD_SIZE`, `API_KEY` |

## Model Standards

### Always Include
```python
class MyModel(models.Model):
    name = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-created_at']
        verbose_name = "my model"
        verbose_name_plural = "my models"

    def __str__(self):
        return self.name

    def get_absolute_url(self):
        from django.urls import reverse
        return reverse('mymodel-detail', kwargs={'slug': self.slug})
```

### Field Types by Use Case
| Field Type | Use Case |
|------------|----------|
| `CharField` | Short strings (<255 chars) |
| `TextField` | Long text content |
| `EmailField` | Email addresses (validates) |
| `URLField` | URLs (validates) |
| `SlugField` | URL-friendly identifiers |
| `DateField` | Dates without time |
| `DateTimeField` | Dates with time |
| `BooleanField` | True/False values |
| `IntegerField` | Whole numbers |
| `DecimalField` | Monetary values (max_digits, decimal_places) |
| `FileField` | File uploads (upload_to path) |
| `ImageField` | Image uploads (requires Pillow) |
| `ForeignKey` | Many-to-one relationships |
| `ManyToManyField` | Many-to-many relationships |

### Relationship Best Practices
```python
# Always use related_name on ForeignKey
author = models.ForeignKey(
    User,
    on_delete=models.CASCADE,
    related_name='blog_posts'
)

# Use on_delete appropriately
# CASCADE: Delete related objects
# PROTECT: Prevent deletion
# SET_NULL: Set to null (null=True required)
# SET_DEFAULT: Set to default value
# DO_NOTHING: Do nothing (not recommended)
```

## View Standards

### Function-Based Views (FBV)
```python
from django.shortcuts import render, get_object_or_404, redirect
from django.contrib.auth.decorators import login_required

@login_required
def post_list(request):
    posts = Post.objects.filter(status='published')
    return render(request, 'blog/post_list.html', {'posts': posts})

def post_detail(request, slug):
    post = get_object_or_404(Post, slug=slug)
    return render(request, 'blog/post_detail.html', {'post': post})
```

### Class-Based Views (CBV)
```python
from django.views.generic import ListView, DetailView, CreateView
from django.contrib.auth.mixins import LoginRequiredMixin

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
    paginate_by = 10

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'
    slug_url_kwarg = 'slug'

class PostCreateView(LoginRequiredMixin, CreateView):
    model = Post
    template_name = 'blog/post_form.html'
    fields = ['title', 'slug', 'content']
    success_url = reverse_lazy('blog:post-list')
```

## URL Standards

### Use `path()` Over `url()`
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.PostListView.as_view(), name='post-list'),
    path('<slug:slug>/', views.PostDetailView.as_view(), name='post-detail'),
    path('create/', views.PostCreateView.as_view(), name='post-create'),
]
```

### URL Converters
- `<int:pk>` - Integer primary key
- `<str:slug>` - String slug
- `<uuid:id>` - UUID
- `<path:subpath>` - Full path including slashes

### Always Name Your URLs
```python
# Reverse in Python
url = reverse('blog:post-detail', kwargs={'slug': 'my-post'})

# Reverse in templates
{% url 'blog:post-detail' slug=post.slug %}
```

## Template Standards

### Base Template
```django
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}My Site{% endblock %}</title>
</head>
<body>
    <header>{% include "partials/header.html" %}</header>
    <main>{% block content %}{% endblock %}</main>
    <footer>{% include "partials/footer.html" %}</footer>
</body>
</html>
```

### Template Loading
```python
# settings/base.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        # ...
    },
]
```

## Form Standards

### ModelForm
```python
from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'slug', 'content', 'status']
        widgets = {
            'content': forms.Textarea(attrs={'rows': 10}),
        }
```

### Form Validation
```python
def clean_slug(self):
    slug = self.cleaned_data['slug']
    if Post.objects.filter(slug=slug).exists():
        raise forms.ValidationError("This slug already exists.")
    return slug
```

## Admin Standards

### Use @admin.register Decorator
```python
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'status', 'created_at']
    list_filter = ['status', 'created_at']
    search_fields = ['title', 'content']
    date_hierarchy = 'created_at'
    ordering = ['-created_at']
```

## Settings Standards

### Use Environment Variables
```python
import os
from pathlib import Path

SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')
DEBUG = os.environ.get('DEBUG', 'False') == 'True'
DATABASE_URL = os.environ.get('DATABASE_URL')
```

### Database Configuration
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST'),
        'PORT': os.environ.get('DB_PORT'),
    }
}
```

## Security Standards

### Always in Production
```python
# settings/production.py
DEBUG = False

SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'
```

## Testing Standards

### Use TestCase
```python
from django.test import TestCase
from django.urls import reverse
from .models import Post

class PostModelTests(TestCase):
    def setUp(self):
        self.post = Post.objects.create(title='Test', slug='test')

    def test_str_representation(self):
        self.assertEqual(str(self.post), 'Test')

    def test_get_absolute_url(self):
        url = reverse('post-detail', kwargs={'slug': 'test'})
        self.assertEqual(self.post.get_absolute_url(), url)
```

## Performance Standards

### Optimize Queries
```python
# Good - single query with select_related
Post.objects.select_related('author').all()

# Good - efficient with prefetch_related
Post.objects.prefetch_related('tags', 'comments').all()

# Bad - N+1 query problem
posts = Post.objects.all()
for post in posts:
    print(post.author.username)  # Query for each post
```

### Use Pagination
```python
class PostListView(ListView):
    paginate_by = 20  # Items per page
```

## Documentation Standards

### Docstrings
```python
def get_absolute_url(self):
    """
    Return the absolute URL for this post.

    Returns:
        str: The URL path to the post detail page.
    """
    return reverse('post-detail', kwargs={'slug': self.slug})
```

### Comments
```python
# Calculate reading time based on word count (200 words = 1 minute)
word_count = len(self.content.split())
reading_time = max(1, word_count // 200)
```

## Git Standards

### .gitignore
```
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
.venv/
ENV/
*.egg-info/
.pytest_cache/
.coverage
htmlcov/
db.sqlite3
/staticfiles/
/media/
/logs/
*.log
.env
.DS_Store
```

### Commit Messages
```
feat: add user authentication
fix: resolve login redirect loop
docs: update README with deployment guide
refactor: simplify view logic
test: add post model tests
chore: update dependencies
```
