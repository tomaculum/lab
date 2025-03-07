.. author:: stunkymonkey <http://stunkymonkey.de>
.. author:: Matthias Kolja Miehl <https://makomi.net>
.. author:: Kevin Jost <https://github.com/systemsemaphore>
.. author:: EV21 <uberlab@ev21.de>

.. tag:: lang-nodejs
.. tag:: web
.. tag:: collaborative-editing

.. highlight:: console

.. sidebar:: Logo

  .. image:: _static/images/HedgeDoc.svg
      :align: center

########
HedgeDoc
########

.. tag_list::

.. abstract::
  HedgeDoc_ (formerly CodiMD / HackMD) is an open-source software written in :manual:`Node.js <lang-nodejs>`. HedgeDoc lets you create real-time collaborative markdown notes. It is inspired by Hackpad, :lab:`Etherpad <guide_etherpad>` and similar collaborative editors.

  HedgeDoc_ is licensed under the AGPLv3_.

----

.. note:: For this guide you should be familiar with the basic concepts of

  * :manual:`Node.js <lang-nodejs>`, its package manager :manual_anchor:`npm <lang-nodejs.html#npm>` and especially `yarn <https://yarnpkg.com/getting-started>`_
  * :manual:`MySQL <database-mysql>`
  * :manual:`Domains <web-domains>`
  * :manual:`web backends <web-backends>`
  * :manual:`supervisord <daemons-supervisord>`

Prerequisites
=============

We're using :manual:`Node.js <lang-nodejs>` version 14.

::

  [isabell@stardust ~]$ uberspace tools version use node 14
  Selected Node.js version 14
  The new configuration is adapted immediately. Minor updates will be applied automatically.
  [isabell@stardust ~]$

Setup your :manual:`Domain <web-domains>`:

.. include:: includes/web-domain-list.rst

.. include:: includes/my-print-defaults.rst

Installation
============
Download
--------

Check whether the marked line is the latest_ release.

.. code-block:: console
  :emphasize-lines: 1

  [isabell@stardust ~]$ VERSION=1.7.1
  [isabell@stardust ~]$ wget https://github.com/hedgedoc/hedgedoc/releases/download/$VERSION/hedgedoc-$VERSION.tar.gz
  [...]
  100%[======================================================>] 50,784,713  16.8MB/s   in 2.9s
  [isabell@stardust ~]$ tar --extract --gzip --file=hedgedoc-$VERSION.tar.gz
  [isabell@stardust ~]$ rm --verbose hedgedoc-$VERSION.tar.gz
  removed hedgedoc-x.x.x.tar.gz
  [isabell@stardust ~]$

Setup
-----

Then install the dependencies. This step may take a few minutes.

.. code-block:: console

  [isabell@stardust ~]$ cd hedgedoc
  [isabell@stardust hedgedoc]$ bin/setup
  Copying config files...
  Installing packages...
  yarn install v1.22.5
  [1/5] Validating package.json...
  [2/5] Resolving packages...
  [3/5] Fetching packages...
  info There appears to be trouble with your network connection. Retrying...
  info fsevents@1.2.13: The platform "linux" is incompatible with this module.
  info "fsevents@1.2.13" is an optional dependency and failed compatibility check. Excluding it from installation.
  info fsevents@2.1.3: The platform "linux" is incompatible with this module.
  info "fsevents@2.1.3" is an optional dependency and failed compatibility check. Excluding it from installation.
  [4/5] Linking dependencies...
  [5/5] Building fresh packages...
  Done in 253.95s.
  yarn install v1.22.5
  [1/5] Validating package.json...
  [2/5] Resolving packages...
  success Already up-to-date.
  Done in 1.15s.


  Edit the following config file to setup HedgeDoc server and client.
  Read more info at https://github.com/hedgedoc/hedgedoc#configuration-files

  * config.json           -- HedgeDoc config
  * .sequelizerc          -- db config
  [isabell@stardust hedgedoc]$

Database
--------

Create a Database.

.. code-block:: console

  [isabell@stardust hedgedoc]$ mysql --verbose --execute="CREATE DATABASE ${USER}_hedgedoc"
  --------------
  CREATE DATABASE isabell_hedgedoc
  --------------

  [isabell@stardust hedgedoc]$

Configuration
=============

First we remove the config file that was generated by the setup because it might be very confusing. It was a copy of ``config.json.example`` if you like to have a look at it.

.. code-block:: console


  [isabell@stardust hedgedoc]$ rm --verbose config.json
  removed ‘config.json’
  [isabell@stardust hedgedoc]$

.. note::

  You can set here some options in json format you can't yet set with environment variables. For other configuration options, check the `configuration documentation`_.

Create ``~/hedgedoc/config.json`` with the following content:

.. code-block:: json

  {}

Edit the file ``~/hedgedoc/.sequelizerc`` with the following content and adjust the highlighted line.:

