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

Add the following four lines to `/etc/apache2/sites-enabled/000-default`:

```apache
<VirtualHost *:80>
...
    WSGIDaemonProcess django_demo python-path=/var/www/django_demo:/var/www/django_demo/env/lib/python3.4/site-packages
    WSGIProcessGroup django_demo
    SetEnv DJANGO_DEMO_DB_PASSWORD super_secret_password    # use actual password here
    WSGIScriptAlias /django_demo /var/www/django_demo/django_demo/wsgi.py
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

        os.environ['DJANGO_DEMO_DB_PASSWORD'] = environ['DJANGO_DEMO_DB_PASSWORD']
        os.environ.setdefault("DJANGO_SETTINGS_MODULE", "django_demo.settings")
        django.setup()
        return super(WSGIEnvironment, self).__call__(environ, start_response)

application = WSGIEnvironment()
```
