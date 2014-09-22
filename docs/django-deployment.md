SLP: Django: Deployment
=======================

[Go up to the main SLP documents page](index.html) ([md](index.md))

This page is for how to *deploy* your Django apps on an Ubuntu server -- specifically, Ubuntu 14.04.  This is not needed for your local development, as you can run `python manage.py runserver` to test it out locally, or test it out on the course server provided.  However, if you ***do*** want to get it running through Apache, read on...

Installing packages
-------------------

To install Django on your own system, under Ubuntu 14.04, just enter `sudo apt-get install python-django python-mysqldb`, and it will do the rest.  As of the writing of this tutorial (Aug 31, 2014), this installs Python version 2.7.6 and Django version 1.6.1.

Configuring Apache
------------------

For *each* django app, the following 8 lines need to be added to `/etc/apache2/sites-available/000-default.conf`, right above the `</VirtualHost>` line:

```
WSGIScriptAlias /djangohw /home/slp/user/djangohw/djangohw/wsgi.py
WSGIProcessGroup user
WSGIDaemonProcess user python-path=/home/slp/user/djangohw
<Directory /home/slp/user/djangohw/djangohw/>
    <Files wsgi.py>
        Require all granted
    </Files>
</Directory>
```

A bunch of notes:

- The [WSGIScriptAlias](https://code.google.com/p/modwsgi/wiki/ConfigurationDirectives#WSGIScriptAlias) line indicates the URL (after the server) links to the particular Django app
    - If you want them in sub-directories, you can have the URL part be `/django/user`, and you don't have to create the `/django/` directory in the HTML document root (likely `/var/www/html`)
- The [WSGIProcessGroup](https://code.google.com/p/modwsgi/wiki/ConfigurationDirectives#WSGIProcessGroup) creates a process group, which allows one (or more) apps to run as a specific user
    - The `user` part of that line needs to be a valid user name on the system
- The [WSGIDaemonProcess](https://code.google.com/p/modwsgi/wiki/ConfigurationDirectives#WSGIDaemonProcess) line indicates that this particular Django app will run under the `user` user, and the path to the django app is provided
    - The `user` part of that line needs to match the one in the WSGIProcessGroup
- The `Require all granted` line allows the viewing of the Django app
    - If you are using Apache 2.2 (or earlier), replace `Require all granted` with two lines: `Order deny,allow` and then `Allow from all`

If there are going to be many users in a system, then it will make it easier to put this in a separate file (say, called `/etc/apache2/django.conf`), and insert a `Include django.conf` line in `/etc/apache2/sites-available/000-default.conf` (also right above the `</VirtualHost>` line).

If you are enabling these through SSL, there is a slight change.  When Apache restarts (or reloads), the default (i.e., non-SSL) site is brought up first, followed by the SSL site.  As the daemon process has already been declared in 000-default.conf (for the non-SSL site), it will cause an error to declare it again for the SSL site.  Thus, the SSL version should not have the WSGIDaemonProcess line; all the other lines remain the same.  This implies that the file included from default-ssl.conf will be a different one than the one included from 000-default.ssl.

Multiple users
--------------

The issue with the WSGI module is that if **any one** of the WSGI files it is configured with does not exist, then all of the WSGI modules don't work.  This means you can't pre-generate the list of WSGI links for a class, as none of them will work until everybody has it set up, which is not viable.

To work around this, you can see the [wsgi-admin.cpp](../utils/wsgi-admin/wsgi-admin.cpp.html) ([src](../utils/wsgi-admin/wsgi-admin.cpp)), which will allow students to register and remove their WSGI apps.  It will update the necessary Apache configuration files, and then reload the web server.  It can also be used just to do the regeneration and reloading -- and if a file is not found (or is not valid), then it is excluded from the configuration files.  Installation is tricky, and more details will follow shortly...


Django static/ directory
------------------------

In addition to the per-user configuration, there are some configurations for everybody.  In particular, the http://server/static directory needs to point to the base Django static directory.  The following lines in the Apache2 configuration file for the site will achieve this:

```
Alias /static /usr/lib/python2.7/dist-packages/django/contrib/admin/static
<Directory /usr/lib/python2.7/dist-packages/django/contrib/admin/static>
  Require all granted
</Directory>
```

Verify that the `/usr/lib/python2.7/dist-packages/django/contrib/admin/static` directory is correct on your system.