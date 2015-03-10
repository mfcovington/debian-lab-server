# django CMS Demo Project

## Start django CMS Project

```sh
PROJECT_NAME=django_cms_demo
PROJECT_DIR=$HOME/git.repos/$PROJECT_NAME

mkdir -p $PROJECT_DIR
cd $PROJECT_DIR
virtualenv -p /usr/local/opt/python-3.4.2/bin/python3.4 env
source env/bin/activate

pip install djangocms-installer
djangocms -s -p . $PROJECT_NAME
# Database configuration (in URL format) [default sqlite://localhost/project.db]: 
# django CMS version (choices: 2.4, 3.0, stable, develop) [default stable]: 3.0
# Django version (choices: 1.4, 1.5, 1.6, 1.7, stable) [default stable]: 1.7
# Activate Django I18N / L10N setting (choices: yes, no) [default yes]: no
# Install and configure reversion support (choices: yes, no) [default yes]: 
# Languages to enable. Option can be provided multiple times, or as a comma separated list. Only language codes supported by Django can be used here: en-us
# Optional default time zone [default America/Chicago]: America/Los_Angeles
# Activate Django timezone support (choices: yes, no) [default yes]: 
# Activate CMS permission management (choices: yes, no) [default yes]: 
# Use Twitter Bootstrap Theme (choices: yes, no) [default no]: 
# Use custom template set [default no]: 
# ...
# Creating admin user
# Username (leave blank to use 'mfc'):    
# Email address: mfcovington@gmail.com
# Password: 
# Password (again): 
# Superuser created successfully.
```

Check versions:

```sh
python -c '
from cms import __version__
import django
import sys

print("django CMS: %s" % __version__)
print("Django:     %s" % django.get_version())
print("Python:     %s" % sys.version)
'
```

>     django CMS: 3.0.12
>     Django:     1.7.6
>     Python:     3.4.2 (default, Feb 10 2015, 23:06:53) 
>     [GCC 4.7.2]

## Place project under version control

```sh
cd $PROJECT_DIR
git init
git flow init -d

curl https://raw.githubusercontent.com/github/gitignore/master/Python.gitignore > .gitignore
git add .gitignore
git commit -m "Populate .gitignore with template from github/gitignore"

git add .
git commit -m "Create django CMS project $PROJECT_NAME"

pip freeze > requirements.txt
git add requirements.txt
git commit -m "Add python package requirements"
```

## Create clone of project within `/var/www/`

```sh
sudo su - root
PROJECT_NAME=django_cms_demo
PROJECT_DIR_DEVELOPMENT=/home/mfc/git.repos/$PROJECT_NAME
PROJECT_DIR_PRODUCTION=/var/www/$PROJECT_NAME

mkdir $PROJECT_DIR_PRODUCTION
chown :www-data $PROJECT_DIR_PRODUCTION
chmod g+s $PROJECT_DIR_PRODUCTION

git clone --origin mfc-local $PROJECT_DIR_DEVELOPMENT/.git $PROJECT_DIR_PRODUCTION

cd $PROJECT_DIR_PRODUCTION
virtualenv -p /usr/local/opt/python-3.4.2/bin/python3.4 env
source env/bin/activate

pip install -r requirements.txt
pip install psycopg2    # To allow interaction with PostgreSQL as DB

exit
```

Confirm versions:

```sh
python -c '
from cms import __version__
import django
import sys

print("django CMS: %s" % __version__)
print("Django:     %s" % django.get_version())
print("Python:     %s" % sys.version)
'
```

>     django CMS: 3.0.12
>     Django:     1.7.6
>     Python:     3.4.2 (default, Feb 10 2015, 23:06:53) 
>     [GCC 4.7.2]

## Create and configure PostgreSQL database

Create database and admin

```sh
sudo su - postgres
createdb django_cms_demo_db
createuser -P
# Enter name of role to add: django_cms_demo_admin
# Enter password for new role: 
# Enter it again: 
# Shall the new role be a superuser? (y/n) n
# Shall the new role be allowed to create databases? (y/n) n
# Shall the new role be allowed to create more new roles? (y/n) n
psql
```

Grant privileges

```sql
GRANT ALL PRIVILEGES ON DATABASE django_cms_demo_db TO django_cms_demo_admin;
\q
```

Logout postgres user

```sh
exit
```

## Configure project to use PostgreSQL database

Set the project's database to the PostgreSQL database `django_cms_demo_db` in `~/git.repos/django_cms_demo/django_cms_demo/settings.py` by removing this:

```python
DATABASES = {
    'default':
        {'NAME': 'project.db', 'PORT': '', 'PASSWORD': '', 'HOST': 'localhost', 'USER': '', 'ENGINE': 'django.db.backends.sqlite3'}
}
```

