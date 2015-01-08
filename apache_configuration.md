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
    
### fixing genome-browser link

I made a mistake and linked a tinyURL to "http://symposium.plb.ucdavis.edu/genome-%20browser/pages/" instead of "http://symposium.plb.ucdavis.edu/genome-browser/pages/"

  
Use a .htaccess redirect to fix this.  In the `000-default` file edit the Override line in the root directory section to:

    AllowOverride FileInfo=Redirect
    		
make a .htaccess file in the root directory with:

    redirect "/genome- browser/pages/" http://symposium.plb.ucdavis.edu/genome-browser/pages/
    
Apparently the better way to do this is to use the mod_rewrite but I *can not* make that work after hours of trying.


    