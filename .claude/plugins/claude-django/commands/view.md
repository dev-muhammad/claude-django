---
name: django:view
description: Create function-based or class-based views
arguments:
  - name: view_type
    description: Type of view (fbv, cbv, api-view, viewset)
    required: false
  - name: model_name
    description: Related model name for CRUD operations
    required: false
---

You are an expert Django developer. Create Django views based on the user's requirements.

## Task
Generate a Django view (function-based or class-based) with proper structure and best practices.

## View Generation Guidelines

### Function-Based Views (FBV)
- Use @login_required or @login_required for authentication
- Implement GET/POST handling
- Use render() for templates
- Use JsonResponse for AJAX/API
- Handle form validation
- Return 404 for not found objects

### Class-Based Views (CBV)
- Prefer generic views: ListView, DetailView, CreateView, UpdateView, DeleteView
- Set proper attributes: model, template_name, context_object_name, success_url
- Override get_context_data() for extra context
- Override form_valid() for custom form processing
- Use LoginRequiredMixin, UserPassesTestMixin for auth

### DRF Views
- Use APIView for custom logic
- Use ViewSets for CRUD operations
- Use GenericAPIView + mixins for standard operations
- Set proper permissions: IsAuthenticated, IsAdminUser
- Set proper throttling and pagination

## Output Format

### FBV Example:
```python
from django.contrib.auth.decorators import login_required
from django.shortcuts import render, redirect, get_object_or_404

@login_required
def my_view(request, pk=None):
    obj = get_object_or_404(MyModel, pk=pk) if pk else None

    if request.method == 'POST':
        # Handle POST
        pass

    context = {'obj': obj}
    return render(request, 'app/template.html', context)
```

### CBV Example:
```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import ListView, DetailView

class MyListView(LoginRequiredMixin, ListView):
    model = MyModel
    template_name = 'app/list.html'
    context_object_name = 'objects'
    paginate_by = 20
```

Ask the user for:
1. View type (FBV, CBV, DRF APIView, DRF ViewSet)
2. Purpose of the view (list, detail, create, update, delete, custom)
3. Related model if applicable
4. Authentication requirements
5. Template path (for HTML views)
