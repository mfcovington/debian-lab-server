# An installation guide for Drupal and Debian stable (Wheezy)

## get debian

download [netinst image](http://cdimage.debian.org/debian-cd/7.6.0/amd64/iso-cd/debian-7.6.0-amd64-netinst.iso)

during installation select:

* debian desktop environment
* web server
* print server
* file server
* SQL database
* SSH server
* Laptop
* Standard system utilities

## install prereqs
    su
    aptitude install php5
    aptitude install mysqlserver 
    aptitude install php-mysql
    aptitude install phpmyadmin
    
You can skip phpmyadmin if you are comfortable with command line administration of mySQL
    
## Add phpmyadmin to apache

If you are going to use phpmyadmin then you need to tell apache about it:

    vi /etc/apache2/apache2.conf

Then add `Include /etc/phpmyadmin/apache.conf` to the end of the file

Restart the server

    sudo service apache2 restart
    
Now you can adminster mySQL by going to http://localhost/phpmyadmin
    
## Download drupal and start installation

	 cd /var/www
	 wget http://ftp.drupal.org/files/projects/drupal-7.31.tar.gz
	 tar -xvzf drupal-7.31.tar.gz 
	 mv drupal-7.31/* drupal-7.31/.htaccess ./
	 mv drupal-7.31/.gitignore ./

Note: it might be better to give drupal its own directory inside of /var/www but for now I am just following the standard directions and placing it at top level

### set up mysql

follow [these instructions](https://www.drupal.org/documentation/install/create-database#phpmyadmin) for setting up mySQL using phpmyadmin

Having phpmyadmin automatically create the database when I created the user did not work.  I had to create a new database "drupal" and the edit the drupal user to have permissions.  All of this was done in phpmyadmin.

### set up configuration files

Follow [these instructions](https://www.drupal.org/documentation/install/settings-file)

### start the installation script!

First move the default index page away so that apache will start with the drupal page

    mv index.html index.bak.html
    
Then point your browser to localhost.  The drupal install and config script should start.

### set permissions back

	cd /var/www/sites/default/
	chmod 0644 settings.php 
	chown -R www-data files
	chmod -R 0755 files/
	
I hope these are right!
     
