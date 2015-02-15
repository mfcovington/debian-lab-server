# DjangoCMS Demo Project

```sh
sudo su - root

PROJECT_NAME=demo_cms
PROJECT_DIR=/var/www/$PROJECT_NAME

mkdir $PROJECT_DIR
chown www-data:www-data $PROJECT_DIR
chmod g+s $PROJECT_DIR
cd $PROJECT_DIR
virtualenv -p /usr/local/opt/python-3.4.2/bin/python3.4 env
source env/bin/activate

pip install djangocms-installer

djangocms -s -p . $PROJECT_NAME
# Database configuration (in URL format) [default sqlite://localhost/project.db]: sqlite://localhost//var/www/demo_cms/project.db
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
# Load a starting page with examples after installation. Choose "no" if you use a custom template set# . (choices: yes, no) [default no]: 
```