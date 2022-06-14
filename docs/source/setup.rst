====
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

.. admonition::
    Do not use the transparency here with malicious intent.