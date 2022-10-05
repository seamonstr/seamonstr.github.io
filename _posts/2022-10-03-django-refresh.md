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
      urlpatterns - [Path] mapping strings to app_name and urlpatterns (via calls to include())
  /appmodule
    /urls.py # Needs creating!
      urlpatterns - [Path] mapping strings to view callables
      app_name - name of the app to be published
    /views.py # Contain the view callables

```

## Set up the DB and add some models
Settings for the database are in `<projdir>/settings.py`, in the `DATABASES` list. The apps in the `INSTALLED_APPS` list already require some tables, so there's a migration pending. 
 
*Apply* the data migration by doing `python manage.py migrate`. This applies the migration code that has already been generated. Adds a load of tables and things to the DB. 

Using `sqlite3 db.sqlite3` opens a DB client; doing `.tables` lists the tables.  [SQLite syntax is here](https://www.sqlite.org/lang.html). 

The DB schema for the app is derived from the model classes defined in `<app>/models.py`:
* Each class is a `django.db.models.Model` subclass.
* Define members of some subclass of `django.db.models.Field`: `django.db.models.CharField`, `...IntegerField`, `...DateTimeField`, `ForeignKey` directly in the class
* Metadata supplied in the constructor as kwargs: max_length, default.
* `ForeignKey` takes a type that is the destination of the foreign key relationship; also a `on_delete = models.CASCADE`, or other value.
* Do `python manage.py makemigrations` to generate the migration code the for the new models. Migration code for an app is in `<appdir>/migrations/*.py`
* Can see the sql that would be generated for these classes by `python migrate.py sqlmigration helloworld 0001`. `0001` is the index of the migration to generate for.
* You don't have to specify identifiers; Django does this automatically, and creates foreign key fields too

## Django data API
* Can get a django shell with `python manage.py shell`
* Import the models: `from <app>.models import Blah, Bloo`
* `Blah.objects` is a manager object that is query interface.
* Variety of cool queries you can do:
  * `Blah.objects.all()` - QuerySet of everything
  * `Blah.objects.get(pk = 1)`
  * Create a new one: `b = Blah(field_1 = 'Yes', field_2 = 'more!'); b.save()`
* Model objects with a ForeignKey field generate a matching attribute on the other side of the relationship. `person1.address` has a corresponding `address1.person_set` which is a type of manager thing (so you can do queries & filters on that too).
* Relationship spanning queries has a [weird expression language](https://docs.djangoproject.com/en/4.1/topics/db/queries/#lookups-that-span-relationships) to filter the items across the relationship, and span further relationships.  [Another friendly article here](https://docs.djangoproject.com/en/4.1/intro/tutorial02/#playing-with-the-api).

## Django Authentication
* Implemented in the `django.contrib.auth` app; installed by default in a Django project
* Uses django's standard models approach; `django.contrib.auth.models.User` is the main class.
* `User.objects` is a manager, but it's a UserManager thing.
* `User.objects.create_user(name, email, passwd)`
* ...also `create_superuser`, same args
* Superuser creation available from the shell with `python -m manage.py createsuperuser`
* Low level call to authentication: `django.contrib.auth.authenticate(username="Simon" password="mypasswd")` -> returns `User` object if okay, or `None`

All model types in Django have a set of CRUD permissions created for them that can be assigned to users and checked with `User.has_perm('<app_name>.<add|view|change|delete>_ModelClass)`

Can add custom permissions to a model type by using a `Meta` class, and giving it a `permissions` list: list of tuples of (perm_name, description). They get created on the next migrate.

`django.contrib.contenttypes.models.ContentType` is a django model used internally to manage the generated model classes. We need to use it to programmatically add permissions:
```python
content_type = ContentType.objects.get_for_model(BlogPost)
    permission = Permission.objects.get(
        content_type=content_type,
        codename='change_blogpost',
    )
    user.user_permissions.add(permission)
```
## Django Sessions
* Ensure the project has the session middleware `django.contrib.sessions.middleware.SessionMiddleware` and app `django.contrib.sessions` set in settings.
* Every request object will now get a `.session` attribute; a dict of values.  It contains session expiries, etc.
* By default the session is stored in the DB; backends are available for redis & filesystem

## Standard ER model extensions to support object-type constructs (like subtyping/supertyping & aggregation):
* Specialisation & Generalisation: use an additional table to act as the subclass, and hold the specialisation-specific data. In Django, it'll be handled by the model system just by subclassing the parent class.
* Aggregation: Sometimes data belongs on the relationship between two things. Eg: Person -> DoesJob -> Job: "DoesJob" could have related set of records to the equipment issued to do the job; so, create a table for the equipment and relate it to the associative table.

## Random points of interest

* Deferrable foreign key constraints are used when FK constraints make life awkward. Eg. You have Teacher and Class, and Class has a unique Teacher constraint and a non-NULL teacher constraint. How do you move teachers?  Defer the constraint to the end of the transaction. Also, where you want to totally load a child table and then a parent table in a batch. *BUT:* the DB knowing that a column can be enforced unique can let it do optimisations, so deferring constraints can slow things down.
* `time.time()` returns a float time of seconds since epoch; `time.time_ns()` is nanos since epoch. `time.strftime()` formatted time in utc; %Y-%M-%D %H:%M:%s
* `datetime.datetime` is a class that's more colloquial. 
  *  datetime.today() is the same as datetime.fromtimestamp(time.time()) -> takes a Posix date (secs since epoch). This is a *naive* date because it has no associated timezone; saving in django model will result in a warning and an assumption of UTC.
  *  datetime.now(datetimetimezone.utc) is preferred - has a timezone.
