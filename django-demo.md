# Django Demo Project

## Start Django Project

```sh
PROJECT_NAME=django_demo
PROJECT_DIR=$HOME/git.repos/$PROJECT_NAME

mkdir -p $PROJECT_DIR
cd $PROJECT_DIR
virtualenv -p /usr/local/opt/python-3.4.2/bin/python3.4 env
source env/bin/activate

pip install django
django-admin startproject $PROJECT_NAME .

pip install psycopg2    # To allow interaction with PostgreSQL as DB
```

## Place project under version control

```sh
cd $PROJECT_DIR
git init
git flow init

curl https://raw.githubusercontent.com/github/gitignore/master/Python.gitignore > .gitignore
git add .gitignore
git commit -m "Populate .gitignore with template from github/gitignore"

git add .
git commit -m "Create Django project $PROJECT_NAME"

pip freeze > requirements.txt
git add requirements.txt
git commit -m "Add python package requirements"
```

## Create clone of project within `/var/www/`

```sh
sudo su - root
PROJECT_NAME=django_demo
PROJECT_DIR_PRODUCTION=/var/www/$PROJECT_NAME

mkdir $PROJECT_DIR_PRODUCTION
chown :www-data $PROJECT_DIR_PRODUCTION
chmod g+s $PROJECT_DIR_PRODUCTION

git clone --origin mfc-local /home/mfc/git.repos/django_demo/.git $PROJECT_DIR_PRODUCTION

cd $PROJECT_DIR_PRODUCTION
virtualenv -p /usr/local/opt/python-3.4.2/bin/python3.4 env
source env/bin/activate

pip install -r requirements.txt
```

## Create and configure PostgreSQL database

Create database and admin

```sh
sudo su - postgres
createdb django_demo_db
createuser -P
# Enter name of role to add: django_demo_admin
# Enter password for new role: 
# Enter it again: 
# Shall the new role be a superuser? (y/n) n
# Shall the new role be allowed to create databases? (y/n) n
# Shall the new role be allowed to create more new roles? (y/n) n
psql
```

Grant privileges

```sql
GRANT ALL PRIVILEGES ON DATABASE django_demo_db TO django_demo_admin;
\q
```

Logout postgres user

```sh
exit
```

## Configure project to use PostgreSQL database

Set the project's database to the PostgreSQL database `django_demo_db` in `~/git.repos/django_demo/django_demo/settings.py` by removing this:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

And adding this:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'django_demo_db',
        'USER': 'django_demo_admin',
        # password will passed from environmental variable:
        'PASSWORD': os.environ.get("DJANGO_DEMO_DB_PASSWORD", ''),
        'HOST': '127.0.0.1',
        'PORT': '5432', 
    }   
}      
```

## Configure demo site in Apache and set DB password

Make changes as shown in the section on [Passing environmental variables to Django project](django-env-vars.md).

Add the following four lines to `/etc/apache2/sites-enabled/000-default` (with the actual password and secret key instead of 'super_secret_password' and 'super_secret_key'):

```apache
<VirtualHost *:80>
...
    WSGIDaemonProcess django_demo python-path=/var/www/django_demo:/var/www/django_demo/env/lib/python3.4/site-packages
    <Location /django_demo>
        WSGIProcessGroup django_demo
    </Location>
    SetEnv DJANGO_DEMO_SECRET_KEY super_secret_key
    SetEnv DJANGO_DEMO_DB_PASSWORD super_secret_password
    WSGIScriptAlias /django_demo /var/www/django_demo/django_demo/wsgi.py
    Alias /static/django_demo /var/www/django_demo/static/
...
</VirtualHost>
```

Change `$PROJECT_DIR/$PROJECT_NAME/wsgi.py` (`/home/mfc/git.repos/django_demo/django_demo/wsgi.py`) to:

```python
"""
WSGI config for django_demo project.

It exposes the WSGI callable as a module-level variable named ``application``.

For more information on this file, see
https://docs.djangoproject.com/en/1.7/howto/deployment/wsgi/
"""

from django.core.handlers.wsgi import WSGIHandler
import django
import os

