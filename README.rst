=================
 OneGov buildout
=================

This buildout can be used for installing OneGov Box.

.. contents::

Usage
=====

The buildout includes various configuration files such as for development and
for production but it is also possible to just use parts of it, such as the
version pinnings (known good sets).

No ``buildout.cfg`` file is supplied und should never be added to the
repository. Instead either create a symbolic link to an appropriate
configuration file (e.g. ``development.cfg`` or ``production.cfg``) or extend
a configuration if you intend to make some customizations for your personal
usage only.


Fast and easy
-------------

Open a terminal and execute:

.. code:: shell

    $ git clone https://github.com/OneGov/onegov-buildout.git
    $ cd onegov-buildout
    $ python2.7 bootstrap.py
    $ bin/buildout

This has installs OneGov. You can start it by executing:

.. code:: shell

    $ ./bin/supervisord

Now fire up ``http://localhost:8081/`` in a browser.
The username is ``admin`` and the password is ``admin`` too by default.

Shut down:

.. code:: shell

    $ ./bin/supervisorctl shutdown


Production installation in detail
---------------------------------

Installing the buildout for a productive environment:

.. code:: shell

    $ git clone https://github.com/OneGov/onegov-buildout.git
    $ cd onegov-buildout

Create a ``buildout.cfg`` with your configuration.
Here is an example configuration:

.. code:: ini

    [buildout]
    extends =
        production.cfg
        instances/2.cfg
        versions/3.1.cfg

    port-base = 100

    [filestorage]
    parts =
        foo.mydomain.com
        bar.mydomain.com

Then run the buildout:

.. code:: shell

    $ python2.7 bootstrap.py
    $ bin/buildout

This will install you a complete environment with theese tools and features:

- ``./bin/zeo`` - The ZEO server provides and manages the database (port ``10020``).
- ``./bin/instance0`` - This ZEO client instance (HTTP-Server) can be used for
  debugging and is usually not running (port ``10080``).
- ``./bin/instance1``, ``./bin/instance2`` - By extending the configuration
  ``instances/2.cfg`` it creates us two ZEO client instances (HTTP-Servers)
  for serving the HTTP requests (ports ``10081`` and ``10082``).
- ``./bin/superviserd``, ``./bin/supervisorctl`` - The production buildout
  includes a supervisor which is automatically configured to start and monitor
  the ZEO server and the ZEO clients (excluding ``instance0``).
  The supervisor daemon runs on port ``10099``.
- ``filestorages`` - Creating Plone site directly on the Zope app root (which is
  in the ``Data.fs`` database) is not recommended.
  Instead you should create a filestorage part (ZODB Mount Point) for each Plone
  site. This allows to easily move sites to other instances / servers later.
  This example buildout creates two mount-points: ``foo.mydomain.com`` and
  ``bar.mydomain.com``.
- ``ftw.recipe.deployment`` - **TODO** describe logrotate etc..

You can now **start** the all necessary parts (zodb, clients) with:

.. code:: shell

    $ bin/supervisord

For showing **status** and **log files**, use:

.. code:: shell

    $ bin/supervisorctl
    HttpOk1                          RUNNING    pid 1857, uptime 1 day, 1:09:36
    HttpOk2                          RUNNING    pid 1858, uptime 1 day, 1:09:36
    Memmon                           RUNNING    pid 1859, uptime 1 day, 1:09:36
    instance0                        STOPPED    Not started
    instance1                        RUNNING    pid 1862, uptime 1 day, 1:09:36
    instance2                        RUNNING    pid 1861, uptime 1 day, 1:09:36
    zeo                              RUNNING    pid 1860, uptime 1 day, 1:09:36
    supervisor> fg instance1
    ...

For **stopping** it, use:

.. code:: shell

    $ bin/supervisorctl shutdown


Ports
-----

By changing the ``buildout:port-base`` configuration in your buildout you can
influence all ports at once (rerun of ``./bin/buildout`` required when changing
it!).

The ``buildout:port-base`` config is the prefix of all ports.
For example if you set ``porta-base = 55`` it will configure theese ports:

- `bin/zeo` - 5520
- `bin/instance0` - 5580
- `bin/instance1` - 5581
- `bin/instance2` - 5582
- `bin/instance3` - 5583 etc...
- `bin/supervisord` - 5599



Development
-----------

The buildout includes a ``development.cfg`` which is configured to checkout the
onegov packages to the ``src`` directory as git repositories (using ``mr.developer``).

For installing the latest development version you can run the buildout with
the ``development.cfg`` like this:

.. code:: shell

    $ git clone https://github.com/OneGov/onegov-buildout.git
    $ cd onegov-buildout
    $ ln -s development.cfg buildout.cfg
    $ python2.7 bootstrap.py
    $ bin/buildout
    $ bin/instance0 fg

Then you can navigate your browser to ``http://localhost:8080/`` and install
a OneGov instance.


Contributing code
-----------------

When you would like to contribute code, please create forks of the modified python packages
on github.

Find the package on github and click on the fork button
(you need to have a github account be logged in).
For getting your forked repository in your installation, you to tell the buildout to check
out your fork.

Example: If you have forked `ftw.book <https://github.com/4teamwork/ftw.book>`_ you could
create a ``buildout.cfg`` file with this content:

.. code:: ini

    [buildout]
    extends = development.cfg


    [sources]
    ftw.book git git://github.com/your-github-username/ftw.book.git


Filestorage
-----------

Each Plone site should be created in a separate filestorage, which is setup
in the ``filestorage`` part in ``base.cfg``. This allows easy moving of
filestorages between Zope instances.
