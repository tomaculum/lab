.. author:: ezra <ezra@posteo.de>

.. tag:: lang-nodejs
.. tag:: web
.. tag:: file-storage

.. highlight:: console

.. sidebar:: Logo

  .. image:: _static/images/up1.png
      :align: center

###
Up1
###

.. tag_list::

.. abstract::
  Up1_ provides a very simple web interface to share files end-to-end encrypted via an autogenerated url. It works without any registration or login, the encryption takes place in the clients browser, so the server will know nothing about the files content.

----

.. note:: For this guide you should be familiar with the basic concepts of

  * :manual:`Node.js <lang-nodejs>` and its package manager :manual_anchor:`npm <lang-nodejs.html#npm>`
  * :manual:`supervisord <daemons-supervisord>`
  * :manual:`domains <web-domains>`

Prerequisites
=============

Node and npm
------------

We're using :manual:`Node.js <lang-nodejs>` in the stable version 8:

::

 [isabell@stardust ~]$ uberspace tools version show node
 Using 'Node.js' version: '8'
 [isabell@stardust ~]$

We will need to update the Node Packet Manager npm:

::

  [isabell@stardust ~]$ npm install npm@latest -g
  + npm@6.4.1
  updated 1 package in 14.629s
  [isabell@stardust ~]$ hash -r
  [isabell@stardust ~]$ npm --version
  6.4.1
  [isabell@stardust ~]$

This should return a version from ``6.0.0`` upwards.

Domain
------

Your URL needs to be setup:

.. include:: includes/web-domain-list.rst

Installation
============

First get the Up1 source code from Github_:

::

  [isabell@stardust ~]$ git clone https://github.com/Upload/Up1 up1
  Cloning into 'up1'...
  remote: Enumerating objects: 37, done.
  remote: Counting objects: 100% (37/37), done.
  remote: Compressing objects: 100% (33/33), done.
  remote: Total 37 (delta 0), reused 16 (delta 0), pack-reused 0
  Unpacking objects: 100% (37/37), done.
  Note: checking out '059f199a6f94fc6c90cde1e4dab0b5b85b49533c'.
  [isabell@stardust ~]$

You have now downloaded the master development branch, because this might behave like a beta version of the software, you should check out the latest release.

Please be sure to replace the pseudo version number ``66.6`` here with the latest version number from the release feed_:

::

  [isabell@stardust ~]$ cd ~/up1
  [isabell@stardust up1]$ git checkout tags/v66.6
  You are in 'detached HEAD' state. You can look around, [...]
  [isabell@stardust up1]$ git checkout -b latest
  Switched to a new branch 'latest'
  [isabell@stardust up1]$

Then run :manual_anchor:`npm <lang-nodejs.html#npm>` to install all the dependencies:

::

  [isabell@stardust ~]$ cd ~/up1/server
  [isabell@stardust server]$ npm install
  [...]
  added 131 packages from 90 contributors and audited 219 packages in 6.04s
  found 17 vulnerabilities (4 low, 8 moderate, 5 high)
  run `npm audit fix` to fix them, or `npm audit` for details
  [isabell@stardust server]$

You probably got the error that there are vulnerabilities to fix, so follow the instruction and run ``npm audit fix`` to install the latest package versions:

::

  [isabell@stardust ~]$ cd ~/up1/server
  [isabell@stardust server]$ npm audit fix
  [...]
  fixed 17 of 17 vulnerabilities in 219 scanned packages
  [isabell@stardust server]$

Configuration
=============

Change the configuration
------------------------

Copy the example settings files to create your own configuration:

::

  [isabell@stardust ~]$ cd ~/up1
  [isabell@stardust up1]$ cp server/server.conf.example server/server.conf
  [isabell@stardust up1]$ cp client/config.js.example client/config.js
  [isabell@stardust up1]$

Now edit ``~/up1/server/server.conf`` and set a random string for ``api_key`` and ``delete_key``:

.. note:: You can use the following code, for example, to create a random string: ``pwgen 32 1``

.. code-block:: none
 :emphasize-lines: 2,3,13

  {
  "api_key": "<any_random_string>",
  "delete_key": "<any_other_random_string>",
  "maximum_file_size": 50000000,

  "path": {
    "i": "../i",
    "client": "../client"
  },

  "http": {
    "enabled": true,
    "listen": ":9002"
  },

Then add the same ``api_key`` to the file ``~/up1/client/config.js``:

.. code-block:: none

  upload.config.api_key = '<random_string_from_above>'

If you want to, you can also edit the website footer and contact information to your details in the same file:

.. code-block:: none

  upload.config.footer = '<a href="https://github.com/Upload/Up1" target="_blank">Source Code</a> - <a href="mailto:isabell@stardust.uber.space" id="contact" target="_blank">Contact</a>'

Configure web server
--------------------

.. note::

    Up1 is running on port 9002.

.. include:: includes/web-backend.rst

Setup daemon
------------

Create ``~/etc/services.d/up1.ini`` with the following content:

.. code-block:: ini

 [program:up1]
 directory=%(ENV_HOME)s/up1/server/
 command=node %(ENV_HOME)s/up1/server/server.js
 startsecs=60

.. include:: includes/supervisord.rst

If it's not in state RUNNING, check your configuration.

Check Website
-------------

Point your browser to your domain for example https://isabell.uber.space

.. _Up1: https://github.com/Upload/Up1
.. _Github: https://github.com/Upload/Up1
.. _feed: https://github.com/Upload/Up1/releases

----

Tested with Up1 1.0 and Uberspace 7.1.16.0

.. author_list::
