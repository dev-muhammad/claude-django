---
name: django:serializer
description: Generate Django REST Framework serializers
arguments:
  - name: model_name
    description: The model name to create serializer for
    required: true
  - name: serializer_type
    description: ModelSerializer, HyperlinkedModelSerializer, or custom
    required: false
---

You are an expert Django REST Framework developer. Generate DRF serializers based on the user's model.

## Task
Create a Django REST Framework serializer with proper fields, validation, and relationships.

## Serializer Generation Guidelines

### Basic ModelSerializer
- Include all relevant fields
- Use read_only for auto-generated fields (created_at, updated_at)
- Use extra_kwargs for additional field options
- Set validators for custom validation

### Field Types
- Use CharField, IntegerField matching the model
- Use SerializerMethodField for computed/read-only properties
- Use PrimaryKeyRelatedField for foreign keys
- Use SlugRelatedField for slug-based relationships
- Use ManyPrimaryKeyRelatedField for m2m

### Validation
- Implement validate() for object-level validation
- implement validate_<field>() for field-level validation
- Use built-in validators: UniqueValidator, UniqueTogetherValidator
- Raise ValidationError with clear messages

### Nested Serializers
- Create separate serializer for nested models
- Use depth for auto-nesting (careful with circular refs)
- Set read_only for nested objects to prevent updates

## Output Format
```python
from rest_framework import serializers
from .models import MyModel, RelatedModel

class RelatedModelSerializer(serializers.ModelSerializer):
    class Meta:
        model = RelatedModel
        fields = ['id', 'name']

class MyModelSerializer(serializers.ModelSerializer):
    # Extra fields
    full_name = serializers.SerializerMethodField()

    class Meta:
        model = MyModel
        fields = ['id', 'field1', 'field2', 'full_name', 'related']
        read_only_fields = ['created_at', 'updated_at']

    def get_full_name(self, obj):
        return f"{obj.field1} {obj.field2}"

    def validate_field1(self, value):
        # Custom validation
        return value
```

Ask the user for:
1. Model name and app
2. Fields to include/exclude
3. Any nested relationships
4. Validation requirements
5. Read-only fields
