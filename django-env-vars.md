### How to pass environmental variables to a Django project

Below is an example of passing an environmental variable to a Django project (based on a [StackOverflow post](http://stackoverflow.com/questions/26979579/django-mod-wsgi-set-os-environment-variable-from-apaches-setenv)):

To set the environmental variable `MALOOF_LAB_SITE_DB_PASSWORD`, add the indicated line to `/etc/apache2/sites-enabled/000-default`:

```apache
<VirtualHost *:80>
...
        WSGIDaemonProcess sampleapp python-path=/var/www/sampleapp:/var/www/sampleapp/env/lib/python3.4/site-packages
        WSGIProcessGroup sampleapp
        SetEnv MALOOF_LAB_SITE_DB_PASSWORD super_secret_password    # add this line
        WSGIScriptAlias /test_py3_fresh /var/www/sampleapp/sampleapp/wsgi.py
...
</VirtualHost>
```

To import the variable into the Django project, change `/var/www/sampleapp/sampleapp/wsgi.py` from this:

```python
"""
WSGI config for sampleapp project.

It exposes the WSGI callable as a module-level variable named ``application``.

For more information on this file, see
https://docs.djangoproject.com/en/1.7/howto/deployment/wsgi/
"""

import os
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "sampleapp.settings")

from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()
```

to this:

```python
"""
WSGI config for sampleapp project.

It exposes the WSGI callable as a module-level variable named ``application``.

For more information on this file, see
https://docs.djangoproject.com/en/1.7/howto/deployment/wsgi/
"""

from django.core.handlers.wsgi import WSGIHandler
import django
import os

class WSGIEnvironment(WSGIHandler):

    def __call__(self, environ, start_response):

        os.environ['MALOOF_LAB_SITE_DB_PASSWORD'] = environ['MALOOF_LAB_SITE_DB_PASSWORD']
        os.environ.setdefault("DJANGO_SETTINGS_MODULE", "sampleapp.settings")
        django.setup()
        return super(WSGIEnvironment, self).__call__(environ, start_response)

application = WSGIEnvironment()
```
