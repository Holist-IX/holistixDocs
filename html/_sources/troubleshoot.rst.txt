Common Troubleshooting Issues
=============================

This section contains some common problems and how to fix them.

General Issues
--------------

Member unreachable
~~~~~~~~~~~~~~~~~~
Check if the member's details in IXP Manager matches with what the member
actually has.

The most important ones are:

- MAC Address
- IPv4/6 Address
- Port connection matches where the member's connection is

A common scenario is a member performed maintenance and their MAC changed
without notification.


Members lost traffic right after deployment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the unlikely case that members lose traffic right after a deployment, make
use of the rollback function. To start the rollback process, click
``File`` -> ``Rollback Cerberus config``. This restores the configuration on the
switches to the last working config. By default, the failed configuration will
be saved to ``/etc/cerberus/failed/``. This makes it easier to troubleshoot
and debug issues at a later time.

Issues in Miru
--------------

Miru not showing up on the sidebar
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If Miru is not on the sidebar, you most likely missed a step in the
:ref:`Miru <miru-install>` installation process. Follow the below checklist to
ensure everything is configured correctly. If things are missing, please follow
the install instructions for :ref:`Miru <miru-install>`.

- The ``belthazaar`` folder exists in ``${IXPROOT}/vendor/``
  - If not, Miru was not installed. Follow all the instructions for the :ref:`install <miru-install>`.
- That the ``miru`` folder exists in ``${IXPROOT}/resources/skins/``
- Ensure that ``$IXPROOT/.env`` has the following config set in it ``VIEW_SKIN=miru``
- Ensure that ``${IXPROOT}/public/mxgraph`` exists
- Ensure that ``${IXPROOT}/config/custom.php``has ``Athos`` set
- Ensure that ``${IXPROOT}/config/custom.php``has ``Cerberus`` set


Switches not showing up in Miru
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Ensure that there are switches configured within IXP Manager
- Ensure that the switches are configured as active

"Undefined" links when connecting 2 switches
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This occurs if there are no core ports designated between the two designated
switches. To designate switch ports on the sidebar go to
``Switches`` → ``Switch Ports``. Choose a switch in the top right corner.
On the right of the port you want to change, click on the pencil icon to edit
the interface. Change the port ``Type`` to ``Core``.

"error saving config"
~~~~~~~~~~~~~~~~~~~~~

Permission error for saving the configuration. Follow the steps in
`Athos troubleshoot <athos_troubleshoot>`_ to ensure permissions are
configured correctly.

This error is only when storing the config directly and you are not using the
athos api


No output when running tests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Check Athos logs, the issue is most likely due to a failure detected in testing
that is taking too long to report back to Miru.


Clicking ``Start Tests`` does nothing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This indicates that there is an issue between Miru and Athos. Check the
:ref:`Athos troubleshoot <athos_troubleshoot>` section for more details.


Topology not saving
~~~~~~~~~~~~~~~~~~~

This indicates that there is an issue between Miru and Athos. Check the
:ref:`Athos troubleshoot <athos_troubleshoot>` section for more details.

.. _no_deploy:

No option to deploy
~~~~~~~~~~~~~~~~~~~

- Ensure that ``${IXPROOT}/config/custom.php``has ``cerberus`` and its
  ``api_url`` is configured properly


.. _athos_troubleshoot:

Athos
-----

The best way to troubleshoot athos is by running the athos docker container
outside of IXP Manager.

First ensure that you have the docker image for athos by running:

``docker images belthazaar/athos``

The latest image can also be pulled with ``docker image pull belthazaar/athos``.

Within ``$ATHOSROOT`` run ``runDocker.sh`` as root.

This will start the athos docker container and run the tests of the last given
topology.

If you are using the athos wrapper api, ensure that it is running with
``systemctl status athos_api.service``. If it is not running, run
``systemctl start athos_api.service``.

If you are not using the Athos wrapper API, ensure that your ``$MY_WWW_USER`` has
permission to run the ``runDocker.sh`` script within ``$ATHOSROOT`` and has read
and write permissions at minimum within the following locations:

- ``$ATHOSROOT/etc``
- ``$ATHOSROOT/ixpman_files``


Athos API wrapper
-----------------

First thing to check would be that the wrapper is running. If you are running
it as a service as described in the setup, you can check it's status by running
``systemctl status athos_api.service``. If not, you can always check the status
with ``ps aux | grep athos`` to find all instances of athos and athos api.


Wrapper Talking to Athos
~~~~~~~~~~~~~~~~~~~~~~~

The API wrapper will need to be run as either root, or a user that has docker
permissions with networking capabilities.

The quickest way to ensure that the wrapper API is working as intended is by
running the following:

.. code-block:: bash

  #Get the xml file for the topology
  curl -X GET http://127.0.0.1:8989/get_xml
  #Start and run an instance of Athos with output
  curl -X GET http://127.0.0.1:8989/run_athos

If this does not work check to ensure that the athos directory used in the
API wrapper is set to the directory where you have extracted Athos.
The default location for both of these is ``/athos``.

IXP Manager Configured correctly
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next would be to ensure that the configuration in
``${IXPROOT}/config/custom.php`` matches how the wrapper is configured to run.

Below is a sample config looking for the default address of the wrapper api at
``http://127.0.0.1:8989``.

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


Cerberus
--------

- First step is to ensure that Cerberus is running and using the latest version.
  Check the version with ``cerberus-controller --version`` and check it matches
  the latest version.

- Ensure that the Cerberus controller is running, if you have it setup as service
  check its status with ``sudo systemctl status cerberus.service``.

- Check configurations are set properly. The default configurations are:
  * ``/etc/cerberus/topology.json``
  * ``/etc/cerberus/failed/`` - Failed configurations location
  * ``/etc/cerberus/rollback/`` - Rollback configurations location

- Check the logs in ``/var/log/cerberus/cerberus.log``
