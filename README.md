# backend_practice

command to create virtual environment on windows 
python3 -m venv myvenv

command to create virtual environment on linux
python -m venv myvenv

activate venv on windows 
 myvenv\Scripts\activate

activate venv on linux 
source myvenv/bin/activate

Installing Django
python -m pip install --upgrade pip
Django~=5.1.2 in requirements file 


create Django project on windows
django-admin.exe startproject mysite .


create Django project on linux 
django-admin startproject mysite .


to start webserver 
python manage.py runserver


manage.py is where you manage your site, use to start webserver for example 

settings.py is where configuration of the website is stored (db,languages)

urls.py helps keep track of all the different web addresses your site can handle, so everything is neat and organized.

Models.py  is an object that is save in the db contain columns [fields] and rows [data]

Admin.py use to manipulate your data in the db technically add,delete,edit.


Step 1. 
To create an application
- python manage.py startapp blog 
after created an application it should be added in settings.py under installed apps.


Step 2. 
Create Models, what is included in a models?
class Post(models.Model): to define the models 
class: special programming keyword to define object
models.models: define the django models
Because Models is an object then an object should have his properties, so for example a model can have: 

models.CharField
models.TextField
models.DateTimeField
models.ForeignKey

a model can also has a function to do an action or return something

Step 3. 
After create the models you should add it to the db 
- python manage.py makemigrations "app name"  To prepare migration
- python manage.py migrate blog "app name" To make the actual migration

Step 4.
Make our models available in the django admin panel 

