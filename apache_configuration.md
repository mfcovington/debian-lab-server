# Apache configuration

## Overall configuration

The overall configuration file is `/etc/apache2/

add the following lines to prevent display of server software and version number
    
    ServerSignature Off
    ServerTokens Prod

## First site

The configuration file is `/etc/apache2/sites-enabled/000-default`

change root file directory at the beginning of the file

    DocumentRoot /mnt/data/www

turn off indexes and Symlinks by editing `000-default`

    <Directory />
            Options -FollowSymLinks
            Options -Indexes
            Options MultiViews
            Order allow,deny
            allow from all
    </Directory>

allow CORS for dalliance by editting `000-default`

    <Directory /dalliance/>
        Header set Access-Control-Allow-Origin "*"
        Header set Access-Control-Allow-Headers "Range"
        Header set Access-Control-Max-Age "36000"
    </Directory>
    


    
then enable headers and restart apache

    sudo a2enmod headers
    sudo /usr/sbin/service apache2 restart


    