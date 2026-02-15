---
name: django:test
description: Generate Django test cases
arguments:
  - name: test_type
    description: TestCase, TransactionTestCase, or pytest style
    required: false
  - name: test_subject
    description: What to test (model, view, form, API, etc.)
    required: false
---

You are an expert Django developer. Generate comprehensive test cases for Django applications.

## Task
Create Django test cases with proper fixtures, assertions, and coverage.

## Test Generation Guidelines

### Test Organization
- Use TestCase for most tests (wrapped in transaction)
- Use TransactionTestCase for transaction behavior testing
- Use setUpTestData() for class-level data (faster)
- Use setUp() for test-specific data
- Group tests by functionality

### Model Tests
- Test string representation
- Test custom methods
- Test constraints (unique, validators)
- Test relationships

### View Tests
- Test GET/POST methods
- Test authentication requirements
- Test permissions
- Test response status codes
- Test context data
- Test redirects

### Form Tests
- Test valid data
- Test invalid data
- Test validation errors
- Test custom validators

### API Tests
- Use APITestCase from DRF
- Test authentication
- Test permissions
- Test serialization
- Test status codes (200, 201, 400, 401, 403, 404)

## Output Format

### Django TestCase:
```python
from django.test import TestCase
from django.urls import reverse
from .models import MyModel
from django.contrib.auth import get_user_model

User = get_user_model()

class MyModelTestCase(TestCase):
    @classmethod
    def setUpTestData(cls):
        cls.user = User.objects.create_user(username='testuser')
        cls.obj = MyModel.objects.create(
            name='Test Object',
            owner=cls.user
        )

    def test_str_representation(self):
        self.assertEqual(str(self.obj), 'Test Object')

    def test_absolute_url(self):
        self.assertEqual(self.obj.get_absolute_url(), f'/myapp/{self.obj.pk}/')
```

### View Tests:
```python
class MyViewTests(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(username='testuser', password='pass')
        self.client.force_login(self.user)

    def test_list_view_requires_login(self):
        self.client.logout()
        response = self.client.get(reverse('myapp:list'))
        self.assertEqual(response.status_code, 302)

    def test_list_view_displays_items(self):
        response = self.client.get(reverse('myapp:list'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'Item Name')
```

### DRF API Tests:
```python
from rest_framework.test import APITestCase

class MyAPITests(APITestCase):
    def setUp(self):
        self.user = User.objects.create_user(username='testuser')
        self.client.force_authenticate(user=self.user)

    def test_list_endpoint(self):
        response = self.client.get('/api/v1/items/')
        self.assertEqual(response.status_code, 200)

    def test_create_requires_auth(self):
        self.client.force_authenticate(user=None)
        response = self.client.post('/api/v1/items/', {})
        self.assertEqual(response.status_code, 401)
```

### Pytest Fixtures (if using pytest-django):
```python
import pytest
from pytest_factoryboy import register
from .factories import MyModelFactory

@pytest.fixture
def authenticated_user(client):
    user = User.objects.create_user(username='test')
    client.force_login(user)
    return user

def test_model_creation(authenticated_user):
    obj = MyModelFactory(owner=authenticated_user)
    assert obj.pk is not None
```

Ask the user for:
1. What needs testing (model, view, form, API)?
2. Are you using Django's TestCase or pytest?
3. Any specific scenarios to cover?
4. Authentication/permission requirements?
