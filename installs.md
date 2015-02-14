# Installs

software (status)

If you start working on one of these, then change the status.

Note installation procedure here (if a one liner) or link to a separate .md file

Unless there is a good reason not to, __please use the package manager (apt-get, aptitude, etc)__

* Python 3, etc. [Details](python3_install.md)
    * v3.4.2 with shared library support (Mike 2015-02-10)
    * v3.4.2 (Mike 01/12/15)
    * v3.2.3 (Mike 11/03/14)

* Python 2 headers for libs req'd by django-cms (Mike 11/05/14)

         sudo aptitude install python-dev

* scipy numpy (open)
* django [Details](django.md)
* R (Julin 10/27/14.  Ver 3.1.1).  [Details](r_install.md)
* Shiny server (Julin 10/28/14) [Details](shiny-server_install.md)
* UCSC genome browser (Julin)
    * Actually will try [biodalliance](http://www.biodalliance.org/)
* apache (Julin installed 10/27/14)

         sudo aptitude install apache2

* php (open)
* mysql (open)
* mosh (Julin 10/27/14)

        sudo aptitude install mosh
   
* vim 7.3.547 (Mike 11/03/14)

        sudo aptitude install vim

* htop 1.0.1-1 (Mike 11/04/14)

        sudo aptitude install htop

* screen (Julin 11/24/14)

        sudo aptitude install screen
        
* [node.js 0.10.33](http://nodejs.org/) (Julin 11/24/14)
Followed [these instructions](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager)

* bootstrap (Julin 11/24/14)

        cd /mnt/srv/www
        wget https://github.com/twbs/bootstrap/releases/download/v3.3.1/bootstrap-3.3.1-dist.zip
        unzip bootstrap-3.3.1-dist.zip

* tree (displays directory tree, in color; Mike 2015-02-13)

        sudo aptitude install tree
