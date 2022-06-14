Setup
=====

Networking
----------

**Worker nodes**

In order to statically define the IP addresses in the network, the following snippet is appended to `/etc/dhcpcd.conf` file of the worker nodes.

.. code-block::

    # To be appended to /etc/dhcpcd.conf

    interface eth0
    static ip_address=172.22.69.1*/24	# Change IP according to the node
    static routers=172.22.69.1
    static domain_name_server=8.8.8.8

.. attention::
    Replace the asterisk in the previous code block with the node number.