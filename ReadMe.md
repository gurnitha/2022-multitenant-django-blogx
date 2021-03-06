## Turning Ready Made Blogx Into Django Tenant Blogx

This is my first exercise about DJANGO MULTITENANT based on the tutorial and django-blogx made by Academy Omen on Youtube

My Github repository:
https://github.com/gurnitha/2022-multitenant-django-blogx


#### [View the Tutorial Video](https://youtu.be/J7UOjUshjyY)

-> Download [starter files for this project](https://github.com/Academy-Omen/django-blogx/tree/starter)

-> Create Virtual environment
```bash
# Windows
py -3 -m venv env
# Linux and Mac
python -m venv env
```

-> Activate environment
```bash
# Windows
.\env\Scripts\activate
# Linux and Mac
source env/bin/activate
```

-> Install Requirements
```bash
pip install -r requirements.txt

```

-> Create Django project in the present directory
```bash
# the '.' tells python to create the project in the present folder
django-admin startproject core .

```

-> Create Blog app
```bash
python manage.py startapp blog

```

-> Register blog app in project settings file
```py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # register blog here
    'blog',
]

```

-> Apply migrations
```bash
python manage.py migrate

```

-> Create basic views
```py

from django.shortcuts import render


def home(request):
    return render(request, 'index.html')


def article(request):
    return render(request, 'article.html')

```

-> Create the blog app urls file
```py

from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    path('', views.home, name='homepage'),
    path('post/', views.article, name='article'),
]

```

-> Register the blog urls file in the project urls file
```py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    # import include and add blog urls.py
    path('', include('blog.urls', namespace='blog')),
]
```

-> Create blog app template directory in blog app directory
```html
<!-- Example Home page -->
<h1>Home Page</h1>
```

-> Configure static files in settings file
```py
import os

STATIC_URL = '/static/'
MEDIA_URL = '/media/'

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static')
]

MEDIA_ROOT = os.path.join(BASE_DIR, 'static/images')
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')

```
-> Place the css files in static/css, images in static/images and the html in blog/templates


-> Load the static files in the html files
```html
<!-- place this at top -->
{% load static %}

<!-- example -->
<link rel="stylesheet" href="{% static 'css/style.css' %}"">
```

-> Create Models and register to admin interface

```bash
python manage.py makemigrations
python manage.py migrate
#  create superuser
python manage.py createsuperuser
python manage.py runserver
```
```py

# blog admin.py file
from django.contrib import admin
from . import models


admin.site.register(models.Tag)
admin.site.register(models.Profile)



@admin.register(models.Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ('headline', 'status', 'slug', 'author')
    prepopulated_fields = {'slug': ('headline',), }

```

-> Tell django where to get static files in development
```py
# core.urls.py
from django.conf.urls.static import static
from django.conf import settings

# .
# .

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

-> Add ckeditor to installed app and add the settings
```py
# settings.py
INSTALLED_APPS = [

# .
# .

    'ckeditor',
    'ckeditor_uploader',
]

# CKEditor settigs

CKEDITOR_UPLOAD_PATH = 'uploads/'

CKEDITOR_CONFIGS = {
    'default': {
        'toolbar': 'full',
        'height': 300,
        'width': '100%',
    },
}
```

-> Add ckeditor urls
```py
# core.urls.py
# .
# .

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls', namespace='blog')),
    path('ckeditor/', include('ckeditor_uploader.urls')),
]

```

-> Collect statics all static so as to copy ckeditor required media files
```bash
python manage.py collectstatic
```

-> Add content to your database

-> Update home views and add load data on template

```py

def home(request):

    # feature articles on the home page
    featured = Article.articlemanager.filter(featured=True)[0:3]

    context = {
        'articles': featured
    }

    return render(request, 'index.html', context)
```

-> Update articles views and add load data on template

```py
# Django Q objects use to create complex queries
from django.db.models import Q

def articles(request):

    # get query from request
    query = request.GET.get('query')
    # print(query)
    # Set query to '' if None
    if query == None:
        query = ''

    # articles = Article.articlemanager.all()
    # search for query in headline, sub headline, body
    articles = Article.articlemanager.filter(
        Q(headline__icontains=query) |
        Q(sub_headline__icontains=query) |
        Q(body__icontains=query)
    )

    tags = Tag.objects.all()

    context = {
        'articles': articles,
        'tags': tags,
    }

    return render(request, 'articles.html', context)
```

-> Add get_absolute_url to Article model which will be used to get a single article
```py

