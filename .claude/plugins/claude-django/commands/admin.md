---
name: django:admin
description: Configure Django admin registration
arguments:
  - name: model_name
    description: The model name to register in admin
    required: true
  - name: features
    description: Admin features (list_display, list_filter, search_fields, etc.)
    required: false
---

You are an expert Django developer. Generate Django admin configuration with best practices.

## Task
Create a Django admin configuration for the specified model with useful features.

## Admin Generation Guidelines

### Basic Registration
- Use @admin.register() decorator or admin.site.register()
- Inherit from ModelAdmin
- Set list_display for visible columns
- Add list_filter for sidebar filters
- Add search_fields for search functionality

### Display Options
- list_display - Fields to show in list view
- list_display_links - Which fields are clickable
- list_filter - Fields for sidebar filtering
- search_fields - Fields to search
- date_hierarchy - Date-based navigation
- ordering - Default sort order
- list_per_page - Items per page

### Inline Configuration
- Use TabularInline for related models (compact)
- Use StackedInline for related models (expanded)
- Set extra to control number of blank forms

### Actions
- Create custom admin actions
- Use @admin.action() decorator
- Apply to selected objects

### Fieldsets
- Group related fields
- Use 'collapse' for less important sections
- Separate readonly from editable fields

### Permissions
- Use has_add_permission, has_change_permission, has_delete_permission
- Override get_queryset() to filter by user

## Output Format
```python
from django.contrib import admin
from .models import MyModel, RelatedModel

@admin.register(MyModel)
class MyModelAdmin(admin.ModelAdmin):
    list_display = ['name', 'created_at', 'is_active']
    list_display_links = ['name']
    list_filter = ['is_active', 'created_at']
    search_fields = ['name', 'description']
    date_hierarchy = 'created_at'
    ordering = ['-created_at']
    list_per_page = 25

    fieldsets = (
        ('Basic Information', {
            'fields': ('name', 'description')
        }),
        ('Settings', {
            'fields': ('is_active', 'options'),
            'classes': ('collapse',)
        }),
    )

    def get_queryset(self, request):
        qs = super().get_queryset(request)
        if not request.user.is_superuser:
            qs = qs.filter(owner=request.user)
        return qs
```

Ask the user for:
1. Model name
2. Fields for list_display
3. Fields that need filtering
4. Fields for search
5. Any inline relationships
6. Permission requirements
