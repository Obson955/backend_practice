# Django Views Blueprint: Complete Step-by-Step Guide

## Table of Contents
1. [Function-Based Views (FBV)](#function-based-views-fbv)
2. [Class-Based Views (CBV)](#class-based-views-cbv)
3. [DRF APIView](#drf-apiview)
4. [DRF ViewSets](#drf-viewsets)
5. [Common Patterns](#common-patterns)

---

## Function-Based Views (FBV)

### 📚 Documentation
- [Django FBV Documentation](https://docs.djangoproject.com/en/stable/topics/http/views/)
- [Request/Response Objects](https://docs.djangoproject.com/en/stable/ref/request-response/)
- [Decorators Reference](https://docs.djangoproject.com/en/stable/topics/http/decorators/)

### ✅ Step-by-Step Checklist

#### Step 1: Import Required Modules
- [ ] Import HttpResponse or JsonResponse from `django.http`
- [ ] Import render or redirect from `django.shortcuts`
- [ ] Import get_object_or_404 if needed
- [ ] Import required models
- [ ] Import required forms (if applicable)
- [ ] Import any decorators (@login_required, @require_http_methods, etc.)

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.http import HttpResponse, JsonResponse
from django.contrib.auth.decorators import login_required
from django.views.decorators.http import require_http_methods
from .models import YourModel
from .forms import YourForm
```

#### Step 2: Define Function Signature
- [ ] Choose descriptive function name (verb + noun: list_products, create_user)
- [ ] Add `request` as first parameter
- [ ] Add URL parameters as additional arguments (pk, slug, etc.)
- [ ] Add type hints if desired

```python
def view_name(request, pk=None):
    pass
```

#### Step 3: Add Decorators (if needed)
- [ ] @login_required for authentication
- [ ] @require_http_methods(['GET', 'POST']) for HTTP method restrictions
- [ ] @permission_required for permissions
- [ ] Custom decorators

```python
@login_required
@require_http_methods(['GET', 'POST'])
def view_name(request, pk=None):
    pass
```

#### Step 4: Handle HTTP Methods
- [ ] Check request.method (GET, POST, PUT, DELETE)
- [ ] Separate logic for each method
- [ ] Return appropriate response for each method

```python
def view_name(request, pk=None):
    if request.method == 'GET':
        # Handle GET
        pass
    elif request.method == 'POST':
        # Handle POST
        pass
```

#### Step 5: Query Database (if needed)
- [ ] Use get_object_or_404 for single objects
- [ ] Use Model.objects.filter() for querysets
- [ ] Add select_related() or prefetch_related() for optimization
- [ ] Handle DoesNotExist exceptions

```python
# Single object
obj = get_object_or_404(YourModel, pk=pk)

# Multiple objects
queryset = YourModel.objects.filter(active=True).select_related('related_model')
```

#### Step 6: Handle Forms (if applicable)
- [ ] Initialize form (empty for GET, with data for POST)
- [ ] Check form.is_valid()
- [ ] Save form data
- [ ] Add success/error messages
- [ ] Redirect after successful POST

```python
if request.method == 'POST':
    form = YourForm(request.POST, request.FILES)
    if form.is_valid():
        instance = form.save(commit=False)
        instance.user = request.user
        instance.save()
        return redirect('success_url')
else:
    form = YourForm()
```

#### Step 7: Prepare Context Data
- [ ] Create context dictionary
- [ ] Add all variables needed in template
- [ ] Include form if applicable

```python
context = {
    'object': obj,
    'form': form,
    'queryset': queryset,
}
```

#### Step 8: Return Response
- [ ] render() for HTML templates
- [ ] redirect() for redirects
- [ ] JsonResponse() for JSON
- [ ] HttpResponse() for plain text/custom content

```python
return render(request, 'app/template.html', context)
# OR
return JsonResponse({'status': 'success', 'data': data})
# OR
return redirect('view_name', pk=obj.pk)
```

#### Step 9: Add URL Pattern
- [ ] Import view in urls.py
- [ ] Add path() or re_path()
- [ ] Give URL a name
- [ ] Include any parameters

```python
# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('items/', views.list_items, name='list_items'),
    path('items/<int:pk>/', views.item_detail, name='item_detail'),
]
```

#### Step 10: Test Your View
- [ ] Test with GET request
- [ ] Test with POST request (if applicable)
- [ ] Test authentication/permissions
- [ ] Test edge cases (invalid data, missing objects)
- [ ] Check response status codes

---

## Class-Based Views (CBV)

### 📚 Documentation
- [CBV Documentation](https://docs.djangoproject.com/en/stable/topics/class-based-views/)
- [Built-in CBVs](https://docs.djangoproject.com/en/stable/ref/class-based-views/)
- [Mixins Reference](https://docs.djangoproject.com/en/stable/topics/class-based-views/mixins/)
- [Classy Class-Based Views](https://ccbv.co.uk/) (unofficial but excellent)

### ✅ Step-by-Step Checklist

#### Step 1: Choose Appropriate Base Class
- [ ] ListView - for listing objects
- [ ] DetailView - for single object detail
- [ ] CreateView - for creating objects
- [ ] UpdateView - for updating objects
- [ ] DeleteView - for deleting objects
- [ ] FormView - for forms without models
- [ ] TemplateView - for simple template rendering
- [ ] View - for custom logic

```python
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
```

#### Step 2: Import Required Modules
- [ ] Import chosen base class
- [ ] Import mixins (LoginRequiredMixin, PermissionRequiredMixin, etc.)
- [ ] Import models
- [ ] Import forms (if using CreateView/UpdateView)
- [ ] Import reverse_lazy for success_url

```python
from django.views.generic import CreateView
from django.contrib.auth.mixins import LoginRequiredMixin
from django.urls import reverse_lazy
from .models import YourModel
from .forms import YourForm
```

#### Step 3: Define Class and Inheritance
- [ ] Choose class name (ends with View: ProductListView)
- [ ] Add mixins BEFORE base view class
- [ ] Inherit from appropriate base class

```python
class ProductListView(LoginRequiredMixin, ListView):
    pass
```

#### Step 4: Set Required Attributes
- [ ] **model** - the model to work with
- [ ] **template_name** - template path (optional, has default)
- [ ] **context_object_name** - variable name in template (optional)
- [ ] **form_class** - form to use (for CreateView/UpdateView)
- [ ] **success_url** - redirect after success (for CreateView/UpdateView/DeleteView)
- [ ] **fields** - model fields for form (alternative to form_class)
- [ ] **paginate_by** - number of items per page (for ListView)

```python
class ProductListView(LoginRequiredMixin, ListView):
    model = Product
    template_name = 'products/product_list.html'
    context_object_name = 'products'
    paginate_by = 10
```

#### Step 5: Override get_queryset() (if needed)
- [ ] Filter queryset
- [ ] Add select_related/prefetch_related
- [ ] Order queryset
- [ ] Filter by request.user if needed

```python
def get_queryset(self):
    queryset = super().get_queryset()
    return queryset.filter(active=True).select_related('category')
```

#### Step 6: Override get_context_data() (if needed)
- [ ] Call super().get_context_data(**kwargs)
- [ ] Add additional context variables
- [ ] Return context

```python
def get_context_data(self, **kwargs):
    context = super().get_context_data(**kwargs)
    context['extra_data'] = 'some value'
    return context
```

#### Step 7: Override form_valid() (for Create/UpdateView)
- [ ] Add custom logic before save
- [ ] Modify form.instance if needed
- [ ] Call super().form_valid(form)
- [ ] Add success message
- [ ] Return response

```python
def form_valid(self, form):
    form.instance.user = self.request.user
    messages.success(self.request, 'Successfully created!')
    return super().form_valid(form)
```

#### Step 8: Add Authentication/Permissions
- [ ] Add LoginRequiredMixin
- [ ] Add PermissionRequiredMixin with permission_required
- [ ] Override test_func() for UserPassesTestMixin
- [ ] Set login_url if needed

```python
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin

class ProductCreateView(LoginRequiredMixin, PermissionRequiredMixin, CreateView):
    permission_required = 'products.add_product'
    login_url = '/login/'
```

#### Step 9: Add URL Pattern
- [ ] Import view class
- [ ] Use .as_view() method
- [ ] Give URL a name
- [ ] Include parameters

```python
# urls.py
from django.urls import path
from .views import ProductListView, ProductDetailView

urlpatterns = [
    path('products/', ProductListView.as_view(), name='product_list'),
    path('products/<int:pk>/', ProductDetailView.as_view(), name='product_detail'),
]
```

#### Step 10: Test Your View
- [ ] Test GET requests
- [ ] Test POST requests (for forms)
- [ ] Test authentication/permissions
- [ ] Test pagination (for ListView)
- [ ] Test form validation
- [ ] Verify redirects

---

## DRF APIView

### 📚 Documentation
- [DRF APIView Documentation](https://www.django-rest-framework.org/api-guide/views/)
- [Serializers](https://www.django-rest-framework.org/api-guide/serializers/)
- [Permissions](https://www.django-rest-framework.org/api-guide/permissions/)
- [Authentication](https://www.django-rest-framework.org/api-guide/authentication/)

### ✅ Step-by-Step Checklist

#### Step 1: Import Required Modules
- [ ] Import APIView from rest_framework.views
- [ ] Import Response from rest_framework.response
- [ ] Import status from rest_framework
- [ ] Import serializers
- [ ] Import models
- [ ] Import permissions
- [ ] Import authentication classes

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from rest_framework.permissions import IsAuthenticated
from django.shortcuts import get_object_or_404
from .models import YourModel
from .serializers import YourSerializer
```

#### Step 2: Define Class
- [ ] Choose descriptive class name (ends with APIView)
- [ ] Inherit from APIView
- [ ] Add docstring

```python
class ProductListAPIView(APIView):
    """
    List all products or create a new product.
    """
    pass
```

#### Step 3: Set Class Attributes
- [ ] **permission_classes** - list of permission classes
- [ ] **authentication_classes** - list of authentication classes
- [ ] **throttle_classes** - rate limiting (optional)

```python
class ProductListAPIView(APIView):
    permission_classes = [IsAuthenticated]
    # authentication_classes = [TokenAuthentication]
```

#### Step 4: Create Serializer (if not exists)
- [ ] Create serializer class in serializers.py
- [ ] Define Meta class with model and fields
- [ ] Add validation methods if needed

```python
# serializers.py
from rest_framework import serializers
from .models import YourModel

class YourSerializer(serializers.ModelSerializer):
    class Meta:
        model = YourModel
        fields = '__all__'  # or list specific fields
        # read_only_fields = ['id', 'created_at']
```

#### Step 5: Implement GET Method
- [ ] Define get() method with request parameter
- [ ] Query database
- [ ] Serialize data (many=True for querysets)
- [ ] Return Response with serialized data
- [ ] Handle exceptions

```python
def get(self, request, pk=None):
    if pk:
        # Single object
        obj = get_object_or_404(YourModel, pk=pk)
        serializer = YourSerializer(obj)
    else:
        # Multiple objects
        queryset = YourModel.objects.all()
        serializer = YourSerializer(queryset, many=True)
    
    return Response(serializer.data)
```

#### Step 6: Implement POST Method
- [ ] Define post() method with request parameter
- [ ] Create serializer with request.data
- [ ] Check serializer.is_valid()
- [ ] Call serializer.save()
- [ ] Return Response with created data and 201 status
- [ ] Return validation errors if invalid

```python
def post(self, request):
    serializer = YourSerializer(data=request.data)
    if serializer.is_valid():
        serializer.save()
        return Response(serializer.data, status=status.HTTP_201_CREATED)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

#### Step 7: Implement PUT/PATCH Method (for updates)
- [ ] Define put() or patch() method
- [ ] Get object with pk
- [ ] Create serializer with instance and request.data
- [ ] Set partial=True for PATCH
- [ ] Validate and save
- [ ] Return updated data

```python
def put(self, request, pk):
    obj = get_object_or_404(YourModel, pk=pk)
    serializer = YourSerializer(obj, data=request.data)
    if serializer.is_valid():
        serializer.save()
        return Response(serializer.data)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

def patch(self, request, pk):
    obj = get_object_or_404(YourModel, pk=pk)
    serializer = YourSerializer(obj, data=request.data, partial=True)
    if serializer.is_valid():
        serializer.save()
        return Response(serializer.data)
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

#### Step 8: Implement DELETE Method
- [ ] Define delete() method
- [ ] Get object with pk
- [ ] Call obj.delete()
- [ ] Return Response with 204 status

```python
def delete(self, request, pk):
    obj = get_object_or_404(YourModel, pk=pk)
    obj.delete()
    return Response(status=status.HTTP_204_NO_CONTENT)
```

#### Step 9: Add URL Pattern
- [ ] Import view class
- [ ] Use .as_view() method
- [ ] Give URL a name
- [ ] Include parameters for detail views

```python
# urls.py
from django.urls import path
from .views import ProductListAPIView, ProductDetailAPIView

urlpatterns = [
    path('api/products/', ProductListAPIView.as_view(), name='api_product_list'),
    path('api/products/<int:pk>/', ProductDetailAPIView.as_view(), name='api_product_detail'),
]
```

#### Step 10: Test Your API
- [ ] Test with Postman or curl
- [ ] Test GET requests
- [ ] Test POST with valid data
- [ ] Test POST with invalid data
- [ ] Test PUT/PATCH updates
- [ ] Test DELETE
- [ ] Test authentication
- [ ] Test permissions
- [ ] Check status codes

---

## DRF ViewSets

### 📚 Documentation
- [ViewSets Documentation](https://www.django-rest-framework.org/api-guide/viewsets/)
- [Routers](https://www.django-rest-framework.org/api-guide/routers/)
- [Generic ViewSets](https://www.django-rest-framework.org/api-guide/viewsets/#genericviewset)

### ✅ Step-by-Step Checklist

#### Step 1: Import Required Modules
- [ ] Import viewsets from rest_framework
- [ ] Import models
- [ ] Import serializers
- [ ] Import permissions
- [ ] Import filters (if needed)
- [ ] Import decorators for custom actions

```python
from rest_framework import viewsets
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from .models import YourModel
from .serializers import YourSerializer
```

#### Step 2: Choose ViewSet Type
- [ ] **ModelViewSet** - full CRUD (list, create, retrieve, update, destroy)
- [ ] **ReadOnlyModelViewSet** - list and retrieve only
- [ ] **GenericViewSet** - build custom with mixins
- [ ] **ViewSet** - completely custom

```python
from rest_framework import viewsets

class ProductViewSet(viewsets.ModelViewSet):
    pass
```

#### Step 3: Set Required Attributes
- [ ] **queryset** - base queryset
- [ ] **serializer_class** - default serializer
- [ ] **permission_classes** - list of permissions
- [ ] **authentication_classes** - authentication methods
- [ ] **filter_backends** - filtering (optional)
- [ ] **filterset_fields** - fields to filter by (optional)
- [ ] **search_fields** - fields to search (optional)
- [ ] **ordering_fields** - fields to order by (optional)

```python
from rest_framework import viewsets, filters
from django_filters.rest_framework import DjangoFilterBackend

class ProductViewSet(viewsets.ModelViewSet):
    queryset = YourModel.objects.all()
    serializer_class = YourSerializer
    permission_classes = [IsAuthenticated]
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_fields = ['category', 'active']
    search_fields = ['name', 'description']
    ordering_fields = ['created_at', 'name']
```

#### Step 4: Override get_queryset() (if needed)
- [ ] Add custom filtering logic
- [ ] Filter by request.user
- [ ] Add optimizations
- [ ] Return filtered queryset

```python
def get_queryset(self):
    queryset = super().get_queryset()
    if not self.request.user.is_staff:
        queryset = queryset.filter(user=self.request.user)
    return queryset.select_related('category')
```

#### Step 5: Override get_serializer_class() (if needed)
- [ ] Return different serializers for different actions
- [ ] Check self.action
- [ ] Return appropriate serializer

```python
def get_serializer_class(self):
    if self.action == 'list':
        return ProductListSerializer
    elif self.action == 'retrieve':
        return ProductDetailSerializer
    return ProductSerializer
```

#### Step 6: Override perform_create() (if needed)
- [ ] Add custom logic before save
- [ ] Modify serializer.validated_data
- [ ] Call serializer.save()

```python
def perform_create(self, serializer):
    serializer.save(user=self.request.user)
```

#### Step 7: Override perform_update() (if needed)
- [ ] Add custom logic before update
- [ ] Call serializer.save()

```python
def perform_update(self, serializer):
    serializer.save(modified_by=self.request.user)
```

#### Step 8: Add Custom Actions (if needed)
- [ ] Use @action decorator
- [ ] Set detail=True for single object, False for collection
- [ ] Set methods=['get'] or ['post']
- [ ] Set url_path and url_name (optional)
- [ ] Define method
- [ ] Return Response

```python
@action(detail=True, methods=['post'])
def activate(self, request, pk=None):
    obj = self.get_object()
    obj.active = True
    obj.save()
    serializer = self.get_serializer(obj)
    return Response(serializer.data)

@action(detail=False, methods=['get'])
def recent(self, request):
    recent_items = self.get_queryset().order_by('-created_at')[:10]
    serializer = self.get_serializer(recent_items, many=True)
    return Response(serializer.data)
```

#### Step 9: Register with Router
- [ ] Import DefaultRouter or SimpleRouter
- [ ] Create router instance
- [ ] Register viewset with router
- [ ] Include router.urls in urlpatterns

```python
# urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import ProductViewSet

router = DefaultRouter()
router.register(r'products', ProductViewSet, basename='product')

urlpatterns = [
    path('api/', include(router.urls)),
]
```

#### Step 10: Test Your ViewSet
- [ ] Test list endpoint (GET)
- [ ] Test create endpoint (POST)
- [ ] Test retrieve endpoint (GET with pk)
- [ ] Test update endpoints (PUT/PATCH with pk)
- [ ] Test delete endpoint (DELETE with pk)
- [ ] Test custom actions
- [ ] Test filtering, searching, ordering
- [ ] Test pagination
- [ ] Test permissions
- [ ] Test authentication

---

## Common Patterns

### Adding Pagination (DRF)

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}

# Or in view
from rest_framework.pagination import PageNumberPagination

class CustomPagination(PageNumberPagination):
    page_size = 10
    page_size_query_param = 'page_size'
    max_page_size = 100

class ProductViewSet(viewsets.ModelViewSet):
    pagination_class = CustomPagination
```

### Adding Filtering (DRF)

```python
# Install django-filter: pip install django-filter

# settings.py
INSTALLED_APPS = [
    'django_filters',
]

REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': ['django_filters.rest_framework.DjangoFilterBackend']
}

# views.py
class ProductViewSet(viewsets.ModelViewSet):
    filterset_fields = ['category', 'price', 'active']
```

### Custom Permissions (DRF)

```python
# permissions.py
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.user == request.user

# views.py
class ProductViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated, IsOwnerOrReadOnly]
```

### Handling File Uploads (FBV)

```python
def upload_file(request):
    if request.method == 'POST':
        form = FileUploadForm(request.POST, request.FILES)
        if form.is_valid():
            instance = form.save()
            return redirect('success')
    else:
        form = FileUploadForm()
    return render(request, 'upload.html', {'form': form})
```

### Ajax Responses (FBV)

```python
from django.http import JsonResponse

def ajax_view(request):
    if request.headers.get('x-requested-with') == 'XMLHttpRequest':
        data = {
            'status': 'success',
            'message': 'Data processed'
        }
        return JsonResponse(data)
    return render(request, 'template.html')
```

---

## Quick Reference Table

| View Type | Use Case | Pros | Cons |
|-----------|----------|------|------|
| FBV | Simple views, custom logic | Easy to understand, flexible | Can get verbose |
| CBV | Standard CRUD operations | DRY, reusable, powerful | Steeper learning curve |
| DRF APIView | Custom API endpoints | Full control, flexible | More code required |
| DRF ViewSet | Standard REST API | Less code, automatic routing | Less control |

---

## Testing Checklist

### General Testing
- [ ] View accessible at correct URL
- [ ] Correct template rendered (for HTML views)
- [ ] Correct status codes returned
- [ ] Redirects work properly
- [ ] Error handling works
- [ ] Edge cases handled

### Authentication Testing
- [ ] Unauthenticated users redirected/blocked
- [ ] Authenticated users can access
- [ ] Correct permissions enforced
- [ ] Authorization errors handled

### Form/Data Testing
- [ ] Valid data accepted
- [ ] Invalid data rejected with errors
- [ ] Form validation works
- [ ] Database updates correctly
- [ ] Files upload correctly

### API Testing
- [ ] JSON responses well-formed
- [ ] Serialization works correctly
- [ ] Filtering works
- [ ] Pagination works
- [ ] Search works
- [ ] Custom actions work

---

## Troubleshooting Common Issues

### FBV Issues
- **Form not saving**: Check `form.is_valid()` and `form.save()`
- **Template not found**: Verify template path and TEMPLATES setting
- **Redirect not working**: Check URL name and parameters

### CBV Issues
- **AttributeError**: Check that required attributes are set
- **Method not allowed**: Verify HTTP methods in form
- **Context not available**: Override `get_context_data()`

### DRF Issues
- **403 Forbidden**: Check authentication and permissions
- **Serializer errors**: Check serializer fields and validation
- **Router not working**: Verify router registration and URL include

---

## Additional Resources

- [Django Documentation](https://docs.djangoproject.com/)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [Classy Class-Based Views](https://ccbv.co.uk/)
- [Classy Django REST Framework](https://www.cdrf.co/)
- [Django Girls Tutorial](https://tutorial.djangogirls.org/)
- [Two Scoops of Django](https://www.feldroy.com/books/two-scoops-of-django-3-x)

---

**Pro Tips:**
- Start with FBV for learning, move to CBV for reusability
- Use ViewSets for standard REST APIs, APIView for custom logic
- Always add docstrings to your views
- Use type hints for better IDE support
- Write tests for all views
- Follow Django/DRF naming conventions
- Keep views thin - move business logic to models or services