# models.py file

    def get_absolute_url(self):
        return reverse('blog:article', args=[self.slug])

    class Meta:
        ordering = ('-publish',)
```

-> Update the blog urls file
```py
# .
# .

urlpatterns = [
    path('', views.home, name='home'),
    path('articles/', views.articles, name='articles'),

    # update the article url
    path('<slug:article>/', views.article, name='article'),
]

# .
# .
```

-> Update articles views and add load data on template
```py

def article(request, article):

    article = get_object_or_404(Article, slug=article, status='published')

    context = {
        'article': article
    }

    return render(request, 'article.html', context)
    
```

-> 1. Clonning Django-Blogx for Multi Tenant Exercise
```bash
.gitignore
.vscode/
LICENSE
ReadMe.md
blog/
core/
db.sqlite3
manage.py
requirements.txt
static/
staticfiles/
```

-> 2. Install django tenants, psycopg2 and update requirements.txt file

```
pip install django-tenants
pip install psycopg2
pip freeze > requirements.txt

modified:   ReadMe.md
modified:   requirements.txt
```

-> 3. Modify Readme.md file
```bash
modified:   ReadMe.md
```

-> 4. Create Postgres database and connect it with the blogx
```py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'django_dukcapildesabojongbaru_new',
        'USERNAME': 'postgres',
        'PASSWORD': 'xxx',
        'HOST': 'localhost',
        'PORT': '5432'
    }
}
```

-> 5. Create superuser
```py
# Create tables
(multitenant) ?? python manage.py makemigrations
(multitenant) ?? python manage.py migrate

# Create superuser
(multitenant) ?? python manage.py createsuperuser
Username (leave blank to use 'hp'): admin
Email address: admin@admin.com
Password:
Password (again):
The password is too similar to the username.
This password is too short. It must contain at least 8 characters.
This password is too common.
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.
```

-> 6. Create new users and add some posts 
```html
        new file:   static/images/article/gladiator.jpg
        new file:   static/images/article/limitless.JPG
        new file:   static/images/article/luther.JPG
        new file:   static/images/profile/darling.PNG
        new file:   static/images/profile/ing.PNG
