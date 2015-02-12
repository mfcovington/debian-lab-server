### mod_wsgi for Apache

#### Install and enable mod_wsgi via Aptitude (Mike, 2015-01-14)

First tried the Python 3 version:

```sh
sudo aptitude install libapache2-mod-wsgi-py3
sudo a2enmod wsgi-py3
```

With the Python 3 version of mod_wsgi, the Apache server wouldn't start when the config file tries to work with WSGI. I think this is because the Django project uses Python 3.4.2, but `aptitude` installs mod_wsgi for Python 3.2.3. Therefore, I'll try the Python 2 version.

```sh
# disable and remove py3 version
sudo a2dismod wsgi-py3
sudo aptitude remove libapache2-mod-wsgi-py3

# install and enable py3 version
sudo aptitude install libapache2-mod-wsgi
sudo a2enmod wsgi
```

This worked for Django projects using the native Python 2 version. So, at least I know I can get Django working. However, it still doesn't work for my Django lab site project that uses Python 3.4.2. Therefore, I will need to install from source.

### PostgreSQL

Install and configure PostgreSQL based on [DigitalOcean tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-django-with-postgres-nginx-and-gunicorn)

#### Install PostgreSQL (Mike, 2015-01-14)

```sh
sudo aptitude install libpq-dev
sudo aptitude install postgresql postgresql-contrib
```

#### Configure PostgreSQL (Mike, 2015-01-14)

```sh
sudo su - postgres
createdb maloof_lab_site_db
createuser -P
# Enter name of role to add: maloof_lab_site_admin
# Enter password for new role: 
# Enter it again: 
# Shall the new role be a superuser? (y/n) n
# Shall the new role be allowed to create databases? (y/n) n
# Shall the new role be allowed to create more new roles? (y/n) n
psql
```

```sql
GRANT ALL PRIVILEGES ON DATABASE maloof_lab_site_db TO maloof_lab_site_admin;
```
