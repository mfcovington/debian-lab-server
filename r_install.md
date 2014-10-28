# R Install

Followed [instructions here](http://cran.us.r-project.org/bin/linux/debian/)

Made backup copy of `/etc/apt/sources.list` to `/etc/apt/sources.list.bak`

Added `deb http://cran.cnr.Berkeley.edu/bin/linux/debian wheezy-cran3/` to `/etc/apt/sources.list`

    sudo apt-key adv --keyserver keys.gnupg.net --recv-key 381BA480
    sudo apt-get update
    sudo apt-get install r-base r-base-dev