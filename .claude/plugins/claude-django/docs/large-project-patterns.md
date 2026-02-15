# Large Project Patterns

This document outlines organizational patterns for large Django projects, including per-class file structure.

## Per-Class File Structure

For large projects, organizing code by class in separate files improves maintainability and navigation.

### Models Directory Structure

```
myapp/
├── __init__.py
├── models/
│   ├── __init__.py
│   ├── base.py              # Abstract base models
│   ├── mixins.py            # Model mixins and shared fields
│   ├── user.py              # User-related models
│   ├── post.py             # Post-related models
│   ├── comment.py          # Comment-related models
│   ├── tag.py              # Tag-related models
│   └── querysets.py        # Custom QuerySets
```

### models/__init__.py - Import All Models

```python
# myapp/models/__init__.py

# Import all models for convenient access
from .base import (
    TimeStampedModel,
    TitleSlugModel,
    PublishableModel,
    BaseModel,
)
from .mixins import (
    SoftDeleteMixin,
    MetadataMixin,
    SEOFieldsMixin,
)
from .user import User, UserProfile
from .post import Post, Category
from .comment import Comment
from .tag import Tag

# This makes models available as:
# from myapp.models import Post
# from myapp.models import TimeStampedModel

__all__ = [
    # Base models
    'TimeStampedModel',
    'TitleSlugModel',
    'PublishableModel',
    'BaseModel',
    # Mixins
    'SoftDeleteMixin',
    'MetadataMixin',
    'SEOFieldsMixin',
    # User models
    'User',
    'UserProfile',
    # Content models
    'Post',
    'Category',
    'Comment',
    'Tag',
]
```

### models/base.py - Base Models

```python
# myapp/models/base.py

from django.db import models
from django.utils.translation import gettext_lazy as _


class TimeStampedModel(models.Model):
    """Abstract base model with timestamp fields."""

    created_at = models.DateTimeField(
        verbose_name=_('created at'),
        auto_now_add=True,
        db_index=True
    )
    updated_at = models.DateTimeField(
        verbose_name=_('updated at'),
        auto_now=True,
        db_index=True
    )

    class Meta:
        abstract = True
        ordering = ['-created_at']


class TitleSlugModel(models.Model):
    """Abstract base model with title and slug fields."""

    title = models.CharField(
        verbose_name=_('title'),
        max_length=255,
        db_index=True
    )
    slug = models.SlugField(
        verbose_name=_('slug'),
        max_length=255,
        unique=True,
        db_index=True
    )

    class Meta:
        abstract = True
        ordering = ['title']

    def __str__(self):
        return self.title


class PublishableModel(models.Model):
    """Abstract base model with publish workflow."""

    class Status(models.TextChoices):
        DRAFT = 'draft', _('Draft')
        PUBLISHED = 'published', _('Published')
        ARCHIVED = 'archived', _('Archived')
        SCHEDULED = 'scheduled', _('Scheduled')

    status = models.CharField(
        verbose_name=_('status'),
        max_length=20,
        choices=Status.choices,
        default=Status.DRAFT,
        db_index=True
    )
    published_at = models.DateTimeField(
        verbose_name=_('published at'),
        null=True,
        blank=True,
        db_index=True
    )

    class Meta:
        abstract = True


class BaseModel(TimeStampedModel):
    """Common base model for all project models."""

    class Meta:
        abstract = True
        ordering = ['-created_at']
```

### models/mixins.py - Model Mixins

```python
# myapp/models/mixins.py

from django.db import models
from django.utils.translation import gettext_lazy as _


class SoftDeleteMixin(models.Model):
    """Mixin for soft delete functionality."""

    is_deleted = models.BooleanField(
        verbose_name=_('is deleted'),
        default=False,
        db_index=True
    )
    deleted_at = models.DateTimeField(
        verbose_name=_('deleted at'),
        null=True,
        blank=True
    )

    def soft_delete(self):
        """Mark the object as deleted without removing from database."""
        from django.utils import timezone
        self.is_deleted = True
        self.deleted_at = timezone.now()
        self.save()

    def restore(self):
        """Restore a soft-deleted object."""
        self.is_deleted = False
        self.deleted_at = None
        self.save()

    class Meta:
        abstract = True


class MetadataMixin(models.Model):
    """Mixin for metadata fields."""

    metadata = models.JSONField(
        verbose_name=_('metadata'),
        default=dict,
        blank=True
    )

    class Meta:
        abstract = True


class SEOFieldsMixin(models.Model):
    """Mixin for SEO-related fields."""

    meta_title = models.CharField(
        verbose_name=_('meta title'),
        max_length=255,
        blank=True
    )
    meta_description = models.TextField(
        verbose_name=_('meta description'),
        blank=True
    )
    meta_keywords = models.CharField(
        verbose_name=_('meta keywords'),
        max_length=255,
        blank=True
    )

    class Meta:
        abstract = True
```

