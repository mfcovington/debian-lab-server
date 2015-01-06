## webadmin group
created webadmin group for web directories

    sudo adduser tiaho webadmin
    sudo adduser jmaloof webadmin
    sudo adduser mfc webadmin
    cd /mnt/data/www
    sudo chgrp -R webadmin ./shiny-server
