---
date: 2025-06-21
tags:
  - kanban
  - project
---
# Django tutorial

> [w3school django tutorial](https://www.w3schools.com/django/django_create_project.php)
> based on django version 5.1.7

## Instaliation

_create env first_
_I did it with conda._

```
$ python -m pip install Django
```

## Create Project

```
$ django-admin startproject mysite
```

## Create App

An app is a web application 
that has a specific meaning in your project, 
like a _home page_, a _contact form_, or a _members database_.

```
python manage.py startapp members
```

### Create app inside subfolder

> [tistory: django app 폴더 서브폴더 안으로 이동시키기](https://min23th.tistory.com/125)

1.  create app folder inside `sub folder` (or just move it)
```
$ mkdir apps/myapp
$ python manage.py startapp myapp apps/myapp
```

2. inside `app folder` there is `apps.py`. edit the `name` value to `{subfolder}.{appname}`
3. inside `settings.py` edit `INSTALLED_APPS` into `{subfolder}.{appname}`
4. inside `urls.py` edit `urlpatterns` into `{subfolder}.{appname}.urls`

## Views

Django views are Python functions 
that take http requests and return http response, 
like HTML documents.

```
from django.shortcuts import render
from django.http import HttpResponse

def members(request):
	return HttpResponse("Hello world!")
```

## Urls

1. Create a `urls.py` inside the `app folder`

```
from django.urls import path
from . import views

urlpatterns = [
	path('members/', views.members, name='members'),
]
```

Now this is specific for the `members` app. 
We have to do some routing in the `root dir`(project dir) as well.

2. add the app inside the `root dir` `urls.py`
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
	path('', include('members.urls')),
	path('admin/', admin.site.urls),
]
```

## Templates

1. Create a `templates` folder inside the `members` folder, and create a HTML file named `myfirst.html` and put any simple html

2. Modify the View
```
from django.http import HttpResponse
from django.template import loader

def members(request):
	template = loader.get_template('myfirst.html')
	return HttpResponse(template.render())
```

## Change Settings

we need to tell Django that
a new app is created.

from `settings.py` look for `INSTALLED_APPS[]` list and add the `members`  app.

## Models

In Django, data is created in objects, called Models, and is actually tables in a database.

1. Inside `models.py` file in the `/members/` folder.
   add a `Member` table by creating a `Member class`, and describe the table fields in it

```
from django.db import models

class Member(modles.Model):
	firstname = models.CharField(max_length=255)
	lastname = models.CharField(max_length=255)
```

2. run a command to create table in DB
```
python manage.py makemigrations members
```

Django creates a file describing the changes and stores the file in the `/migrations/` folder.
The table is not created yet, you will have to run a command.
Then Django will create and execute an SQL statement, based on the content of the new file in the `/migrations/` folder.

```
python manage.py migrate
```

3. To review the sql you can run this.
```
python manage.py sqlmigrate members 0001
```