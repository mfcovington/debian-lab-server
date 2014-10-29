# Installing shiny-server

Generally followed [these instructions](https://github.com/rstudio/shiny-server/wiki/Building-Shiny-Server-from-Source)

## Prerequisites


The version of cmake in the wheezy repository is too old, so install our own:

    wget http://www.cmake.org/files/v2.8/cmake-2.8.12.2.tar.gz
    tar -xvzf cmake-2.8.12.2.tar.gz
    cd cmake-2.8.12.2/
    more Readme.txt
    ./bootstrap --prefix=/usr/local
    make
    sudo make install

Other prequisites:

    sudo apt-get install git
    sudo -s
    R

within R:

    install.packages('shiny', repos='http://cran.rstudio.com/')
    q()

back to bash

    exit
    cd
    
## Download and install shiny-server

    mkdir git
    cd git
    git clone https://github.com/rstudio/shiny-server.git
    cd shiny-server
    mkdir tmp
    cd tmp
    DIR=`pwd`
    PATH=$PATH:$DIR/../bin/
    PYTHON=`which python`
    # Check the version of Python. If it's not 2.6.x or 2.7.x, see the Python section below.
    $PYTHON --version
    /usr/local/bin/cmake -DCMAKE_INSTALL_PREFIX=/usr/local -DPYTHON="$PYTHON" ../
    make
    mkdir ../build
    (cd .. && bin/npm --python="$PYTHON" rebuild)
    (cd .. && ext/node/lib/node_modules/npm/node_modules/node-gyp/bin/node-gyp.js --python="$PYTHON" rebuild)
    sudo make install

## Do some configuration

    sudo ln -s /usr/local/shiny-server/bin/shiny-server /usr/bin/shiny-server
    sudo useradd -r -m shiny
    sudo mkdir -p /var/log/shiny-server
    sudo mkdir -p /srv/shiny-server
    sudo mkdir -p /var/lib/shiny-server
    sudo chown shiny /var/log/shiny-server
    sudo mkdir -p /etc/shiny-server
    cd ../
    sudo cp config/default.config /etc/shiny-server/shiny-server.conf #this can be modified to change file locations,etc

## Set up the start init daemon

    sudo cp config/init.d/debian/shiny-server /etc/init.d/
    sudo chmod 755 /etc/init.d/shiny-server


Need to change the init.d script.  Differences are shown below:

    jmaloof@symposium:/etc/init.d$ diff ~/git/shiny-server/config/init.d/debian/shiny-server /etc/init.d/shiny-server 21c21
    < DAEMON=shiny-server
    ---
    > DAEMON=/usr/bin/shiny-server
    28a29,32
    >
    > # override default VERBOSE setting
    > VERBOSE=yes
    >

Now tell the system to load this on startup

    sudo insserv shiny-server

And get the server started 

     sudo service shiny-server start
     
## note

Currently serving from /srv/shiny-server.  Probably want to change that in the future.
See `/etc/shiny-server/shiny-server.conf`



