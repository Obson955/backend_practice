Django Setup and Development Guide
Virtual Environment Setup
Windows
# Create virtual environment
python3 -m venv myvenv

# Activate virtual environment
myvenv\Scripts\activate
Linux
# Create virtual environment
python -m venv myvenv

# Activate virtual environment
source myvenv/bin/activate
Installing Django
# Upgrade pip
python -m pip install --upgrade pip

# Install Django (add to requirements.txt)
Django~=5.1.2
Creating a Django Project
Windows
django-admin.exe startproject mysite .
Linux
django-admin startproject mysite .
Starting the Development Server
python manage.py runserver
Core Django Files
| File | Purpose | |------|---------| | manage.py | Command-line utility for managing your site (e.g., starting the web server) | | settings.py | Configuration file for your website (database, languages, etc.) | | urls.py | URL routing - keeps track of all web addresses your site can handle | | models.py | Defines database objects with columns (fields) and rows (data) | | admin.py | Interface to manipulate database data (add, delete, edit) |

Development Steps
Step 1: Create an Application
python manage.py startapp blog
Important: After creating an application, add it to INSTALLED_APPS in settings.py

INSTALLED_APPS = [
    ...
    'blog',
]
Step 2: Create Models
Models are defined using classes and represent database tables.

class Post(models.Model):
    # Your model fields here
    pass
Key Concepts:

class: Python keyword to define an object
models.Model: Defines a Django model
Models are objects with properties
Common Field Types:

models.CharField - Short text field
models.TextField - Long text field
models.DateTimeField - Date and time field
models.ForeignKey - Relationship to another model
Example Model:

class Post(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    created_date = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.title
Step 3: Database Migrations
After creating or modifying models, sync them with the database:

# Prepare migration file
python manage.py makemigrations blog

# Apply migration to database
python manage.py migrate blog
Step 4: Register Models in Admin Panel
Add your models to admin.py to make them available in the Django admin interface:

from django.contrib import admin
from .models import Post

admin.site.register(Post)
Quick Reference Commands
# Create project
django-admin startproject mysite .

# Create app
python manage.py startapp appname

# Make migrations
python manage.py makemigrations appname

# Apply migrations
python manage.py migrate appname

# Create superuser
python manage.py createsuperuser

# Run development server
python manage.py runserver
