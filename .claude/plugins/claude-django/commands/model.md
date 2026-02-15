---
name: django:model
description: Generate Django models with fields, relationships, composite primary keys (Django 5.2+), and methods
arguments:
  - name: app_label
    description: The Django app label (e.g., 'blog', 'shop')
    required: true
  - name: model_name
    description: The name of the model class (e.g., 'Post', 'Product')
    required: true
  - name: fields
    description: Field definitions (e.g., 'title:CharField, body:TextField')
    required: false
  - name: composite_pk
    description: Use composite primary key (Django 5.2+)
    required: false
---

You are an expert Django developer. Generate a Django model based on the user's requirements with support for Django 5.2+ composite primary keys.

## Task
Create a Django model in the specified app with proper:
- Field definitions with appropriate types and parameters
- String representation (__str__)
- Relevant Meta options (ordering, verbose_name, etc.)
- Relationships (ForeignKey, ManyToMany, etc.) if applicable
- Custom methods if needed
- Composite primary keys (Django 5.2+) if requested

## Model Generation Guidelines

### Primary Key Options (Django 5.2+)

**1. Traditional Single Primary Key (default):**
```python
class Product(models.Model):
    id = models.AutoField(primary_key=True)  # Auto-generated
    name = models.CharField(max_length=100)
```

**2. Composite Primary Key (Django 5.2+):**
```python
class OrderItem(models.Model):
    order = models.ForeignKey('Order', on_delete=models.CASCADE, primary_key=True)
    product = models.ForeignKey('Product', on_delete=models.CASCADE, primary_key=True)
    quantity = models.IntegerField()

    # When using composite PKs, unique_together is automatically enforced
```

**3. Define composite PK via Meta (Django 5.2+):**
```python
class TemperatureReading(models.Model):
    station = models.ForeignKey('WeatherStation', on_delete=models.CASCADE)
    timestamp = models.DateTimeField()
    temperature = models.FloatField()

    class Meta:
        primary_key = models.composite_primary_key('station', 'timestamp')
```

### Field Types
- CharField for short strings
- TextField for long text
- IntegerField, DecimalField for numbers
- DateField, DateTimeField for dates
- EmailField, URLField for validated inputs
- BooleanField for true/false
- ForeignKey for many-to-one
- ManyToManyField for many-to-many

### Always Include
- A descriptive __str__ method
- Meta class with ordering and verbose_name
- Help text for user-facing fields
- get_absolute_url() for detail pages

### Best Practices
- Use related_name for reverse relationships
- Add db_index for frequently queried fields
- Use on_delete appropriately (CASCADE, PROTECT, SET_NULL)
- Add editable=False for auto-generated fields
- Use constraints instead of unique_together (Django 2.2+)
- Use composite PKs for natural junctions (Django 5.2+)

### Django Version Compatibility
- Django 5.2+: Composite primary keys support
- Django 5.0+: Field choices using enumeration
- Django 4.2+: Constraint classes (CheckConstraint, UniqueConstraint)
- Django 3.2+: JSONField on all databases

## Output Format

### Single PK Model (Traditional):
```python
# app_label/models.py
from django.db import models
from django.contrib.auth import get_user_model

User = get_user_model()

class Product(models.Model):
    name = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    description = models.TextField(blank=True)
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-created_at']
        verbose_name = "product"
        verbose_name_plural = "products"
        indexes = [
            models.Index(fields=['slug']),
            models.Index(fields=['-created_at', 'is_active']),
        ]

    def __str__(self):
        return self.name

    def get_absolute_url(self):
        from django.urls import reverse
        return reverse('product-detail', kwargs={'slug': self.slug})
```

### Composite PK Model (Django 5.2+):
```python
# app_label/models.py
from django.db import models

class OrderItem(models.Model):
    # Composite primary key: order + product
    order = models.ForeignKey(
        'Order',
        on_delete=models.CASCADE,
        primary_key=True,
        related_name='items'
    )
    product = models.ForeignKey(
        'Product',
        on_delete=models.CASCADE,
        primary_key=True
    )
    quantity = models.PositiveIntegerField(default=1)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    added_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        verbose_name = "order item"
        verbose_name_plural = "order items"
        # unique_together automatically enforced by composite PK
        ordering = ['order', 'product']

    def __str__(self):
        return f"{self.order} - {self.product} (x{self.quantity})"

    @property
    def total_price(self):
        return self.price * self.quantity
```

### Model with Constraints:
```python
class Event(models.Model):
    name = models.CharField(max_length=200)
    start_time = models.DateTimeField()
    end_time = models.DateTimeField()
    max_attendees = models.PositiveIntegerField()

    class Meta:
        verbose_name = "event"
        verbose_name_plural = "events"
        ordering = ['start_time']
        constraints = [
            models.CheckConstraint(
                check=models.Q(end_time__gt=models.F('start_time')),
                name='check_end_after_start'
            ),
            models.UniqueConstraint(
                fields=['name', 'start_time'],
                name='unique_event_start_time'
            )
        ]

    def __str__(self):
        return self.name
```

Ask the user for:
1. App label
2. Model name
3. Required fields with types
4. Primary key type (auto, single, or composite)?
5. If composite: which fields compose the primary key?
6. Any relationships needed
7. Additional requirements (permissions, constraints, etc.)
