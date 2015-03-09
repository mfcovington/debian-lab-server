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

#### Cheatsheet for examining PostgreSQL databases

- List psql meta-commands

        \?

- Change to postgres user and open the psql prompt

        sudo -u postgres psql postgres

- List databases

        \l

- Connect to a database

        \c <db_name>

- List tables in connected database

        \dt

- List columns on table

        \d <table_name>

- Show data in columns

        SELECT <column_name> FROM <table_name>;                     -- Single column
        SELECT <column_name_1>, <column_name_2> FROM <table_name>;  -- Multiple columns
        SELECT * FROM <table_name>;                                 -- All columns

There are lots of online PostgreSQL cheatsheet. Here are a few not-so-carefully-selected ones:

- http://www.postgresonline.com/downloads/special_feature/postgresql83_psql_cheatsheet.pdf
- http://blog.jasonmeridth.com/posts/postgresql-command-line-cheat-sheet/
- https://gist.github.com/Kartones/dd3ff5ec5ea238d4c546
