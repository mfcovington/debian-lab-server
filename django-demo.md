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

pip install psycopg2
```