```

-> 7. Add DATABASE_ROUTERS

```py
# DATABASE ROUTER
DATABASE_ROUTERS = (
    'django_tenants.routers.TenantSyncRouter',
)
```

-> 8. Setup Middleware

```py
MIDDLEWARE = [
    # add this at the top
    # django tenant middleware
    'django_tenants.middleware.main.TenantMainMiddleware',
```

-> 9. Create tenant app and add tenant app to settings.py

```py
# Create tenant app
(multitenant) ?? python manage.py startapp tenant

# Add tentant app to settings.py
INSTALLED_APPS = [
    ...

    'blog',
    'tenant',

    'ckeditor',
    'ckeditor_uploader',
]

# new files
        modified:   .gitignore
        modified:   core/__pycache__/settings.cpython-39.pyc
        modified:   core/settings.py
        new file:   tenant/__init__.py
        new file:   tenant/admin.py
        new file:   tenant/apps.py
        new file:   tenant/migrations/__init__.py
        new file:   tenant/models.py
        new file:   tenant/tests.py
        new file:   tenant/views.py
```

-> 10. Add DATABASE_ROUTERS (repeating step no. 7 because of mistaken)

```py
# DATABASE ROUTER
DATABASE_ROUTERS = (
    'django_tenants.routers.TenantSyncRouter',
)
```

-> 11. Modified db engine, create Tenant model and run migration

```py
# Change db engine
DATABASES = {
    'default': {
        'ENGINE': 'django_tenants.postgresql_backend',
        # 'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'multitenant_django_blogx_2022',
        'USERNAME': 'postgres',
        'PASSWORD': 'ing',
        'HOST': 'localhost',
        'PORT': '5432'
    }
}
# Create Tenant model
from django.db import models
from django.contrib.auth.models import User
from django_tenants.models import DomainMixin, TenantMixin


class Tenant(TenantMixin):
    user = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
    blog_name = models.CharField(max_length=50)
    blog_image = models.ImageField(null=True, blank=True, upload_to="profile")

    featured = models.BooleanField(default=False)
    updated_at = models.DateTimeField(auto_now=True, null=True)

    description = models.TextField(blank=True)

    is_active = models.BooleanField(default=False, blank=True)
    created_on = models.DateField(auto_now_add=True)

    # default true, schema will be automatically created and
    # synced when it is saved
    auto_create_schema = True

    """
    USE THIS WITH CAUTION!
    Set this flag to true on a parent class if you want the schema to be
    automatically deleted if the tenant row gets deleted.
    """
    auto_drop_schema = True


    class Meta:
        ordering = ('-featured', '-updated_at')

    def __str__(self):
        return f"{self.blog_name}"


class Domain(DomainMixin):
    pass

# Run migration
# create migrations files
python manage.py makemigrations
# You may need to run migrations for specific app
python manage.py makemigrations blog
# Apply migrations
python manage.py migrate_schemas

# New/modified files
        modified:   ReadMe.md
        modified:   core/__pycache__/settings.cpython-39.pyc
        modified:   core/settings.py
        modified:   tenant/admin.py
        new file:   tenant/migrations/0001_initial.py
        modified:   tenant/models.py
```

-> 12. Setup Initial User, Tenant and Admin

```py
# create first user
python manage.py createsuperuser

# Create the Public Schema
(multitenant) ?? python manage.py create_tenant
schema name: public
user: 1
blog name: Main
blog image:
featured:
description:
is active: True
domain: localhost
is primary (leave blank to use 'True'):

# Create the Administrator
(multitenant) ?? python manage.py create_tenant_superuser
Enter Tenant Schema ('?' to list schemas): ?
public - localhost
Enter Tenant Schema ('?' to list schemas): public
Username (leave blank to use 'hp'): admin1
Email address: admin1@email.com
Password:
Password (again):
The password is too similar to the username.
This password is too short. It must contain at least 8 characters.
This password is too common.
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.

# Run server
python manage.py runserver

# Logout and login as admin1
Go to: http://127.0.0.1:8000/admin/ and logout then login as admin1

# New/changed files
        modified:   ReadMe.md
```

-> 13. Create a custom middleware

```py
from django_tenants.middleware.main import TenantMainMiddleware


class TenantMiddleware(TenantMainMiddleware):
    """
    Field is_active can be used to temporary disable tenant and
    block access to their site. Modifying get_tenant method from
    TenantMiddleware allows us to check if tenant should be available
    """
    def get_tenant(self, domain_model, hostname):
        tenant = super().get_tenant(domain_model, hostname)
        if not tenant.is_active:
            raise self.TENANT_NOT_FOUND_EXCEPTION("Tenant is inactive")
        return tenant

```

-> 14. Add the middleware and re-peat poin 8 (mistaken)

```py
    # add this at the top
    # django tenant middleware
    'django_tenants.middleware.main.TenantMainMiddleware',
    # custom tenant middleware
    'core.middleware.TenantMiddleware',
```

-> 15. Created blog1 schema from tenant's admin panel

```py
# Check in the db you will see db schema like bellow:
multitenant_django_blogx_2022
> blog1
> public
# NOTE:
1. I created a new post, but it was display on the tenant's page.
2. To make the owner of blog1 to publish posts, he must first
   create superuser.
```

-> 16. CreatE superuser for blog1, login and create posts

```py
# Create superuser
(multitenant) ?? python manage.py create_tenant_superuser
Enter Tenant Schema ('?' to list schemas): ?
blog1 - blog1.localhost
public - localhost
Enter Tenant Schema ('?' to list schemas): blog1
Username (leave blank to use 'hp'): adminblog1
Email address: adminblog1@email.com
Password:(adminblog1)
Password (again):(adminblog1)
The password is too similar to the username.
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.

# Login as admin of blog1
Go to: http://blog1.localhost:8000/admin
Username and password: adminblog1

# Create post
Create some post 

# Testing
Go to: http://blog1.localhost:8000/
Refresh the browser
DONE :)
```

-> 17. Create a new db schema 'blog2'
```
STEPS:

1. Parent creates a new tenant, named 'blog2'
2. Activate
3. Create: create_tenant_superuser
4. Login
5. Create posts
6. Run the server, refresh browser

NOTE: Step 3 --> see poin 16 above

# Create superuser
(multitenant) ?? python manage.py create_tenant_superuser
Enter Tenant Schema ('?' to list schemas): ?
blog1 - blog1.localhost
blog2 - blog2.localhost
public - localhost
Enter Tenant Schema ('?' to list schemas): blog2
Username (leave blank to use 'hp'): adminblog2
Email address: adminblog2@email.com
Password:
Password (again):
Error: Your passwords didn't match.
Password:
Password (again):
The password is too similar to the username.
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.

# New/modified files
        modified:   ReadMe.md
        new file:   static/images/article/darling.PNG
        new file:   static/images/profile/postgresql-card.png
```