### models/user.py - User Models

```python
# myapp/models/user.py

from django.contrib.auth.models import AbstractUser
from django.db import models
from django.utils.translation import gettext_lazy as _

from .base import TimeStampedModel
from .mixins import SEOFieldsMixin


class User(AbstractUser):
    """Custom user model extending Django's AbstractUser."""

    email = models.EmailField(
        verbose_name=_('email'),
        unique=True,
        db_index=True
    )
    bio = models.TextField(
        verbose_name=_('bio'),
        blank=True
    )
    avatar = models.ImageField(
        verbose_name=_('avatar'),
        upload_to='avatars/',
        blank=True,
        null=True
    )
    birth_date = models.DateField(
        verbose_name=_('birth date'),
        null=True,
        blank=True
    )
    is_verified = models.BooleanField(
        verbose_name=_('is verified'),
        default=False,
        db_index=True
    )

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']

    class Meta:
        verbose_name = _('user')
        verbose_name_plural = _('users')
        ordering = ['username']

    def __str__(self):
        return self.get_full_name() or self.username

    def get_absolute_url(self):
        from django.urls import reverse
        return reverse('user-detail', kwargs={'username': self.username})


class UserProfile(TimeStampedModel, SEOFieldsMixin):
    """Extended user profile with additional information."""

    user = models.OneToOneField(
        User,
        on_delete=models.CASCADE,
        related_name='profile',
        verbose_name=_('user')
    )
    phone = models.CharField(
        verbose_name=_('phone'),
        max_length=20,
        blank=True
    )
    address = models.TextField(
        verbose_name=_('address'),
        blank=True
    )
    city = models.CharField(
        verbose_name=_('city'),
        max_length=100,
        blank=True
    )
    country = models.CharField(
        verbose_name=_('country'),
        max_length=100,
        blank=True
    )
    postal_code = models.CharField(
        verbose_name=_('postal code'),
        max_length=20,
        blank=True
    )

    class Meta:
        verbose_name = _('user profile')
        verbose_name_plural = _('user profiles')

    def __str__(self):
        return f"{self.user.username}'s profile"
```

### models/post.py - Post Models

