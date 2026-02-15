---
name: migrate-to-composite-pk
description: Migrate models to use composite primary keys (Django 5.2+)
---

# Django Composite Primary Keys Migration Skill

You are an expert Django 5.2+ developer. Guide the user through migrating models to use composite primary keys.

## Django 5.2+ Composite PK Support

Django 5.2 introduced native **composite primary key** support, eliminating the need for workarounds like:
- Multiple unique constraints
- Custom save() methods
- Third-party packages (django-composite-foreignkey)

## Interactive Migration Process

### Step 1: Assessment
Ask the user:
- Which models need composite primary keys?
- What fields should compose the primary key?
- Are there existing ForeignKeys pointing to this model?
- What's the current database state? (new project vs production data)

### Step 2: Understand Composite PKs

**Traditional (Single Primary Key):**
```python
class OrderItem(models.Model):
    id = models.AutoField(primary_key=True)  # Surrogate key
    order = models.ForeignKey('Order', on_delete=models.CASCADE)
    product = models.ForeignKey('Product', on_delete=models.CASCADE)
    quantity = models.IntegerField()
```

**With Composite PK (Django 5.2+):**
```python
class OrderItem(models.Model):
    order = models.ForeignKey('Order', on_delete=models.CASCADE, primary_key=True)
    product = models.ForeignKey('Product', on_delete=models.CASCADE, primary_key=True)
    quantity = models.IntegerField()

    class Meta:
        unique_together = [['order', 'product']]  # Enforced by PK now
        # OR use constraints (preferred):
        constraints = [
            models.UniqueConstraint(fields=['order', 'product'], name='unique_order_product')
        ]
```

### Step 3: Defining Composite Primary Keys

**Method 1: Multiple primary_key=True fields**
```python
class Membership(models.Model):
    organization = models.ForeignKey('Organization', on_delete=models.CASCADE, primary_key=True)
    user = models.ForeignKey('User', on_delete=models.CASCADE, primary_key=True)
    role = models.CharField(max_length=50)
    joined_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.user} in {self.organization}"
```

**Method 2: Using Meta.primary_key (Django 5.2+)**
```python
class TemperatureReading(models.Model):
    station = models.ForeignKey('WeatherStation', on_delete=models.CASCADE)
    timestamp = models.DateTimeField()
    temperature = models.FloatField()

    class Meta:
        primary_key = models.composite_primary_key('station', 'timestamp')
        # Or for Django 5.2:
        # primary_key = ('station', 'timestamp')  # Simplified syntax
```

### Step 4: ForeignKeys with Composite PKs

**Pointing to a model with composite PK:**
```python
class MembershipLog(models.Model):
    # Composite PK reference
    membership = models.ForeignKey(
        'Membership',
        on_delete=models.CASCADE,
        # Django 5.2+ handles composite PK automatically
    )
    action = models.CharField(max_length=100)
    created_at = models.DateTimeField(auto_now_add=True)
```

### Step 5: Working with Composite PK Objects

#### Retrieval
```python
# Get by composite key (Django 5.2+)
membership = Membership.objects.get(
    organization_id=org_id,
    user_id=user_id
)

# Or using the natural key
membership = Membership.objects.get(
    organization=org,
    user=user
)
```

#### URL Patterns
```python
# urls.py - Composite PK in URL
path('memberships/<int:organization_id>/<int:user_id>/', views.membership_detail)

# views.py
def membership_detail(request, organization_id, user_id):
    membership = get_object_or_404(Membership, organization_id=organization_id, user_id=user_id)
    return render(request, 'membership_detail.html', {'membership': membership})
```

#### get_absolute_url()
```python
class Membership(models.Model):
    # ... fields ...

    def get_absolute_url(self):
        from django.urls import reverse
        return reverse('membership-detail', kwargs={
            'organization_id': self.organization_id,
            'user_id': self.user_id,
        })
```

### Step 6: Migration Strategy

#### For New Projects
Simply define composite PKs from the start:
```python
class OrderItem(models.Model):
    order = models.ForeignKey('Order', on_delete=models.CASCADE, primary_key=True)
    product = models.ForeignKey('Product', on_delete=models.CASCADE, primary_key=True)
    # ... other fields
```

#### For Existing Projects (Without Data)
```python
# 1. Add composite PK to new model
class NewModel(models.Model):
    field1 = models.ForeignKey(..., primary_key=True)
    field2 = models.ForeignKey(..., primary_key=True)
```

#### For Existing Projects (With Data) - Advanced

This requires careful migration:

```python
# Step 1: Create new model with composite PK
class OrderItemNew(models.Model):
    order = models.ForeignKey('Order', on_delete=models.CASCADE, primary_key=True)
    product = models.ForeignKey('Product', on_delete=models.CASCADE, primary_key=True)
    quantity = models.IntegerField()

# Step 2: Migrate data
python manage.py makemigrations
python manage.py migrate

# Step 3: Copy data from old to new
python manage.py shell
>>> for item in OrderItem.objects.all():
>>>     OrderItemNew.objects.create(order=item.order, product=item.product, quantity=item.quantity)

# Step 4: Rename models
# Remove old model, rename new model in next migration
```

### Step 7: Admin Configuration

```python
@admin.register(Membership)
class MembershipAdmin(admin.ModelAdmin):
    list_display = ['organization', 'user', 'role', 'joined_at']
    list_filter = ['role', 'joined_at']
    search_fields = ['organization__name', 'user__email']

    # Composite key in admin URLs handled automatically
```

### Step 8: Forms with Composite PKs

```python
class MembershipForm(forms.ModelForm):
    class Meta:
        model = Membership
        fields = ['role']
        # organization and user in PK, may be read-only or hidden
```

### Step 9: Serializers (DRF)

```python
from rest_framework import serializers

class MembershipSerializer(serializers.ModelSerializer):
    # Composite PK fields
    organization_id = serializers.IntegerField()
    user_id = serializers.IntegerField()

    class Meta:
        model = Membership
        fields = ['organization_id', 'user_id', 'role', 'joined_at']
```

## Composite PK Best Practices

1. **Use for natural junctions** - ManyToMany through models, junction tables
2. **Consider surrogate keys** - Sometimes an auto-increment ID is simpler
3. **Document your choice** - Comment why composite PK was chosen
4. **Update related code** - URLs, forms, serializers all affected

## Common Use Cases

### Junction Table (ManyToMany through)
```python
class StudentCourse(models.Model):
    student = models.ForeignKey('Student', on_delete=models.CASCADE, primary_key=True)
    course = models.ForeignKey('Course', on_delete=models.CASCADE, primary_key=True)
    enrolled_at = models.DateTimeField(auto_now_add=True)
```

### Multi-Tenant Data
```python
class TenantData(models.Model):
    tenant = models.ForeignKey('Tenant', on_delete=models.CASCADE, primary_key=True)
    data_id = models.IntegerField(primary_key=True)
    value = models.CharField(max_length=255)
```

### Time-Series Data
```python
class SensorReading(models.Model):
    sensor = models.ForeignKey('Sensor', on_delete=models.CASCADE, primary_key=True)
    timestamp = models.DateTimeField(primary_key=True)
    value = models.FloatField()
```

## Ask the user:

1. **Which models need composite primary keys?**
2. **What fields compose the key?**
3. **Are there existing ForeignKeys to this model?**
4. **What's your current data state?** (new vs existing data)
5. **Do you need help with the migration strategy?**
