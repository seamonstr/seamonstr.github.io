---
title: First steps with Flask
---

Flask is a lightweight web framework. I'm using it for my snoop project because snoop is going to be SPA, and Flask seems better suited to APIs than Django. Note that it's a framework; you still need a WSGI server to host it.

Zoopla has switched to using FastAPI rather than Flask, but Flask seems to have a bigger following, so I'll go with that.

## Installation, first app 

* Install "flask".
* Minimum flask app is as follows:
    ```python
    from flask import Flask

    app = Flask(__name__)

    @app.route("/")
    def hello_world():
    
    return "<p>Hello, World!</p>"
    ```
    * Create a flask app object, then use the object's `.route` method as a decorator to declare routes.
* Run it from the command line with `flask run`.
    * Naming the main app file `app.py` or `wsgi.py` means that flask will autodiscover it.
    * Otherwise, have to use `flask --app <python module> run`
    * Serves by default on port 5000.

* Enable debug with `--debug` - will watch sources and rebuild on mod.
    * includes a built-in debugger from `wekzeug` - will show tracebacks in the page if something blows up, and gives you a console to dick about with it.

## Routing

* Grab values out of the url as follows:
    ```python
    @app.route('/user/<username>')
    def show_user_profile(username):
        # show the user profile for that user
        return f'User {escape(username)}'
    ```
* There is a framework of converters to use on these values:
    ```python
    @app.route('/post/<int:post_id>')
    def show_post(post_id:int):
        # show the post with the given id, the id is an integer
        return f'Post {post_id}'

    @app.route('/path/<path:subpath>')
    def show_subpath(subpath:str):
        # show the subpath after /path/; like "str", but accepts slashes
        return f'Subpath {escape(subpath)}'
    ```

## User model

Database models can be managed in Flask using SQLAlchemy. You use it by installing flask_sqlalchemy (which provides you the ORM and a SQL-in-python mechansim) and flask_migration (which provides an integration with Alembic; gives you automatic generation of migration & rollback scripts).

There's a peculiarity of the design here.  You need to do the following:
1. Create a `db` object of type `flask_sqlalchemy.SQLAlchemy`
1. Create model classes that subclass `db.Model` - a generated class on your `db` object.
1. Import the model classes into your app so that the migration mechanism can see them - it scans for classes that are subclasses of `db.Model`.

The obvious thing is to create code in an `app.py` like:
```python
from flask_sqlalchemy import SQLAlchemy
import my_models # Issue!

db = SQLAlchemy()
```

The issue above is that the model classes need to subclass the generated class that is only initialised when you initialise the `db` object, so your `my_models.py` file has to look like:
```python
import app # Issue!

class MyModel(app.db.Model):
    # stuff
    pass
```

...and the above two files have got a lovely circular dependency, which is designed-in.

The best solution I found is to have a third, `db.py` file which does nothing but create that `db` object:
```python
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()
```

...and then import that into both `app.py` and `models.py` above.

So, in your `app.py`, you need the following:

```python
from .db import db

migrate = Migrate()

# Flask autodiscover looks for this factory method name
def create_app(): 
    app = Flask(__name__)
    # Need at least SQLALCHEMY_DATABASE_URI & SQLALCHEMY_TRACK_MODIFICATIONS as
    # class attributes on Config
    app.config.from_object(config.Config)
    # This pattern causes the extension objects (db & migrate) to inject config
    # into the app, enabling one global extension for multiple apps
    db.init_app(app)
    migrate.init_app(app, db)
```

### Initialise

The extensions can extend Flask's shell CLI, and the `migrate` one adds:
* `flask db init` - creates the database, and the directory in the source tree for the migration/rollback files
* `flask db migrate` - analyse current sources and figure out how to modify the db to the new schema
* `flask db upgrade` - run the upgrade scripts needed to bring the DB up to current version.

There are others: 

```
Usage: flask db [OPTIONS] COMMAND [ARGS]...

  Perform database migrations.

Options:
  --help  Show this message and exit.

Commands:
  branches        Show current branch points
  current         Display the current revision for each database.
  downgrade       Revert to a previous version
  edit            Edit a revision file
  heads           Show current available heads in the script directory
  history         List changeset scripts in chronological order.
  init            Creates a new migration repository.
  list-templates  List available templates.
  merge           Merge two revisions together, creating a new revision file
  migrate         Autogenerate a new revision file (Alias for 'revision')
  revision        Create a new revision file.
  show            Show the revision denoted by the given symbol.
  stamp           'stamp' the revision table with the given revision;...
  upgrade         Upgrade to a later version
```

### SQLAlchemy data querying & writing

If you have a model class, you get a free constructor that takes attributes named after the fields you created:

```python
class User(db.Model, UserMixin):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False, index=True)
    pwd = db.Column(db.String(300), nullable=False, unique=True)

my_user = User(username="fred", pwd="Boo")
```

You can get a shell with your Flask app loaded where you can access your model and the database:

```sh
poetry run flask --app backend.app shell
```

In there, you can do:

```python
>>>from backend.db import db
>>>from backend.models import User
>>> my_user = User(username="fred", pwd="Boo")
>>> db.session.add(my_user)
>>> db.session.commit()
>>> User.query.get(1) 
<User fred@redcabbage.org>
```

More [interesting bits you can do are here](https://flask-sqlalchemy.palletsprojects.com/en/2.x/quickstart/).