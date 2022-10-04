---
title: Stuff to remember about Django
---

# Stuff to remember about Django

## Installation

* Install with pip: `pip install django; pip install djangorestframework`
* Django has a CLI with `django-admin` in python/bin

## Create a project

Create a project (the overall containing Django project) with `django-admin startproject <name>`. Will create 
* The project is a container for the indiviual apps
* Apps are WSGI callables that can be provided by django & other third parties
* ...and obv by us

The project directory is created at `./<name>`; it's a Python package containing:
* The ASGI and WSGI applications for the project
* A settings.py module for all the apps & middleware installed in the project, as well as other globale settings (BASEDIR, etc)

A ./manage.py file will be created: a script for adding apps to the project, building new module migrations, etc.

## Create an app

From the project's root directory (where `manage.py` is), do `python manage.py startapp <appname>`
* Creates a package `<appname>` off of `.` (sibling to the project package)
* Contains default config for the app: urls (used to map urls relative to the app), models (db config), etc

## Mapping URLs 

URLs are mapped at project level and at app level:
* `<projectdir>/urls.py` has a `urlpatterns` list of `django.urls.Path` objects used to publish the URLs from within the apps.
* Likewise, each app has a (by convention; can be any package name) `urls.py` which has two members: an `app_name` and a list of Path objects `urlpatterns`.

Generally speaking, each `Path` maps a url to one of:
1. *For the project level mapping:* the app name and the list of sub-URLs.  These are supplied by calling `django.urls.include` pass the name of the python module for the app.  The module has a `url_patterns` list of its own, and an `app_name` that the `include` call looks for; it passes those in a tuple to the Path's constructor.
1. *For the app level mapping:* A callable which is the view renderer, as declared in the apps `views.py`. The callable is called with a HttpRequest obj plus some kwargs from the route. Returns a HttpResponse obj. Also takes kwargs (from the URL?) and a name to use when referring to the URL fromm elsewhere. Can be passed by calling `include`, but you can just pass the callable instead.

So:

```
project
  manage.py
  /projectkg
    /urls.py
      urlpatterns - [Path] mapping strings to modules
  /appmodule
    /urls.py # Needs creating!
      urlpatterns - [Path] mapping strings to view callables
    /views.py # Contain the view callables

```

## Set up the DB and add some models