```python
# myapp/models/post.py

from django.db import models
from django.urls import reverse
from django.utils.translation import gettext_lazy as _
from django.contrib.auth import get_user_model

from .base import BaseModel, TitleSlugModel, PublishableModel
from .mixins import MetadataMixin, SEOFieldsMixin

User = get_user_model()


class Category(BaseModel, TitleSlugModel):
    """Blog post categories."""

    description = models.TextField(
        verbose_name=_('description'),
        blank=True
    )
    parent = models.ForeignKey(
        'self',
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='children',
        verbose_name=_('parent category')
    )
    order = models.PositiveIntegerField(
        verbose_name=_('order'),
        default=0
    )
    is_active = models.BooleanField(
        verbose_name=_('is active'),
        default=True,
        db_index=True
    )

    class Meta:
        verbose_name = _('category')
        verbose_name_plural = _('categories')
        ordering = ['order', 'title']

    def get_absolute_url(self):
        return reverse('category-detail', kwargs={'slug': self.slug})


class PostQuerySet(models.QuerySet):
    """Custom queryset for Post model."""

    def published(self):
        """Return only published posts."""
        return self.filter(status=PublishableModel.Status.PUBLISHED)

    def featured(self):
        """Return featured posts."""
        return self.filter(is_featured=True)

    def recent(self, days=7):
        """Return posts from recent days."""
        from django.utils import timezone
        cutoff = timezone.now() - timezone.timedelta(days=days)
        return self.filter(published_at__gte=cutoff)

    def by_category(self, category_slug):
        """Filter posts by category."""
        return self.filter(category__slug=category_slug)


class Post(BaseModel, TitleSlugModel, PublishableModel, MetadataMixin, SEOFieldsMixin):
    """Blog posts with full publishing workflow."""

    # Content fields
    excerpt = models.CharField(
        verbose_name=_('excerpt'),
        max_length=300,
        blank=True,
        help_text=_('Short description for listings')
    )
    content = models.TextField(
        verbose_name=_('content'),
    )

    # Relationships
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='posts',
        verbose_name=_('author')
    )
    category = models.ForeignKey(
        Category,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='posts',
        verbose_name=_('category')
    )
    tags = models.ManyToManyField(
        'Tag',
        blank=True,
        related_name='posts',
        verbose_name=_('tags')
    )

    # Media
    featured_image = models.ImageField(
        verbose_name=_('featured image'),
        upload_to='posts/images/',
        blank=True,
        null=True
    )
    og_image = models.ImageField(
        verbose_name=_('Open Graph image'),
        upload_to='posts/og/',
        blank=True,
        null=True
    )

    # Engagement
    is_featured = models.BooleanField(
        verbose_name=_('is featured'),
        default=False,
        db_index=True
    )
    allow_comments = models.BooleanField(
        verbose_name=_('allow comments'),
        default=True
    )
    views_count = models.PositiveIntegerField(
        verbose_name=_('views count'),
        default=0,
        editable=False
    )

    # Custom manager with QuerySet methods
    objects = PostQuerySet.as_manager()

    class Meta:
        verbose_name = _('post')
        verbose_name_plural = _('posts')
        ordering = ['-published_at', '-created_at']
        indexes = [
            models.Index(fields=['slug']),
            models.Index(fields=['-published_at', 'status']),
            models.Index(fields=['author', 'status']),
        ]
        constraints = [
            models.CheckConstraint(
                check=models.Q(status=PublishableModel.Status.PUBLISHED) & models.Q(published_at__isnull=False)
                    | models.Q(status__in=[PublishableModel.Status.DRAFT, PublishableModel.Status.ARCHIVED]),
                name='check_published_has_date'
            )
        ]

    def __str__(self):
        return self.title

    def get_absolute_url(self):
        return reverse('post-detail', kwargs={'slug': self.slug})

    def increment_views(self):
        """Increment the view count."""
        self.views_count += 1
        self.save(update_fields=['views_count'])

    @property
    def reading_time(self):
        """Estimate reading time based on word count."""
        word_count = len(self.content.split())
        return max(1, word_count // 200)

    def save(self, *args, **kwargs):
        """Set published_at when status changes to published."""
        from django.utils import timezone
        if self.status == PublishableModel.Status.PUBLISHED and not self.published_at:
            self.published_at = timezone.now()
        super().save(*args, **kwargs)
```

### models/comment.py - Comment Models

```python
# myapp/models/comment.py

from django.db import models
from django.utils.translation import gettext_lazy as _

from .base import BaseModel
from .post import Post


class Comment(BaseModel):
    """Comments on blog posts with threaded replies."""

    post = models.ForeignKey(
        Post,
        on_delete=models.CASCADE,
        related_name='comments',
        verbose_name=_('post')
    )
    parent = models.ForeignKey(
        'self',
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='replies',
        verbose_name=_('parent comment')
    )

    # For non-authenticated users
    author_name = models.CharField(
        verbose_name=_('author name'),
        max_length=100
    )
    author_email = models.EmailField(
        verbose_name=_('author email')
    )
    author_ip = models.GenericIPAddressField(
        verbose_name=_('author IP'),
        null=True,
        blank=True
    )

    content = models.TextField(
        verbose_name=_('content')
    )

    is_approved = models.BooleanField(
        verbose_name=_('is approved'),
        default=False,
        db_index=True
    )
    spam_score = models.FloatField(
        verbose_name=_('spam score'),
        default=0.0,
        help_text=_('Higher score indicates more likely spam')
    )

    class Meta:
        verbose_name = _('comment')
        verbose_name_plural = _('comments')
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['post', 'is_approved']),
            models.Index(fields=['parent', 'created_at']),
        ]

    def __str__(self):
        return f"{self.author_name} on {self.post.title[:30]}..."

    @property
    def is_reply(self):
        """Check if this is a reply to another comment."""
        return self.parent is not None

    def get_replies(self):
        """Get all approved replies to this comment."""
        return self.replies.filter(is_approved=True).order_by('created_at')

    def save(self, *args, **kwargs):
        # Auto-approve users with low spam score
        if not self.pk and self.spam_score < 0.3:
            self.is_approved = True
        super().save(*args, **kwargs)
```