.. code-block:: js
  :emphasize-lines: 7

  var path = require('path');

  module.exports = {
      'config':          path.resolve('config.json'),
      'migrations-path': path.resolve('lib', 'migrations'),
      'models-path':     path.resolve('lib', 'models'),
      'url':             'mysql://isabell:YourMySQL-Password@localhost:3306/isabell_hedgedoc'
  }

.. note:: You will also need the highlighted string later at the step for the ``~/etc/services.d/hedgedoc.ini``, so note it.

Generate session secret
-----------------------

| The cookie session secret is used to sign the session cookie. If none is set, one will randomly be generated on each startup, meaning all your users will be logged out.
| You will need the highlighted string at the next step for the ``~/etc/services.d/hedgedoc.ini``.

.. code-block:: console
  :emphasize-lines: 2

  [isabell@stardust ~]$ pwgen 32 1
  somethingSuperRandom
  [isabell@stardust ~]$


Setup daemon
------------

.. note::

  You can set a lot of environment variables. For other configuration options, check the `configuration documentation`_. The prefix for the variables may change in a future release because of the renaming of the project.

Create ``~/etc/services.d/hedgedoc.ini`` with the following content:

.. code-block:: ini
  :emphasize-lines: 7-8

  [program:hedgedoc]
  environment=
    NODE_ENV="production",
    CMD_ALLOW_EMAIL_REGISTER="false",
    CMD_ALLOW_ANONYMOUS="false",
    CMD_ALLOW_FREEURL="true",
    CMD_SESSION_SECRET="somethingSuperRandom",
    CMD_DB_URL="mysql://isabell:YourMySQL-Password@localhost:3306/isabell_hedgedoc"
  directory=%(ENV_HOME)s/hedgedoc
  command=yarn start
  startsecs=60
  autostart=yes
  autorestart=yes

.. include:: includes/supervisord.rst

If it's not in state ``RUNNING`` after one minute (at first you will see ``STARTING``), check your configuration. You can also check the run log with ``supervisorctl tail -f hedgedoc``. You can ignore the warnings about the domain, port and ssl settings.

Configure web server
--------------------

.. note::

    HedgeDoc_ is running on port 3000 in the default configuration.

.. include:: includes/web-backend.rst

Create a User
-------------

Even if you have deactivated the web based user registration you can always create users with the following command. The username has to be in the form of an email address. You will then be asked to choose a password.

.. code-block:: console
  :emphasize-lines: 1

  [isabell@stardust hedgedoc]$ NODE_ENV=production bin/manage_users --add isabell@uber.space
  Password for isabell@uber.space:*************
  Created user with email isabell@uber.space
  [isabell@stardust hedgedoc]$

You should now be able to access your HedgeDoc_ via https://isabell.uber.space

Updates
=======

.. note:: Check the `release notes`_ regularly or subscribe to the project's `GitHub release feed`_ with your favorite feed reader to stay informed about new updates and releases.

While HedgeDoc 2.0 is currently under development and 1.7 as latest release is also having bigger changes you should watch HedgeDocs `manual installation guide <https://docs.hedgedoc.org/setup/manual-setup/>`_ to notice if instructions have been changed. You should also read the release notes. Make sure you always download the already build tarball as the building process needs at least 2 GB RAM, so building is not possible on uberspace.

Manual update
-------------

Check whether the marked line is the latest_ release or the version you like to update to.

.. code-block:: console
  :emphasize-lines: 5

  [isabell@stardust ~]$ supervisorctl stop hedgedoc
  hedgedoc: stopped
  [isabell@stardust ~]$ mv --verbose hedgedoc hedgedoc_old
  ‘hedgedoc’ -> ‘hedgedoc_old’
  [isabell@stardust ~]$ VERSION=1.7.1
  [isabell@stardust ~]$ wget https://github.com/hedgedoc/hedgedoc/releases/download/$VERSION/hedgedoc-$VERSION.tar.gz
  [...]
  100%[======================================================>] 50,784,713  16.8MB/s   in 2.9s
  [isabell@stardust ~]$ tar --extract --gzip --file=hedgedoc-$VERSION.tar.gz
  [isabell@stardust ~]$ rm --verbose hedgedoc-$VERSION.tar.gz
  removed ‘hedgedoc-x.x.x.tar.gz’
  [isabell@stardust ~]$ cp --verbose hedgedoc_old/config.json hedgedoc/config.json
  ‘hedgedoc_old/config.json’ -> ‘hedgedoc/config.json’
  [isabell@stardust ~]$ cp --verbose hedgedoc_old/.sequelizerc hedgedoc/.sequelizerc
  ‘hedgedoc_old/.sequelizerc’ -> ‘hedgedoc/.sequelizerc’
  [isabell@stardust ~]$ cd hedgedoc
  [isabell@stardust hedgedoc]$ bin/setup
  Copying config files...
  Installing packages...
  yarn install v1.22.5
  [1/5] Validating package.json...
  [2/5] Resolving packages...
  [3/5] Fetching packages...
  info fsevents@1.2.13: The platform "linux" is incompatible with this module.
  info "fsevents@1.2.13" is an optional dependency and failed compatibility check. Excluding it from installation.
  info fsevents@2.1.3: The platform "linux" is incompatible with this module.
  info "fsevents@2.1.3" is an optional dependency and failed compatibility check. Excluding it from installation.
  [4/5] Linking dependencies...
  [5/5] Building fresh packages...
  Done in 597.69s.
  yarn install v1.22.5
  [1/5] Validating package.json...
  [2/5] Resolving packages...
  success Already up-to-date.
  Done in 1.22s.

  Edit the following config file to setup HedgeDoc server and client.
  Read more info at https://github.com/hedgedoc/hedgedoc#configuration-files

  * config.json           -- HedgeDoc config
  * .sequelizerc          -- db config
  [isabell@stardust hedgedoc]$ supervisorctl start hedgedoc
  hedgedoc: started
  [isabell@stardust hedgedoc]$ supervisorctl tail -f hedgedoc
  ==> Press Ctrl-C to exit <==
  [...] no errors (red colored) should appear
  [isabell@stardust hedgedoc]$ cd
  [isabell@stardust ~]$ rm --recursive hedgedoc_old
  [isabell@stardust ~]$

