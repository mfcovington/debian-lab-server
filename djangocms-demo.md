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
