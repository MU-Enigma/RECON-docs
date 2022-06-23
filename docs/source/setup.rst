Setup
=====

----------
Networking
----------

Following is the information on the networking setup in the Raspberry-Pis. 


Worker nodes
------------

In order to statically define the IP addresses in the network, the following snippet is appended to ``/etc/dhcpcd.conf`` file of the worker nodes.

.. code-block::

    # To be appended to /etc/dhcpcd.conf

    interface eth0
    static ip_address=172.22.69.1*/24	# Change IP according to the node
    static routers=172.22.69.1
    static domain_name_server=8.8.8.8

.. attention::
    Replace the asterisk in the previous code block with the node number.

|

Master node
-----------

Similar to what was done in the previous sub-section, we append the next code snippet to ``/etc/dhcpcd.conf`` of the master node. It allows the cluster to recognize the universty network
and vice-versa. It also allows the worker nodes to recognize the fact that the master node is on VLAN 10 (lines 3 and 8). 

.. code-block::
    :linenos:

    # To be appended to /etc/dhcpcd.conf

    interface eth0
    static ip_address=10.59.114.70/23 # IP address of the university network
    static routers=10.59.114.1 # IP address of the university router
    static domain_name_server=8.8.8.8

    interface eth0.10
    static ip_address=172.22.69.1/24 # IP address of the master node
    static routers=172.22.69.1 
    static domain_name_servers=8.8.8.8

.. caution::
    Do not use the transparency here with malicious intent.

|

The following must be appended to ``/etc/network/interfaces.d/vlans``. It creates the virtual interfaces which are then recognized by the network. 

.. code-block::

    # To be appended to /etc/network/interfaces.d/vlans 

    auto eth0.10
    iface eth0.10 inet manual
        vlan-raw-device eth0

|

The following bash script is to setup IP forwarding on the master node.

.. code-block:: 

    #! /bin/bash

    IPTABLES=/sbin/iptables

    WANIF='eth0'
    LANIF='eth0.10'

    # enable ip forwarding in the kernel
    #echo 'Enabling Kernel IP forwarding...'
    #/bin/echo 1 > /proc/sys/net/ipv4/ip_forward

    # flush rules and delete chains
    echo 'Flushing rules and deleting existing chains...'
    $IPTABLES -F
    $IPTABLES -X

    # enable masquerading to allow LAN internet access
    echo 'Enabling IP Masquerading and other rules...'
    $IPTABLES -t nat -A POSTROUTING -o $LANIF -j MASQUERADE
    $IPTABLES -A FORWARD -i $LANIF -o $WANIF -m state --state RELATED,ESTABLISHED -j ACCEPT
    $IPTABLES -A FORWARD -i $WANIF -o $LANIF -j ACCEPT

    $IPTABLES -t nat -A POSTROUTING -o $WANIF -j MASQUERADE
    $IPTABLES -A FORWARD -i $WANIF -o $LANIF -m state --state RELATED,ESTABLISHED -j ACCEPT
    $IPTABLES -A FORWARD -i $LANIF -o $WANIF -j ACCEPT

    # Make sure to install the package 'iptables-persistent' to make the routing persistent across reboots