Under usual circumstances you don't need to change the config files. The setup script does not override your config files even if it says ``Copying config files...``.

Update script
-------------

Create ``~/bin/hedgedoc-update`` with the following content:

.. code-block:: bash

  #!/usr/bin/env bash
  ORG=hedgedoc # Organisation or GitHub user
  REPO=hedgedoc
  LOCAL=$(jq --raw-output .version ~/hedgedoc/package.json)
  LATEST=$(curl -s https://api.github.com/repos/$ORG/$REPO/releases/latest | jq --raw-output .tag_name)

  function do_upgrade() {
    supervisorctl stop hedgedoc
    mv --verbose ~/hedgedoc ~/hedgedoc_$LOCAL
    VERSION=$LATEST
    cd
    wget https://github.com/hedgedoc/hedgedoc/releases/download/$VERSION/hedgedoc-$VERSION.tar.gz
    tar --extract --gzip --file=hedgedoc-$VERSION.tar.gz
    rm --verbose hedgedoc-$VERSION.tar.gz
    cp --verbose hedgedoc_$LOCAL/config.json hedgedoc/config.json
    cp --verbose hedgedoc_$LOCAL/.sequelizerc hedgedoc/.sequelizerc
    cd hedgedoc
    bin/setup
    supervisorctl start hedgedoc
    echo "You may need to wait a minute until you can access the HedgeDoc web interface"
    echo "If everything works fine you can delete ~/hedgedoc_$LOCAL"
    #rm --recursive ~/hedgedoc_$LOCAL
  }

  if [ "$LOCAL" = "$LATEST" ]; then
    echo "Your HedgeDoc is already up to date."
  else
    if [[ "$LOCAL" < "$LATEST" ]]; then
      echo "There is a new Version available of HedgeDoc"
      echo "The latest Version is $LATEST"
      echo "Your local Version is $LOCAL"
      echo "Upgrades to next major releases are not tested."
      echo "Please read the release notes."
      echo "Also check if the upgrade instructions have changed."
      echo "Your instance might break."
      while true; do
        read -p "Do you wish to proceed with the upgrade? (Y/n) " ANSWER
        if [ "$ANSWER" = "" ]; then
          ANSWER='Y'
        fi
        case $ANSWER in
          [Yy]* | [Jj]* )
            do_upgrade
            break;;
          [Nn]* ) exit;;
          * ) echo "Please answer yes or no. ";;
        esac
      done
    fi
  fi

To make this script executable you have to run this once:

.. code-block:: console

  [isabell@stardust ~]$ chmod u+x --verbose ~/bin/hedgedoc-update
  mode of ‘/home/isabell/bin/hedgedoc-update’ changed from 0664 (rw-rw-r--) to 0764 (rwxrw-r--)
  [isabell@stardust ~]$

You can run this script with:

.. code-block:: console

  [isabell@stardust ~]$ hedgedoc-update
  Your HedgeDoc is already up to date.
  [isabell@stardust ~]$

.. _HedgeDoc: https://hedgedoc.org/
.. _`release notes`: https://hedgedoc.org/releases/
.. _latest: https://hedgedoc.org/latest-release/
.. _`Github release feed`: https://github.com/hedgedoc/hedgedoc/releases.atom
.. _`configuration documentation`: https://docs.hedgedoc.org/configuration/
.. _AGPLv3: https://github.com/hedgedoc/hedgedoc/blob/master/LICENSE

----

Tested with HedgeDoc 1.7.1, Uberspace 7.8.1.0

.. author_list::