class WSGIEnvironment(WSGIHandler):

    def __call__(self, environ, start_response):

        os.environ['DJANGO_DEMO_SECRET_KEY'] = environ['DJANGO_DEMO_SECRET_KEY']
        os.environ['DJANGO_DEMO_DB_PASSWORD'] = environ['DJANGO_DEMO_DB_PASSWORD']
        os.environ.setdefault("DJANGO_SETTINGS_MODULE", "django_demo.settings")
        django.setup()
        return super(WSGIEnvironment, self).__call__(environ, start_response)

application = WSGIEnvironment()
```

## Pull changes from development repository to production repository

This is a reminder to make changes and commits to the git repo at `$PROJECT_DIR` and then pull those changes to the repo at `/var/www/$PROJECT_NAME`:

```sh
sudo su - root
PROJECT_NAME=django_demo
PROJECT_DIR_PRODUCTION=/var/www/$PROJECT_NAME

BRANCH=develop
REMOTE=mfc-local

cd $PROJECT_DIR_PRODUCTION
git fetch $REMOTE
git checkout $BRANCH
git merge $REMOTE/$BRANCH
exit
```

## Collect static files

Change `STATIC_URL` and add `STATIC_ROOT` to `$PROJECT_DIR/$PROJECT_NAME/settings.py`:

```sh
STATIC_URL = '/static/django_demo/'    # allows static files for multiple projects
STATIC_ROOT = os.path.join(BASE_DIR, 'static')    # 'static' dir at top of project
```

After gitignoring `/static/` and pulling the changes to the production repo, collect the static files. This can be done whenever there are changes/additions to the static files.

```sh
sudo su - root
PROJECT_NAME=django_demo
PROJECT_DIR_PRODUCTION=/var/www/$PROJECT_NAME

cd $PROJECT_DIR_PRODUCTION
source env/bin/activate
source secrets.txt
./manage.py collectstatic
exit
```

## Hide secret key

Change the `SECRET_KEY` variable in `~/git.repos/django_demo/django_demo/settings.py` from this:

```python
SECRET_KEY = '9=*awu9wtsgz7*7&)zj_+l&pw+m1=uwboiqr3m&#vka&z@)a5*'
```

to this:

```python
SECRET_KEY = os.environ.get("DJANGO_DEMO_SECRET_KEY", '')
```

The new secret key was made using an [online generator](http://www.miniwebtool.com/django-secret-key-generator/).

## Add allowed host and turn off debug mode

Change the `DEBUG` and `ALLOWED_HOSTS` variables in `~/git.repos/django_demo/django_demo/settings.py` from this:

```python
DEBUG = True
...
ALLOWED_HOSTS = []
```

to this:

```python
DEBUG = False
...
ALLOWED_HOSTS = ['.plb.ucdavis.edu', ]
```

## Create a secret file to hold secrets

In order to do migrations (and some other things), the secret key and the database password need to be set as environmental variables. However, the way we are passing them from Apache's configuration to `settings.py` via `wsgi.py` prevents them from being set for everyday use.

We can get around this by placing an `export` statement for each secret variable into a file (`secrets.txt` located in the production directory) that is only readable by root. This is file will be gitignored.

```sh
sudo su - root
SECRETS_FILE=/var/www/django_demo/secrets.txt

echo 'export DJANGO_DEMO_SECRET_KEY="<super_secret_key>"
export DJANGO_DEMO_DB_PASSWORD="<super_secret_password>"' > $SECRETS_FILE

chmod 600 $SECRETS_FILE
exit
```

Now, whenever someone needs to set these secret environmental variables, they just need to run this:

```sh
eval `sudo cat secrets.txt`
```

## Migrate database and create super user

To perform the initial database migration, we do the following:

```sh
cd /var/www/django_demo/
eval `sudo cat secrets.txt`
./manage.py migrate
```

>     Operations to perform:
>       Apply all migrations: auth, contenttypes, admin, sessions
>     Running migrations:
>       Applying contenttypes.0001_initial... OK
>       Applying auth.0001_initial... OK
>       Applying admin.0001_initial... OK
>       Applying sessions.0001_initial... OK

And we need to create a super user:

```sh
./manage.py createsuperuser
```

>     Username (leave blank to use 'mfc'): 
>     Email address: mfcovington@gmail.com
>     Password: 
>     Password (again): 
>     Superuser created successfully.

Now we can login to the [demo site](http://symposium.plb.ucdavis.edu/django_demo/admin/).