### models/tag.py - Tag Models

```python
# myapp/models/tag.py

from django.db import models
from django.utils.translation import gettext_lazy as _
from django.urls import reverse

from .base import BaseModel, TitleSlugModel


class Tag(BaseModel, TitleSlugModel):
    """Blog post tags for categorization."""

    description = models.TextField(
        verbose_name=_('description'),
        blank=True
    )
    color = models.CharField(
        verbose_name=_('color'),
        max_length=7,
        default='#007bff',
        help_text=_('Hex color code')
    )
    is_featured = models.BooleanField(
        verbose_name=_('is featured'),
        default=False,
        db_index=True
    )

    class Meta:
        verbose_name = _('tag')
        verbose_name_plural = _('tags')
        ordering = ['name']

    def get_absolute_url(self):
        return reverse('tag-detail', kwargs={'slug': self.slug})

    @property
    def post_count(self):
        """Return the number of posts with this tag."""
        return self.posts.count()
```

### models/querysets.py - Custom QuerySets

```python
# myapp/models/querysets.py

from django.db import models
from django.utils import timezone


class ActiveQuerySet(models.QuerySet):
    """QuerySet for filtering active (non-deleted) objects."""

    def active(self):
        """Return only active objects."""
        return self.filter(is_deleted=False)


class PublishableQuerySet(models.QuerySet):
    """QuerySet for models with publish status."""

    def published(self):
        """Return only published items."""
        return self.filter(status='published')

    def scheduled(self):
        """Return scheduled items."""
        return self.filter(status='scheduled', published_at__gt=timezone.now())

    def draft(self):
        """Return draft items."""
        return self.filter(status='draft')
```

## Admin Per-Class Structure

Similarly, organize admin by class for large projects:

```
myapp/
└── admin/
    ├── __init__.py
    ├── base.py
    ├── user.py
    ├── post.py
    ├── comment.py
    └── tag.py
```

### admin/__init__.py

```python
# myapp/admin/__init__.py

from .user import UserAdmin, UserProfileAdmin
from .post import PostAdmin, CategoryAdmin
from .comment import CommentAdmin
from .tag import TagAdmin

# This will automatically register all admins
# when the app is loaded
```

## Forms Per-Class Structure

Organize forms similarly:

```
myapp/
└── forms/
    ├── __init__.py
    ├── user.py
    ├── post.py
    ├── comment.py
    └── tag.py
```

### forms/__init__.py

```python
# myapp/forms/__init__.py

from .user import UserForm, UserProfileForm
from .post import PostForm, CategoryForm
from .comment import CommentForm
from .tag import TagForm
```

## Serializers Per-Class Structure (DRF)

For Django REST Framework:

```
myapp/
└── serializers/
    ├── __init__.py
    ├── user.py
    ├── post.py
    ├── comment.py
    └── tag.py
```

### serializers/__init__.py

```python
# myapp/serializers/__init__.py

from .user import UserSerializer, UserProfileSerializer
from .post import PostSerializer, CategorySerializer
from .comment import CommentSerializer
from .tag import TagSerializer
```

## Views Per-Class Structure

Organize views by related functionality:

```
myapp/
└── views/
    ├── __init__.py
    ├── base.py
    ├── user.py
    ├── post.py
    ├── comment.py
    └── tag.py
```

### views/__init__.py

```python
# myapp/views/__init__.py

from .user import (
    UserDetailView,
    UserUpdateView,
    UserProfileUpdateView,
)
from .post import (
    PostListView,
    PostDetailView,
    PostCreateView,
    PostUpdateView,
    PostDeleteView,
)
from .comment import CommentCreateView
from .tag import TagDetailView
```

## Benefits of Per-Class Structure

1. **Easier Navigation** - Find specific classes quickly
2. **Reduced Conflicts** - Less merge conflicts in version control
3. **Better Organization** - Logical grouping of related functionality
4. **Improved Readability** - Smaller files are easier to understand
5. **Targeted Testing** - Test files can mirror the structure

## Migration Considerations

When using per-class models structure, Django still sees them as a single app:

```python
# migrations still work normally
python manage.py makemigrations myapp
python manage.py migrate myapp
```

The `__init__.py` in the models directory ensures all models are properly registered with Django's model system.