And adding this:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'django_cms_demo_db',
        'USER': 'django_cms_demo_admin',
        # password will passed from environmental variable:
        'PASSWORD': os.environ.get("DJANGO_CMS_DEMO_DB_PASSWORD", ''),
        'HOST': '127.0.0.1',
        'PORT': '5432', 
    }   
}
```

Remove SQLite3 database:

```sh
rm $PROJECT_DIR/project.db
```

## Configure demo site in Apache and set DB password

Make changes as shown in the section on [Passing environmental variables to Django project](django-env-vars.md).

Add the following four lines to `/etc/apache2/sites-enabled/000-default` (with the actual password and secret key instead of 'super_secret_password' and 'super_secret_key'):

```apache
<VirtualHost *:80>
...
    WSGIDaemonProcess django_cms_demo python-path=/var/www/django_cms_demo:/var/www/django_cms_demo/env/lib/python3.4/site-packages
    <Location /django_cms_demo>
        WSGIProcessGroup django_cms_demo
    </Location>
    SetEnv DJANGO_CMS_DEMO_SECRET_KEY super_secret_key
    SetEnv DJANGO_CMS_DEMO_DB_PASSWORD super_secret_password
    WSGIScriptAlias /django_cms_demo /var/www/django_cms_demo/django_cms_demo/wsgi.py
    Alias /static/django_cms_demo /var/www/django_cms_demo/static/
...
</VirtualHost>
```

Change `$PROJECT_DIR/$PROJECT_NAME/wsgi.py` (`/home/mfc/git.repos/django_cms_demo/django_cms_demo/wsgi.py`) to:

```python
"""
WSGI config for django_cms_demo project.

It exposes the WSGI callable as a module-level variable named ``application``.

For more information on this file, see
https://docs.djangoproject.com/en/1.7/howto/deployment/wsgi/
"""

from django.core.handlers.wsgi import WSGIHandler
import django
import os

class WSGIEnvironment(WSGIHandler):

    def __call__(self, environ, start_response):

        os.environ['DJANGO_CMS_DEMO_SECRET_KEY'] = environ['DJANGO_CMS_DEMO_SECRET_KEY']
        os.environ['DJANGO_CMS_DEMO_DB_PASSWORD'] = environ['DJANGO_CMS_DEMO_DB_PASSWORD']
        os.environ.setdefault("DJANGO_SETTINGS_MODULE", "django_cms_demo.settings")
        django.setup()
        return super(WSGIEnvironment, self).__call__(environ, start_response)

application = WSGIEnvironment()
```

## Pull changes from development repository to production repository

This is a reminder to make changes and commits to the git repo at `$PROJECT_DIR` and then pull those changes to the repo at `/var/www/$PROJECT_NAME`:

```sh
sudo su - root
PROJECT_NAME=django_cms_demo
PROJECT_DIR_PRODUCTION=/var/www/$PROJECT_NAME

BRANCH=develop
REMOTE=mfc-local

cd $PROJECT_DIR_PRODUCTION
git fetch $REMOTE
git checkout $BRANCH
git merge $REMOTE/$BRANCH
exit
```

## Hide secret key

Change the `SECRET_KEY` variable in `~/git.repos/django_cms_demo/django_cms_demo/settings.py` from this:

```python
SECRET_KEY = 'qa(2+^2f*)z6ubf5y8j@y)(+#qm(_v$%%od+b!_2o-3syouln8'
```

to this:

```python
SECRET_KEY = os.environ.get("DJANGO_CMS_DEMO_SECRET_KEY", '')
```

The new secret key was made using an [online generator](http://www.miniwebtool.com/django-secret-key-generator/).

## Create a secret file to hold secrets

In order to do migrations (and some other things), the secret key and the database password need to be set as environmental variables. However, the way we are passing them from Apache's configuration to `settings.py` via `wsgi.py` prevents them from being set for everyday use.

We can get around this by placing an `export` statement for each secret variable into a file (`secrets.txt` located in the production directory) that is only readable by root. This is file will be gitignored.

```sh
sudo su - root
SECRETS_FILE=/var/www/django_cms_demo/secrets.txt

echo 'export DJANGO_CMS_DEMO_SECRET_KEY="<super_secret_key>"
export DJANGO_CMS_DEMO_DB_PASSWORD="<super_secret_password>"' > $SECRETS_FILE

chmod 600 $SECRETS_FILE
exit
```

Now, whenever someone needs to set these secret environmental variables, they just need to run this:

```sh
eval `sudo cat secrets.txt`
```

## Collect static files

Change `STATIC_URL` in `$PROJECT_DIR/$PROJECT_NAME/settings.py` to:

```sh
STATIC_URL = '/static/django_cms_demo/'    # allows static files for multiple projects
```

Gitkeep empty static directory

```sh
touch django_cms_demo/static/.gitkeep
```

After gitignoring `/static/` and pulling the changes to the production repo, collect the static files. This can be done whenever there are changes/additions to the static files.

```sh
sudo su - root
PROJECT_NAME=django_cms_demo
PROJECT_DIR_PRODUCTION=/var/www/$PROJECT_NAME

cd $PROJECT_DIR_PRODUCTION
source env/bin/activate
source secrets.txt
./manage.py collectstatic
exit
```

## Migrate database and create super user

To perform the initial database migration, we do the following:

```sh
cd /var/www/django_cms_demo/
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

Now we can login to the [demo site](http://symposium.plb.ucdavis.edu/django_cms_demo/admin/).

## Add allowed host and turn off debug mode

Change the `DEBUG` and `ALLOWED_HOSTS` variables in `~/git.repos/django_cms_demo/django_cms_demo/settings.py` from this:

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

