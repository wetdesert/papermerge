.. _server_configurations:

Server Configurations
**********************

The default is to use Django’s development server provided by ``./manage.py runserver``
command, as that’s easy and does the job well enough on a home network.
However it is heavily discouraged to use it for more than that.

If you want to do things right you should use a real webserver capable of
handling more than one thread. You will also have to let the webserver serve
the static files (CSS, JavaScript) from the directory configured in
:ref:`static_dir`. The default static files directory is ``static``.

For that you need to activate your virtual environment and collect the static
files with the command::

    $ ./manage.py collectstatic

Setting up a web server can sound daunting for folks who don't normally do
that kind of thing. This guide will help you walk through the configuration
for Apache or Nginx on Linux and OSX.

.. note::
    If this all looks too overwhelming for you, we do offer `affordable hosted
    solutions <https://papermerge.com/pricing>`_ for folks who want to use
    Papermerge but don't know how to run a web server, or don't have time to keep
    up with updates.


Apache
~~~~~~~~

The most common setup for Papermerge on a linux server is to use Apache, so if
you're not sure what to pick, Apache might be the best bet, as it's free, easy
to configure, and well documented.

In order use apache web server with Django (web framework used by Papermerge)
you need to install so called module `mod_wsgi
<https://modwsgi.readthedocs.io/en/develop/index.html>`_

Step 1 - Install Apache Web Server
####################################

On Ubuntu 20.04 LTS you install apache web server with following command::

    $ sudo apt install apache2

Step 2 - Get mod_wsgi
########################

Get latest release of `mod_wsgi from here <https://github.com/GrahamDumpleton/mod_wsgi/releases>`_. Extract archive::

    unzip mod_wsgi-4.7.1
    cd mod_wsgi-4.7.1

Step 3 - Build & Install mod_wsgi
###################################

In order to build mod_wsgi on Ubuntu Linux, you need three things:
    
        * ``build-essential`` ubuntu package with gcc compiler and friends
        * ``apache2-dev`` package
        * ``python interpreter`` from your **papermerge virtual environment**

Let's first install required packages::

$ sudo apt install build-essential apache2-dev

Next, activate your Papermerge virtual environment (python virtual environment)::

$ source /opt/papermerge/.venv/bin/activate

.. warning::

    Activating python virtual environment is very important step. Because when
    compilying mod_wsgi it must find in $PATH **python interpreter** located
    in same virtual environment with other python dependencies.
    
Switch to extracted directory mod_wsgi-4.7.1 and run following commands::

    $ ./configure
    $ make
    $ sudo make install

On Ubuntu 20.04 LTS ``sudo make install`` command will copy ``mod_wsgi.so``
binary file to ``/usr/lib/apache2/modules/mod_wsgi.so``

Next enable mod_wsgi module with following command::

    $ a2enmod mod_wsgi

You can double check if mod_wsgi module was enabled with::

    $ apachectl -M

It should display a list enabled modules. Among other should be::

    ...
    wsgi_module (shared)  
    ...


Step 4 - Configure Virtual Host
################################

In directory ``/etc/apache2/sites-available`` create a virtual configuration file for papermerge. 
Let's say papermerge.site. Here is configuration example for virtual host::


    <VirtualHost *:8060>
        <Directory /opt/papermerge/config>
            Require all granted
        </Directory>

        Alias /media/ /var/media/papermerge/
        Alias /static/ /var/static/papermerge/

        <Directory /var/media/papermerge>
           Require all granted
        </Directory>

        <Directory /var/startic/papermerge>
          Require all granted
        </Directory>

        ServerName papermerge.home
        ServerRoot /opt/papermerge
    </VirtualHost>

    WSGIPythonHome /opt/papermerge/.venv/
    WSGIPythonPath /opt/papermerge/
    WSGIScriptAlias / /opt/papermerge/config/wsgi.py


The first bit in the WSGIScriptAlias line is the base URL path you want to
serve your application at (/ indicates the root url), and the second is the
location of a WSGI file, inside papermerge project as ``config/wsgi.py``. This
tells Apache to serve any request below the given URL using the WSGI
application defined in that file.

``WSGIPythonHome`` is path to python's virtual environment.

Nginx + Gunicorn
~~~~~~~~~~~~~~~~~

Step 1 - Gunicorn
###################


Step 2 - Systemd Service for Gunicorn
#######################################


Step 3 - Systemd Service for Worker
#######################################


Step 4 - Nginx
################



Worker
~~~~~~~~
Here is worker.service unit::

    [Unit]
    Description=Papermerge Worker
    After=network.target

    [Service]
    Type=simple
    WorkingDirectory=/opt/papermerge
    ExecStart=/opt/papermerge/.venv/bin/python /opt/papermerge/manage.py worker --pidfile /tmp/worker.pid
    Restart=on-failure

    [Install]
    WantedBy=multi-user.target

.. note::

    Notice that ``ExecStart`` is **absolute path to python interpreter inside
    python virtual environment**. Absolute path to python interpreter inside
    virtual environment is enough information for python to figure out the
    rest of python dependencies from the same virtual environment. Thus, you
    don't need to provide futher information about virtual environment.

Systemd .service may be placed in one of several locations. One options is
to place it in ``/etc/systemd/system`` together with other system level
units. In this case you need root access permissions.

Another option is to place .service file inside ``$HOME/.config/systemd/user/``
In this case you can start/check status/stop systemd unit service with following commands::

    systemctl --user start worker
    systemctl --user status worker
    systemctl --user stop worker

.. note::


