### mod_wsgi for Apache

#### Install and enable mod_wsgi via Aptitude (Mike, 2015-01-14)

First tried the Python 3 version:

```sh
sudo aptitude install libapache2-mod-wsgi-py3
sudo a2enmod wsgi-py3
```

With the Python 3 version of mod_wsgi, the Apache server won't start when the config file tries to work with WSGI. The error returned is:

    Invalid command 'WSGIDaemonProcess', perhaps misspelled or defined by a module not included in the server configuration
    Action 'start' failed.

I think this is because the Django project uses Python 3.4.2, but `aptitude` installs mod_wsgi for Python 3.2.3. Therefore, I'll try the Python 2 version.

```sh
# disable and remove py3 version
sudo a2dismod wsgi-py3
sudo aptitude remove libapache2-mod-wsgi-py3

# install and enable py3 version
sudo aptitude install libapache2-mod-wsgi
sudo a2enmod wsgi
```

This worked for Django projects using the native Python 2 version. So, at least I know I can get Django working. However, it still doesn't work for my Django lab site project that uses Python 3.4.2. Therefore, I will need to install from source.

#### Install mod_wsgi 4.4.8 from source (Mike, 2015-02-10)

Information for Apache/mod_wsgi 4.4.8 can be found [here](https://pypi.python.org/pypi/mod_wsgi).

Start by disabling and removing the previous mod_wsgi installation

```sh
sudo a2dismod wsgi
sudo aptitude remove libapache2-mod-wsgi
```

Apache/mod_wsgi 4.4.8 installation from source requires that shared library support be enabled for Python. I [reinstalled Python 3.4.2 using the shared library enabled](python3_install.md) on 2015-02-10.

Install Apache/mod_wsgi 4.4.8:

```sh
export PY34=/usr/local/opt/python-3.4.2/bin/python3.4

cd $HOME
wget https://pypi.python.org/packages/source/m/mod_wsgi/mod_wsgi-4.4.8.tar.gz
tar -xzf mod_wsgi-4.4.8.tar.gz
cd mod_wsgi-4.4.8
sudo $PY34 setup.py install
```

The content of `/etc/apache2/mods-available/wsgi.load` is:

    LoadModule wsgi_module /usr/lib/apache2/modules/mod_wsgi.so

Installation of mod_wsgi 4.4.8 installed this library, but not in that location. Therefore, I will make a symbolic link to the required file:

```sh
cd /usr/lib/apache2/modules/
sudo ln -s /usr/local/opt/python-3.4.2/lib/python3.4/site-packages/mod_wsgi-4.4.8-py3.4-linux-x86_64.egg/mod_wsgi/server/mod_wsgi-py34.cpython-34m.so mod_wsgi.so
```

Enable mod_wsgi:

```sh
sudo a2enmod wsgi
```

>
    Enabling module wsgi.
    To activate the new configuration, you need to run:
      service apache2 restart


Successfully restarted Apache with the expected versions of mod_wsgi and Python:

```sh
sudo /usr/sbin/apachectl -k restart
sudo tail -n2 /var/log/apache2/error.log
```

>
    [Tue Feb 10 23:48:17 2015] [notice] SIGHUP received.  Attempting to restart
    [Tue Feb 10 23:48:18 2015] [notice] Apache/2.2.22 (Debian) mod_wsgi/4.4.8 Python/3.4.2 configured -- resuming normal operations

Clean up:

```sh
sudo rm -r $HOME/mod_wsgi-4.4.8*
```
