---
name: migrate-to-async
description: Migrate existing Django views to async (Django 5.1+)
---

# Django Async Migration Skill

You are an expert Django async developer. Guide the user through migrating synchronous views to async.

## Django 5.1+ Async Support

Django 5.1 added async support for views, allowing:
- Non-blocking I/O operations
- Better performance under load
- Efficient handling of slow external APIs
- Concurrent database queries

## Interactive Migration Process

### Step 1: Assessment
Ask the user:
- Which views do you want to migrate?
- Do these views make slow external API calls?
- Do they perform multiple database queries?
- Are you using Django ORM or a database that supports async (PostgreSQL, MySQL)?
- What async database driver are you using? (asyncpg, aiomysql, etc.)

### Step 2: Prerequisites

#### Database Configuration
```python
# settings/base.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',  # or postgresql_async (Django 6.0)
        'NAME': 'mydb',
        'USER': 'user',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'OPTIONS': {
            # Django 6.0 native async
            'async_mode': True,
        },
    }
}

# For Django 5.x using asyncpg
# DATABASES['default']['ENGINE'] = 'django.db.backends.postgresql'
# Requires: pip install asyncpg
```

#### ASGI Application
```python
# asgi.py (Django 6.0+ has built-in async)
import os
from django.core.asgi import get_asgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')
application = get_asgi_application()
```

### Step 3: View Migration Patterns

#### Function-Based View (FBV)

**Before (Sync):**
```python
from django.shortcuts import render, get_object_or_404
from django.http import HttpResponse
import requests

def product_detail(request, pk):
    product = get_object_or_404(Product, pk=pk)
    # Blocking API call
    response = requests.get(f'https://api.example.com/stock/{pk}')
    stock_data = response.json()
    return render(request, 'product.html', {'product': product, 'stock': stock_data})
```

**After (Async):**
```python
from django.shortcuts import render
from django.http import HttpResponse
from asgiref.sync import sync_to_async
import aiohttp

async def product_detail(request, pk):
    # Async ORM query (Django 6.0)
    product = await Product.objects.aget(pk=pk)

    # Async HTTP request
    async with aiohttp.ClientSession() as session:
        async with session.get(f'https://api.example.com/stock/{pk}') as response:
            stock_data = await response.json()

    return render(request, 'product.html', {'product': product, 'stock': stock_data})
```

#### Class-Based View (CBV)

**Before (Sync):**
```python
from django.views.generic import ListView
from django.db.models import Q

class ProductListView(ListView):
    model = Product
    paginate_by = 20

    def get_queryset(self):
        queryset = super().get_queryset()
        if search := self.request.GET.get('search'):
            queryset = queryset.filter(Q(name__icontains=search))
        return queryset
```

**After (Async):**
```python
from django.views.generic import ListView
from django.db.models import Q

class ProductListView(ListView):
    model = Product
    paginate_by = 20
    # Django 6.0 CBVs support async
    querysets_async = True  # Enable async queryset evaluation

    async def get_queryset(self):
        queryset = await super().aget_queryset()
        if search := self.request.GET.get('search'):
            queryset = queryset.filter(Q(name__icontains=search))
        return queryset

    async def aget(self, request, *args, **kwargs):
        self.object_list = await self.get_queryset()
        context = self.get_context_data()
        return self.render_to_response(context)
```

### Step 4: ORM Migration Guide

#### Reading Objects
```python
# Sync
product = Product.objects.get(pk=1)
products = list(Product.objects.all())

# Async (Django 6.0)
product = await Product.objects.aget(pk=1)
products = [p async for p in Product.objects.all()]
```

#### Creating Objects
```python
# Sync
product = Product.objects.create(name='Widget')

# Async
product = await Product.objects.acreate(name='Widget')
```

#### Filtering
```python
# Sync
products = Product.objects.filter(is_active=True)

# Async - queryset creation is synchronous
queryset = Product.objects.filter(is_active=True)
products = [p async for p in queryset]
```

#### Foreign Keys and Related Objects
```python
# Sync - N+1 problem
for order in Order.objects.all():
    print(order.customer.name)  # Query per order

# Async - still N+1, but non-blocking
async for order in Order.objects.all():
    customer = await order.acustomer  # One query per order
    print(customer.name)

# Better - use select_related (works with async)
async for order in Order.objects.select_related('customer').all():
    print(order.customer.name)  # No extra query
```

### Step 5: Middleware Migration

**Before:**
```python
class CustomMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        # Do something
        response = self.get_response(request)
        # Do something with response
        return response
```

**After:**
```python
class CustomMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    async def __call__(self, request):
        # Do something async
        response = await self.get_response(request)
        # Do something async with response
        return response
```

### Step 6: Common Migration Patterns

#### External API Calls
```python
# Before
import requests

def sync_view(request):
    data = requests.get('https://api.example.com/data').json()
    return JsonResponse(data)

# After
import aiohttp

async def async_view(request):
    async with aiohttp.ClientSession() as session:
        async with session.get('https://api.example.com/data') as resp:
            data = await resp.json()
    return JsonResponse(data)
```

#### Multiple External Calls (Concurrent)
```python
import asyncio
import aiohttp

async def get_user_data(user_id, session):
    async with session.get(f'https://api.example.com/users/{user_id}') as resp:
        return await resp.json()

async def get_user_orders(user_id, session):
    async with session.get(f'https://api.example.com/users/{user_id}/orders') as resp:
        return await resp.json()

async def profile_view(request, user_id):
    async with aiohttp.ClientSession() as session:
        # Run both concurrently
        user_data, orders = await asyncio.gather(
            get_user_data(user_id, session),
            get_user_orders(user_id, session),
        )
    return JsonResponse({'user': user_data, 'orders': orders})
```

#### Email Sending
```python
# Before
from django.core.mail import send_mail

def send_view(request):
    send_mail('Subject', 'Body', 'from@example.com', ['to@example.com'])
    return HttpResponse('Sent')

# After
from django.core.mail import send_mail
from asgiref.sync import sync_to_async

async def send_view(request):
    # Email sending still sync, wrap it
    await sync_to_async(send_mail)('Subject', 'Body', 'from@example.com', ['to@example.com'])
    return HttpResponse('Sent')
```

### Step 7: Testing Async Views

```python
from django.test import AsyncClient
import pytest

@pytest.mark.asyncio
async def test_async_view():
    client = AsyncClient()
    response = await client.get('/async-url/')
    assert response.status_code == 200
```

### Step 8: Deployment

**Daphne (Recommended):**
```bash
pip install daphne
daphne myproject.asgi:application -b 0.0.0.0 -p 8000
```

**Uvicorn:**
```bash
pip install uvicorn
uvicorn myproject.asgi:application --host 0.0.0.0 --port 8000
```

## Common Pitfalls

1. **Mixing sync and async ORM** - Don't mix in same view
2. **Forgetting `await`** - Easy to miss with Django 6.0 hybrid approach
3. **Blocking operations** - Use sync_to_async for blocking libraries
4. **Database connections** - Async uses different connection pool
5. **Middleware** - All middleware in chain must be async

## Migration Checklist

- [ ] Switch database to async-capable backend
- [ ] Update ASGI configuration
- [ ] Install async dependencies (aiohttp, etc.)
- [ ] Migrate views one at a time
- [ ] Test async views thoroughly
- [ ] Update deployment to use ASGI server
- [ ] Monitor performance improvements

Ask the user:
1. **Which views would benefit most from async?** (API calls, slow queries, etc.)
2. **What async database driver are you using?**
3. **What's your current deployment setup?** (Gunicorn needs ASGI server)
