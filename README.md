# Django Web Development Guide

A comprehensive reference guide for building web applications with Django.

---

## 📑 Table of Contents

1. [Environment Setup](#1-environment-setup)
   - [Virtual Environment](#virtual-environment)
   - [Installing Django](#installing-django)
2. [Project Setup](#2-project-setup)
   - [Creating a Project](#creating-a-project)
   - [Project Structure](#project-structure)
   - [Core Django Files](#core-django-files)
3. [Application Development](#3-application-development)
   - [Step 1: Create an Application](#step-1-create-an-application)
   - [Step 2: Define Models](#step-2-define-models)
   - [Step 3: Database Migrations](#step-3-database-migrations)
   - [Step 4: Register Models in Admin](#step-4-register-models-in-admin)
   - [Step 5: Configure URLs](#step-5-configure-urls)
   - [Step 6: Create Views](#step-6-create-views)
   - [Step 7: Create Templates](#step-7-create-templates)
   - [Step 8: Static Files](#step-8-static-files)
   - [Step 9: Forms](#step-9-forms)
4. [Database Configuration](#4-database-configuration)
   - [SQLite (Default)](#sqlite-default)
   - [MySQL Setup](#mysql-setup)
   - [Working with Legacy Databases](#working-with-legacy-databases)
5. [Admin Panel](#5-admin-panel)
6. [Authentication](#6-authentication)
7. [Quick Reference Commands](#7-quick-reference-commands)
8. [Common Field Types](#8-common-field-types)
9. [Troubleshooting](#9-troubleshooting)

---

## 1. Environment Setup

### Virtual Environment

#### Windows
```bash
# Create virtual environment
python -m venv myvenv

# Activate virtual environment
myvenv\Scripts\activate
```

#### Linux / macOS
```bash
# Create virtual environment
python3 -m venv myvenv

# Activate virtual environment
source myvenv/bin/activate
```

> 💡 **Tip:** You'll see `(myvenv)` in your terminal when the environment is active.

### Installing Django

```bash
# Upgrade pip first
python -m pip install --upgrade pip

# Install Django
pip install Django~=5.1.2

# Save dependencies to requirements.txt
pip freeze > requirements.txt

# Install from requirements.txt (on another machine)
pip install -r requirements.txt
```

---

## 2. Project Setup

### Creating a Project

#### Windows
```bash
django-admin.exe startproject mysite .
```

#### Linux / macOS
```bash
django-admin startproject mysite .
```

> ⚠️ **Note:** The `.` at the end creates the project in the current directory.

### Project Structure

After creating a project, you'll have:
```
myproject/
├── manage.py
└── mysite/
    ├── __init__.py
    ├── asgi.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

### Core Django Files

| File | Purpose |
|------|---------|
| `manage.py` | Command-line utility for managing your site (e.g., starting the web server, migrations) |
| `settings.py` | Configuration file for your website (database, installed apps, middleware, etc.) |
| `urls.py` | URL routing - maps URLs to views |
| `wsgi.py` | Entry point for WSGI-compatible web servers (production) |
| `asgi.py` | Entry point for ASGI-compatible web servers (async support) |

### Starting the Development Server

```bash
python manage.py runserver

# Run on specific port
python manage.py runserver 8080

# Run accessible from network
python manage.py runserver 0.0.0.0:8000
```

---

## 3. Application Development

### Step 1: Create an Application

```bash
python manage.py startapp blog
```

**Register the app in `settings.py`:**
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog',  # Add your app here
]
```

**App structure created:**
```
blog/
├── __init__.py
├── admin.py      # Admin panel configuration
├── apps.py       # App configuration
├── models.py     # Database models
├── tests.py      # Unit tests
├── views.py      # View functions/classes
└── migrations/   # Database migration files
```

---

### Step 2: Define Models

Models represent database tables. Each model is a Python class that inherits from `models.Model`.

**Example: `blog/models.py`**
```python
from django.db import models
from django.conf import settings
from django.utils import timezone

class Post(models.Model):
    author = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE
    )
    title = models.CharField(max_length=200)
    text = models.TextField()
    created_date = models.DateTimeField(default=timezone.now)
    published_date = models.DateTimeField(blank=True, null=True)

    def publish(self):
        self.published_date = timezone.now()
        self.save()

    def __str__(self):
        return self.title

    class Meta:
        ordering = ['-created_date']
```

**Key Concepts:**
- `class`: Python keyword to define an object
- `models.Model`: Base class for Django models
- `__str__`: Returns a human-readable representation
- `Meta`: Model metadata (ordering, table name, etc.)

---

### Step 3: Database Migrations

After creating or modifying models, sync them with the database:

```bash
# Create migration files (detects model changes)
python manage.py makemigrations blog

# Preview SQL that will be executed
python manage.py sqlmigrate blog 0001

# Apply migrations to database
python manage.py migrate blog

# Apply all pending migrations
python manage.py migrate
```

> 💡 **Tip:** Always run `makemigrations` after changing models, then `migrate`.

---

### Step 4: Register Models in Admin

Make models available in the Django admin interface.

**`blog/admin.py`**
```python
from django.contrib import admin
from .models import Post

# Basic registration
admin.site.register(Post)

# Advanced: Customize admin display
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'created_date', 'published_date']
    list_filter = ['created_date', 'author']
    search_fields = ['title', 'text']
    date_hierarchy = 'created_date'
    ordering = ['-created_date']
```

---

### Step 5: Configure URLs

#### Project-level URLs (`mysite/urls.py`)
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),  # Include app URLs
]
```

#### App-level URLs (`blog/urls.py`) - Create this file
```python
from django.urls import path
from . import views

app_name = 'blog'  # Namespace for URL reversing

urlpatterns = [
    path('', views.post_list, name='post_list'),
    path('post/<int:pk>/', views.post_detail, name='post_detail'),
    path('post/new/', views.post_new, name='post_new'),
    path('post/<int:pk>/edit/', views.post_edit, name='post_edit'),
    path('post/<int:pk>/delete/', views.post_delete, name='post_delete'),
]
```

**URL Pattern Syntax:**
| Pattern | Matches | Example |
|---------|---------|---------|
| `''` | Empty path | `/` |
| `'post/'` | Literal path | `/post/` |
| `'<int:pk>/'` | Integer parameter | `/post/5/` |
| `'<str:slug>/'` | String parameter | `/post/my-post/` |
| `'<slug:slug>/'` | Slug (letters, numbers, hyphens) | `/post/my-post-123/` |

---

### Step 6: Create Views

Views handle request/response logic. They receive requests, fetch data, and return responses.

**`blog/views.py`**

#### Function-Based Views (FBV)
```python
from django.shortcuts import render, get_object_or_404, redirect
from django.utils import timezone
from .models import Post
from .forms import PostForm

def post_list(request):
    """Display all published posts"""
    posts = Post.objects.filter(
        published_date__lte=timezone.now()
    ).order_by('-published_date')
    return render(request, 'blog/post_list.html', {'posts': posts})

def post_detail(request, pk):
    """Display a single post"""
    post = get_object_or_404(Post, pk=pk)
    return render(request, 'blog/post_detail.html', {'post': post})

def post_new(request):
    """Create a new post"""
    if request.method == "POST":
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.published_date = timezone.now()
            post.save()
            return redirect('blog:post_detail', pk=post.pk)
    else:
        form = PostForm()
    return render(request, 'blog/post_edit.html', {'form': form})

def post_edit(request, pk):
    """Edit an existing post"""
    post = get_object_or_404(Post, pk=pk)
    if request.method == "POST":
        form = PostForm(request.POST, instance=post)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.published_date = timezone.now()
            post.save()
            return redirect('blog:post_detail', pk=post.pk)
    else:
        form = PostForm(instance=post)
    return render(request, 'blog/post_edit.html', {'form': form})

def post_delete(request, pk):
    """Delete a post"""
    post = get_object_or_404(Post, pk=pk)
    post.delete()
    return redirect('blog:post_list')
```

#### Class-Based Views (CBV)
```python
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Post

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
    ordering = ['-published_date']

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'

class PostCreateView(CreateView):
    model = Post
    fields = ['title', 'text']
    template_name = 'blog/post_edit.html'
    
    def form_valid(self, form):
        form.instance.author = self.request.user
        return super().form_valid(form)

class PostUpdateView(UpdateView):
    model = Post
    fields = ['title', 'text']
    template_name = 'blog/post_edit.html'

class PostDeleteView(DeleteView):
    model = Post
    success_url = reverse_lazy('blog:post_list')
```

---

### Step 7: Create Templates

Templates contain HTML with Django Template Language (DTL).

**Directory structure:**
```
blog/
└── templates/
    └── blog/
        ├── base.html
        ├── post_list.html
        ├── post_detail.html
        └── post_edit.html
```

#### Base Template (`blog/templates/blog/base.html`)
```html
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}My Blog{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'css/blog.css' %}">
</head>
<body>
    <header>
        <h1><a href="{% url 'blog:post_list' %}">My Blog</a></h1>
    </header>
    
    <main>
        {% block content %}
        {% endblock %}
    </main>
    
    <footer>
        <p>&copy; 2024 My Blog</p>
    </footer>
</body>
</html>
```

#### List Template (`blog/templates/blog/post_list.html`)
```html
{% extends 'blog/base.html' %}

{% block title %}Posts{% endblock %}

{% block content %}
    <a href="{% url 'blog:post_new' %}">New Post</a>
    
    {% for post in posts %}
        <article>
            <h2><a href="{% url 'blog:post_detail' pk=post.pk %}">{{ post.title }}</a></h2>
            <time>{{ post.published_date|date:"F d, Y" }}</time>
            <p>{{ post.text|truncatewords:30 }}</p>
        </article>
    {% empty %}
        <p>No posts available.</p>
    {% endfor %}
{% endblock %}
```

#### Detail Template (`blog/templates/blog/post_detail.html`)
```html
{% extends 'blog/base.html' %}

{% block title %}{{ post.title }}{% endblock %}

{% block content %}
    <article>
        <h1>{{ post.title }}</h1>
        <time>{{ post.published_date|date:"F d, Y" }}</time>
        <p>{{ post.text|linebreaks }}</p>
        
        <a href="{% url 'blog:post_edit' pk=post.pk %}">Edit</a>
        <a href="{% url 'blog:post_delete' pk=post.pk %}">Delete</a>
    </article>
{% endblock %}
```

#### Form Template (`blog/templates/blog/post_edit.html`)
```html
{% extends 'blog/base.html' %}

{% block title %}Edit Post{% endblock %}

{% block content %}
    <h1>{% if form.instance.pk %}Edit{% else %}New{% endif %} Post</h1>
    
    <form method="POST">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Save</button>
    </form>
{% endblock %}
```

**Common Template Tags:**
| Tag | Purpose |
|-----|---------|
| `{% extends %}` | Inherit from parent template |
| `{% block %}` | Define overridable content blocks |
| `{% include %}` | Include another template |
| `{% for %}` | Loop over items |
| `{% if %}` | Conditional logic |
| `{% url %}` | Generate URL from name |
| `{% static %}` | Reference static files |
| `{% csrf_token %}` | CSRF protection for forms |
| `{{ variable }}` | Output variable value |
| `{{ var\|filter }}` | Apply filter to variable |

---

### Step 8: Static Files

Static files include CSS, JavaScript, and images.

**Directory structure:**
```
blog/
└── static/
    └── css/
        └── blog.css
    └── js/
        └── main.js
    └── images/
        └── logo.png
```

**Configure in `settings.py`:**
```python
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'

# Additional static file directories
STATICFILES_DIRS = [
    BASE_DIR / 'static',
]
```

**Use in templates:**
```html
{% load static %}
<link rel="stylesheet" href="{% static 'css/blog.css' %}">
<script src="{% static 'js/main.js' %}"></script>
<img src="{% static 'images/logo.png' %}" alt="Logo">
```

**Collect static files for production:**
```bash
python manage.py collectstatic
```

---

### Step 9: Forms

Forms handle user input validation and processing.

**`blog/forms.py`**
```python
from django import forms
from .models import Post

# ModelForm - based on a model
class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'text']
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': 'Enter title'
            }),
            'text': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 10
            }),
        }

# Regular Form - not tied to a model
class ContactForm(forms.Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
    message = forms.CharField(widget=forms.Textarea)
    
    def clean_email(self):
        email = self.cleaned_data.get('email')
        if not email.endswith('@example.com'):
            raise forms.ValidationError("Please use a company email.")
        return email
```

---

## 4. Database Configuration

### SQLite (Default)

Default configuration in `settings.py`:
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

---

### MySQL Setup

#### Step 1: Install MySQL Client
```bash
pip install mysqlclient

# On Ubuntu/Debian, you may need:
sudo apt-get install python3-dev default-libmysqlclient-dev build-essential pkg-config
```

#### Step 2: Create MySQL Database
```sql
-- Login to MySQL
mysql -u root -p

-- Create database
CREATE DATABASE myproject_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Create user
CREATE USER 'myproject_user'@'localhost' IDENTIFIED BY 'your_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON myproject_db.* TO 'myproject_user'@'localhost';
FLUSH PRIVILEGES;

-- Exit
EXIT;
```

#### Step 3: Configure Django (`settings.py`)
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'myproject_db',
        'USER': 'myproject_user',
        'PASSWORD': 'your_password',
        'HOST': 'localhost',
        'PORT': '3306',
        'OPTIONS': {
            'init_command': "SET sql_mode='STRICT_TRANS_TABLES'",
            'charset': 'utf8mb4',
        },
    }
}
```

#### Step 4: Run Migrations
```bash
python manage.py migrate
```

---

### Working with Legacy Databases

When working with an existing MySQL database, Django can auto-generate models.

#### Step 1: Configure Database Connection
```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'legacy_database',
        'USER': 'db_user',
        'PASSWORD': 'db_password',
        'HOST': 'localhost',
        'PORT': '3306',
    }
}
```

#### Step 2: Generate Models from Existing Tables
```bash
# Inspect all tables and generate models
python manage.py inspectdb > blog/models.py

# Inspect specific tables
python manage.py inspectdb table1 table2 > blog/models.py
```

#### Step 3: Review and Clean Up Generated Models

The generated models will look like:
```python
# This is an auto-generated Django model module.
# You'll have to do the following manually to clean this up:
#   * Rearrange models' order
#   * Make sure each model has one field with primary_key=True
#   * Remove `managed = False` if you want Django to manage the table

class LegacyTable(models.Model):
    id = models.AutoField(primary_key=True)
    name = models.CharField(max_length=255)
    created_at = models.DateTimeField()

    class Meta:
        managed = False  # Django won't create/modify this table
        db_table = 'legacy_table'  # Actual table name in database
```

**Key cleanup tasks:**
1. Add `__str__` methods
2. Define relationships (`ForeignKey`, `ManyToMany`)
3. Add `related_name` to avoid conflicts
4. Set `managed = True` if you want Django to handle migrations

#### Step 4: Create Initial Migration (Without Applying)
```bash
# Create migration that matches current database state
python manage.py makemigrations blog

# Fake the migration (mark as applied without running)
python manage.py migrate blog --fake-initial
```

#### Step 5: Multiple Databases (Optional)

For using both legacy and new databases:
```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    },
    'legacy': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'legacy_database',
        'USER': 'db_user',
        'PASSWORD': 'db_password',
        'HOST': 'localhost',
        'PORT': '3306',
    }
}
```

**Create a database router (`blog/db_router.py`):**
```python
class LegacyRouter:
    """Route legacy models to the legacy database."""
    
    legacy_models = {'legacytable', 'anotherlegacymodel'}
    
    def db_for_read(self, model, **hints):
        if model._meta.model_name in self.legacy_models:
            return 'legacy'
        return 'default'
    
    def db_for_write(self, model, **hints):
        if model._meta.model_name in self.legacy_models:
            return 'legacy'
        return 'default'
    
    def allow_relation(self, obj1, obj2, **hints):
        return True
    
    def allow_migrate(self, db, app_label, model_name=None, **hints):
        if model_name in self.legacy_models:
            return db == 'legacy'
        return db == 'default'
```

**Register router in `settings.py`:**
```python
DATABASE_ROUTERS = ['blog.db_router.LegacyRouter']
```

---

## 5. Admin Panel

### Create Superuser
```bash
python manage.py createsuperuser
```

### Access Admin
Navigate to: `http://127.0.0.1:8000/admin/`

### Customize Admin Site
```python
# blog/admin.py
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    # Display columns
    list_display = ['title', 'author', 'created_date', 'is_published']
    
    # Filter sidebar
    list_filter = ['created_date', 'author', 'published_date']
    
    # Search fields
    search_fields = ['title', 'text']
    
    # Date navigation
    date_hierarchy = 'created_date'
    
    # Default ordering
    ordering = ['-created_date']
    
    # Fields in edit form
    fields = ['title', 'text', 'author', 'published_date']
    
    # Read-only fields
    readonly_fields = ['created_date']
    
    # Prepopulate slug from title
    prepopulated_fields = {'slug': ('title',)}
    
    # Custom method for display
    def is_published(self, obj):
        return obj.published_date is not None
    is_published.boolean = True
    is_published.short_description = 'Published'
```

---

## 6. Authentication

### Built-in Authentication Views

**Add to `urls.py`:**
```python
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('login/', auth_views.LoginView.as_view(), name='login'),
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),
]
```

**Configure in `settings.py`:**
```python
LOGIN_REDIRECT_URL = '/'
LOGOUT_REDIRECT_URL = '/'
LOGIN_URL = '/login/'
```

### Protect Views

**Function-based views:**
```python
from django.contrib.auth.decorators import login_required

@login_required
def post_new(request):
    # Only authenticated users can access
    pass
```

**Class-based views:**
```python
from django.contrib.auth.mixins import LoginRequiredMixin

class PostCreateView(LoginRequiredMixin, CreateView):
    model = Post
    fields = ['title', 'text']
```

### Check Authentication in Templates
```html
{% if user.is_authenticated %}
    <p>Welcome, {{ user.username }}!</p>
    <a href="{% url 'logout' %}">Logout</a>
{% else %}
    <a href="{% url 'login' %}">Login</a>
{% endif %}
```

---

## 7. Quick Reference Commands

```bash
# ============ PROJECT SETUP ============
django-admin startproject mysite .    # Create project
python manage.py startapp appname     # Create app

# ============ DATABASE ============
python manage.py makemigrations       # Create migration files
python manage.py makemigrations app   # Create for specific app
python manage.py migrate              # Apply all migrations
python manage.py migrate app          # Apply specific app
python manage.py sqlmigrate app 0001  # Preview SQL
python manage.py showmigrations       # List all migrations
python manage.py inspectdb            # Generate models from existing DB

# ============ USER MANAGEMENT ============
python manage.py createsuperuser      # Create admin user
python manage.py changepassword user  # Change user password

# ============ SERVER ============
python manage.py runserver            # Start dev server (port 8000)
python manage.py runserver 8080       # Custom port
python manage.py runserver 0:8000     # Accessible from network

# ============ SHELL & TESTING ============
python manage.py shell                # Interactive Python shell
python manage.py test                 # Run all tests
python manage.py test app             # Run app tests

# ============ STATIC FILES ============
python manage.py collectstatic        # Collect static files

# ============ UTILITIES ============
python manage.py check                # Check for issues
python manage.py dbshell              # Database shell
python manage.py flush                # Delete all data
python manage.py dumpdata app > data.json    # Export data
python manage.py loaddata data.json          # Import data
```

---

## 8. Common Field Types

### Model Fields

| Field Type | Description | Example |
|------------|-------------|---------|
| `CharField` | Short text (max_length required) | `name = models.CharField(max_length=100)` |
| `TextField` | Long text | `content = models.TextField()` |
| `IntegerField` | Integer | `count = models.IntegerField(default=0)` |
| `FloatField` | Floating point | `price = models.FloatField()` |
| `DecimalField` | Precise decimal | `amount = models.DecimalField(max_digits=10, decimal_places=2)` |
| `BooleanField` | True/False | `is_active = models.BooleanField(default=True)` |
| `DateField` | Date | `birth_date = models.DateField()` |
| `DateTimeField` | Date and time | `created_at = models.DateTimeField(auto_now_add=True)` |
| `EmailField` | Email address | `email = models.EmailField()` |
| `URLField` | URL | `website = models.URLField()` |
| `SlugField` | URL-friendly string | `slug = models.SlugField(unique=True)` |
| `ImageField` | Image file | `photo = models.ImageField(upload_to='photos/')` |
| `FileField` | Any file | `document = models.FileField(upload_to='docs/')` |
| `ForeignKey` | Many-to-one | `author = models.ForeignKey(User, on_delete=models.CASCADE)` |
| `ManyToManyField` | Many-to-many | `tags = models.ManyToManyField(Tag)` |
| `OneToOneField` | One-to-one | `profile = models.OneToOneField(User, on_delete=models.CASCADE)` |

### Field Options

| Option | Description |
|--------|-------------|
| `null=True` | Allow NULL in database |
| `blank=True` | Allow empty in forms |
| `default=value` | Default value |
| `unique=True` | Must be unique |
| `primary_key=True` | Primary key |
| `choices=CHOICES` | Limit to choices |
| `verbose_name='Name'` | Human-readable name |
| `help_text='Help'` | Help text for forms |
| `db_index=True` | Create database index |
| `editable=False` | Not editable in admin |

---

## 9. Troubleshooting

### Common Issues

**Migration conflicts:**
```bash
# Delete migrations (keep __init__.py)
rm blog/migrations/0*.py

# Recreate migrations
python manage.py makemigrations blog
python manage.py migrate blog --fake-initial
```

**Static files not loading:**
```bash
# Check STATIC_URL in settings.py
# Run collectstatic for production
python manage.py collectstatic
```

**Template not found:**
- Check `TEMPLATES` settings in `settings.py`
- Verify template directory structure: `app/templates/app/template.html`

**Database connection issues:**
```bash
# Test connection
python manage.py dbshell

# Check settings
python manage.py check --database default
```

**Import errors:**
```bash
# Check INSTALLED_APPS in settings.py
# Verify __init__.py exists in app directory
```

---

## 📚 Additional Resources

- [Django Documentation](https://docs.djangoproject.com/)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [Django Debug Toolbar](https://django-debug-toolbar.readthedocs.io/)
- [Django Girls Tutorial](https://tutorial.djangogirls.org/)

---

*Last updated: January 2026*
