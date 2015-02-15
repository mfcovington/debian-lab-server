# Django Demo Project

```sh
sudo su - root

PROJECT_NAME=demo
PROJECT_DIR=/var/www/$PROJECT_NAME

mkdir $PROJECT_DIR
chown www-data:www-data $PROJECT_DIR
chmod g+s $PROJECT_DIR
cd $PROJECT_DIR
virtualenv -p /usr/local/opt/python-3.4.2/bin/python3.4 env
source env/bin/activate

pip install django
django-admin startproject $PROJECT_NAME .
```
