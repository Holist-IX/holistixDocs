HolistIX usage
==============

We integrate HolistIX into IXP Manager. The main interface to interact with the
HolistIX specific interface is called ``Miru``. To access Miru click on ``Miru``
at the bottom of the left sidebar.

.. note::

    If ``Miru`` is not located in the please ensure that you have followed the
    steps in `installing Miru <miru-install>`_.

Miru
~~~~

Switches
^^^^^^^^

When you first open up Miru, it will populate the graph sidebar with the active
switches configured in IXP Manager.

A switch object contains all the information related to it, including
hostname, IP address, model, switch name, core ports available, as well as all the
members connected to that switch.

Each switch object also contains the information of each member that is
connected to it, including port connected to, VLAN information, IPv4/6 addresses,
and MAC addresses.

To add a new switch to your topology, you can either drag a switch to onto the
diagram, or you can double click it and it will appear on the diagram.

Core Links
^^^^^^^^^^

To establish a connection between two switches, hover over your source switch,
click and drag the arrow to the destination switch to connect to.

Miru will look for core ports on both switches and assign them to the
link. If you want to change the ports used for the link, right click on the link,
click ``Edit Switch Ports`` and then change the ports.

Continue to draw your network topology so that it reflects your network.

.. note::

    Miru will auto update the details of members within the topology when
    changes are found within IXP Manager. However, the config will only be
    generated and deployed after the :ref:`verification process <athos usage>`
    is initiated.

    The topology within Miru only needs to be redrawn if there are changes in
    how the switches are interconnected.


.. _athos usage:

Verification (Athos)
~~~~~~~~~~~~~~~~~~~~

To generate the network configuration and run Athos, at the top left of the
diagram open ``File`` and then ``Start Athos``.

This starts the automated process of generating config files and starting the
Athos network emulator. Athos will run test the network topology that is
specified on the diagram.

Once completed, you will be presented with the option to download configs and
logs.

Deployment (Cerberus)
~~~~~~~~~~~~~~~~~~~~~

If the Cerberus API is configured in IXP Manager's
:ref:`custom config <sample-config>`, you will have the option to
``Deploy to production``. This will send the config to Cerberus which will
apply the config to the current running network.

If you want to have a closer look at the configuration that Cerberus is
currently using in the network, you can retrieve this within ``Miru``. Go to
``File`` and then ``Download Cerberus config``.

Cerberus also has a rollback feature if any issues arise when a new
configuration was deployed. This allows Cerberus to rollback to the
previously used configuration from within ``Miru``. To activate this go to
``File`` and then ``Rollback Cerberus Config``.
