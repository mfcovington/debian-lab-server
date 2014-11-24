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
