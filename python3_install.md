# Python 3

## Install Python 3.2.3 (Mike: 2014-11-03)

The most recent version of Python (3.4.2) is not available via `aptitude`.

The Django docs indicate that 3.2.X is fine for [Django](https://docs.djangoproject.com/en/dev/faq/install/#what-python-version-can-i-use-with-django). The same is true for [Pandas](https://pypi.python.org/pypi/pandas), [Numpy, and SciPy](http://www.scipy.org/scipylib/faq.html#do-numpy-and-scipy-support-python-3-x). Therefore, I'll go with Python 3.2.3 for now.

```sh
sudo aptitude update
sudo aptitude install python3
```

## Install pip-3.2 (& setuptools) (Mike: 2014-11-03)

```sh
sudo aptitude install python3-pip
```

## Install virtualenvwrapper 4.3.1 (Mike: 2014-11-03)

```sh
sudo aptitude install virtualenvwrapper
```

After opening another connection to the server, `virtualenvwrapper.user_scripts` creates several scripts within `~/.virtualenvs/`

## Install Python 3.4.2 (Mike: 2015-01-12)

The py3 version of DjangoCMS requires python 3.3+. Anything above 3.2.X is not available via `aptitude` on Debian Wheezy. Used an article called [Python on Debian Wheezy](http://www.extellisys.com/articles/python-on-debian-wheezy) as a guide for installing Python 3.4.2 from source.

```sh
# prerequisites
sudo apt-get update
sudo apt-get install build-essential
sudo apt-get install libncurses5-dev libncursesw5-dev libreadline6-dev
sudo apt-get install libdb5.1-dev libgdbm-dev libsqlite3-dev libssl-dev
sudo apt-get install libbz2-dev libexpat1-dev liblzma-dev zlib1g-dev
sudo apt-get install tk8.5-dev

# pip-related
mkdir -p ~/.pip/cache
echo '[global]' > ~/.pip/pip.conf
echo 'download_cache = ~/.pip/cache' >> ~/.pip/pip.conf

# installation
cd $HOME
wget https://www.python.org/ftp/python/3.4.2/Python-3.4.2.tgz
cd /tmp
tar -zxf $HOME/Python-3.4.2.tgz
cd Python-3.4.2
./configure --prefix=/usr/local/opt/python-3.4.2
make
sudo make install
sudo rm -rf /tmp/Python-3.4.2
rm $HOME/Python-3.4.2.tgz
```

The PATH for python 3.4.2 is: `/usr/local/opt/python-3.4.2/bin/python3.4`

To start a virtual environment using this version, run the following (after changing `YOUR_VIRTUALENV_NAME`):

```sh
mkvirtualenv -p /usr/local/opt/python-3.4.2/bin/python3.4 YOUR_VIRTUALENV_NAME 
```
