---
name: setup-auth
description: Configure custom user model and authentication system
---

# Django Authentication Setup Skill

You are an expert Django developer. Guide the user through setting up a custom authentication system.

## This skill helps you:

1. **Create a custom User model** (recommended for new projects)
2. **Set up authentication backends**
3. **Configure user registration**
4. **Add social authentication** (optional)
5. **Set up password reset flow**
6. **Configure user permissions**
7. **Add user profile** (if needed)
8. **Set up authentication for REST API** (if using DRF)

## Interactive Process

Ask the user questions progressively:

### Step 1: User Model Requirements
- Do you need a custom User model? (HIGHLY RECOMMENDED for new projects)
- What additional fields do you need?
  - Phone number?
  - Date of birth?
  - Profile picture?
  - Bio/about me?
- Do you need separate user profiles?

### Step 2: Authentication Type
- Is this for a traditional web app? (session-based)
- Is this for a REST API? (token/JWT)
- Is this for a SPA/mobile app? (JWT recommended)
- Do you need social authentication? (Google, Facebook, etc.)

### Step 3: Registration & Password Reset
- Do you need user registration?
- Email verification required?
- Do you need password reset via email?
- Do you need custom password validation?

### Step 4: Permissions
- Do you need custom permissions?
- Do you need groups/roles?
- Do you need object-level permissions?

## Custom User Model Setup

### 1. Create users app
```bash
python manage.py startapp users
```

### 2. Create custom user model (users/models.py)

```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    email = models.EmailField(unique=True)
    first_name = models.CharField(max_length=150)
    last_name = models.CharField(max_length=150)
    phone = models.CharField(max_length=20, blank=True)
    date_of_birth = models.DateField(blank=True, null=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True)
    bio = models.TextField(blank=True)
    is_verified = models.BooleanField(default=False)

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username', 'first_name', 'last_name']

    class Meta:
        verbose_name = 'user'
        verbose_name_plural = 'users'

    def __str__(self):
        return self.email

    def get_full_name(self):
        return f"{self.first_name} {self.last_name}".strip()
```

### 3. Configure settings (settings/base.py)

```python
AUTH_USER_MODEL = 'users.User'

INSTALLED_APPS = [
    # ...
    'django.contrib.auth',
    'django.contrib.sessions',
    'users',  # Must be after django.contrib.auth
]
```

### 4. Create admin (users/admin.py)

```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from .models import User

@admin.register(User)
class UserAdmin(BaseUserAdmin):
    list_display = ['email', 'username', 'first_name', 'last_name', 'is_verified', 'is_staff']
    list_filter = ['is_staff', 'is_superuser', 'is_verified', 'groups']
    search_fields = ['email', 'username', 'first_name', 'last_name']
    ordering = ['email']

    fieldsets = (
        (None, {'fields': ('email', 'username', 'password')}),
        ('Personal info', {'fields': ('first_name', 'last_name', 'phone', 'date_of_birth', 'avatar', 'bio')}),
        ('Verification', {'fields': ('is_verified',)}),
        ('Permissions', {'fields': ('is_active', 'is_staff', 'is_superuser', 'groups', 'user_permissions')}),
        ('Important dates', {'fields': ('last_login', 'date_joined')}),
    )

    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': ('email', 'username', 'first_name', 'last_name', 'password1', 'password2'),
        }),
    )
```

### 5. Create forms (users/forms.py)

```python
from django import forms
from django.contrib.auth.forms import UserCreationForm
from .models import User

class CustomUserCreationForm(UserCreationForm):
    email = forms.EmailField(required=True)
    first_name = forms.CharField(required=True)
    last_name = forms.CharField(required=True)

    class Meta:
        model = User
        fields = ('email', 'username', 'first_name', 'last_name')

    def save(self, commit=True):
        user = super().save(commit=False)
        user.email = self.cleaned_data['email']
        user.first_name = self.cleaned_data['first_name']
        user.last_name = self.cleaned_data['last_name']
        if commit:
            user.save()
        return user
```

### 6. Create views (users/views.py)

```python
from django.contrib.auth.views import LoginView, LogoutView
from django.contrib.auth.decorators import login_required
from django.shortcuts import render, redirect
from django.urls import reverse_lazy
from django.views.generic import CreateView
from .forms import CustomUserCreationForm

class SignUpView(CreateView):
    form_class = CustomUserCreationForm
    template_name = 'users/signup.html'
    success_url = reverse_lazy('login')

class CustomLoginView(LoginView):
    template_name = 'users/login.html'
    redirect_authenticated_user = True

class CustomLogoutView(LogoutView):
    next_page = 'login'

@login_required
def profile_view(request):
    return render(request, 'users/profile.html')
```

## For REST API Authentication

If using Django REST Framework, add JWT authentication:

```python
# settings/base.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
}

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
}
```

## Social Authentication (Optional)

Using django-allauth:

```bash
pip install django-allauth
```

```python
# settings/base.py
INSTALLED_APPS = [
    # ...
    'django.contrib.sites',
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'allauth.socialaccount.providers.google',
]

AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
    'allauth.account.auth_backends.AuthenticationBackend',
]

SITE_ID = 1
```

## Migration Warning

**IMPORTANT**: Set up custom user model BEFORE your first migration. If you've already run migrations, you'll need to:
1. Backup your database
2. Drop all tables
3. Delete migration files (keep __init__.py)
4. Run migrations again

Or follow Django's official migration guide for swapping user models mid-project.
