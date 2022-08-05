Quick Installation
==================

Introduction
------------
This install guide is a quick overview of how to install all the components of
HolistIX and how to get started from scratch. For more details on how to
install of each of the components, check the technical walkthrough section.

We recommend a fresh install of Ubuntu LTS 20.04 on a machine with a minimum of
10GB of space (40GB recommended) and 2GB of RAM (4GB recommended).

During the Ubuntu installation, install OpenSSH so you can ssh in to complete
the process later, and do not install Docker. We will install Docker using
their recommended process later.

IXP Manager
-----------
We will use the IXP Manager automated install script. As with all scripts
downloaded from the internet, we strongly recommend examining them. IXP
Manager offers a breakdown of the install process
`here <https://www.youtube.com/watch?v=qRIl1ioG6Ck>`_ as well as instructions
for the `manual installation <https://docs.ixpmanager.org/install/manually/>`_.

To install using the
`script <https://github.com/inex/IXP-Manager/tree/master/tools/installers>`_,
run the commands below and follow through the install process

.. code-block:: bash

    # change to root user
    sudo su -

    # download the installation script
    wget https://github.com/inex/IXP-Manager/raw/master/tools/installers/ubuntu-lts-2004-ixp-manager-v5.sh

    # and execute it:
    bash ./ubuntu-lts-2004-ixp-manager-v5.sh

Once you have finished the install process, follow the
`post-install steps <https://docs.ixpmanager.org/install/next-steps/>`_.

For HolistIX, the following needs to be configured:

- Infrastructure (think of it as an IXP)
- Facilities (point of presences)
- Racks
- Switches (SNMPv2 is recommended)
- VLAN(s)
- IPv4/6 addresses

.. _miru-install:

Miru
----

The install instructions are based on the IXP Manager automated script and uses
the following environment variables. Change appropriately for your install.

.. code-block:: bash

    IXPROOT=/srv/ixpmanager
    MY_WWW_USER=www-data

- In ``${IXPROOT}`` run ``composer require holist-ix/miru``\
- Edit ``$IXPROOT/.env`` uncomment and change ``VIEW_SKIN=miru``

The package is now installed, however we still need to do a couple more steps to
access HolistIX within IXP Manager.

.. code-block:: bash

    # Add Miru web pages to IXP Manager
    ln -s ${IXPROOT}/vendor/holist-ix/miru/src/skins/miru ${IXPROOT}/resources/skins/miru
    # Copy over custom variable page
    cp ${IXPROOT}/vendor/holist-ix/miru/src/config/custom.php ${IXPROOT}/config/custom.php

    # Add the JavaScript libraries and keep them posted with
    ln -s ${IXPROOT}/vendor/holist-ix/miru/src/js/mxgraph ${IXPROOT}/public/mxgraph

    # Ensure that our user still has permission to work with everything
    chown -R $MY_WWW_USER: ${IXPROOT}/resources/skins/miru
    chmod -R ug+rw ${IXPROOT}/resources/skins/miru

In ``${IXPROOT}/config/custom.php`` you will find the custom variables for
HolistIX. Here we specify the ``Athos`` install directory as well as the
``Cerberus`` api url.


Below is a sample config file:

.. _sample-config:

.. code-block:: php

    return [
        'athos' => [
            // The directory where your athos is stored
            'dir' => '/athos',
            // The URL for the athos wrapper API
            'api_url' => 'http://localhost:8989'
        ],
        'cerberus' => [
            'api_url' => 'http://localhost:8080/api',
        ]
    ];


The ``athos wrapper`` api is a simple REST api to help ``miru`` start the
athos instance. A simple wrapper can be found `here <https://github.com/Belthazaar/athosapi>`_.


Athos
-----

We will be primarily running Athos through docker. However, we will have a local
copy to help with running scripts and storing the configurations generated from
Miru.

Get a local copy through either ``git clone`` or ``wget``:

.. code-block:: bash

    wget -q -O athos.zip https://github.com/Holist-IX/athos/archive/refs/heads/master.zip &&
    unzip athos.zip &&
    rm athos.zip

This downloads and unzips Athos. Ensure that the Athos install directory matches
what you have set in IXP Manager's :ref:`custom config <sample-config>` at
``${IXPROOT}/config/custom.php``.

Docker
~~~~~~

Docker is used primarily to help with emulating p4 enabled switches and to
reduce the impact of running emulated networks on the host machine. Therefore it
is **strongly** recommended to install the Athos docker image. Miru is by
default configured to expect this configuration.

.. important::
    For best results install Docker Community Edition (CE) following the official
    docker `installation guide <https://docs.docker.com/engine/install/>`_.

Once completed, the Athos docker image can be pulled with
``docker pull belthazaar/athos``

To verify everything is working correctly, you can run the following

.. code-block:: bash

    docker run --privileged belthazaar/athos

This will run Athos with the example network topology, running 4 OpenFlow
enabled Open vSwitches configured via faucet and 2 bmv2 switches running p4
compiled umbrella code.

To automate starting and stopping our docker containers, we make
use of docker-compose and the ``runDocker.sh`` script in the root directory
of athos.

Since the docker image needs to be run in privileged mode, we have a wrapper API
for athos that will run the docker image in privileged mode. The wrapper API will
also handle storing the config for athos, getting the results and cleaning up
the container afterwards.


Cerberus installation
---------------------

For deploying network configurations, we make us of the Cerberus SDN controller.
This is the same controller used within the ``Athos`` Docker image.

Cerberus is an OpenFlow controller for OpenFlow 1.3 switches that focuses on
layer-2 switching. Cerberus takes a proactive approach to network configuration
and only allows connections to hosts that are already configured.

We can install Cerberus via pip as follows:

.. code-block:: bash

    % pip install cerberus-controller

You can also install Cerberus from the source code if you prefer via:

.. code-block:: bash

    % git clone https://github/Holist-IX/cerberus/
    % cd cerberus; pip install .

The default configuration is located at ``/etc/cerberus/topology.json``, with
older configs stored at ``/etc/cerberus/rollback/`` and failed configs will be
stored at ``/etc/cerberus/failed/``.

The default api address for Cerberus is ``http://localhost:8080/api``.

.. todo:: Add service install instructions.


OpenFlow Switch configuration
-----------------------------

The best resource location to configure your OpenFlow switch, will be to follow
the guide that ``faucet`` have on their
`docs site <https://docs.faucet.nz/en/latest/vendors/index.html>`_, as it is
maintained primarily by the vendors themselves.

.. note::

    Currently there is no way to declare DPIDs in IXP Manager. To overcome this
    we use the switch id assigned in IXP Manager. To find the switch id, check
    the :ref:`switch id section <dpid>`.



Post Install Instructions
-------------------------

As everything will be run from IXP Manager and HolistIX, ensure that your
``$MY_WWW_USER`` has access to read and write within ``$ATHOSROOT``.

Ensure you can log into IXP Manager and that you can access ``Miru``, this is
available to superuser (aka admins) in IXP Manager and can be located at the
bottom of of the left side bar.
