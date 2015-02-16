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
