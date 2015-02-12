### mod_wsgi for Apache

#### Install and enable mod_wsgi via Aptitude (Mike, 2015-01-14)

```sh
sudo aptitude install libapache2-mod-wsgi
sudo a2enmod wsgi
```

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
