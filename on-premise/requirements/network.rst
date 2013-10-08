.. _enstratus_communications:

Networking
----------

.. note::
    See also the :ref:`supported configurations 'network' section <supported_configuration_network>`

Enstratius components must have direct communications with each other. To enable Enstratius agent communication on the guest VM, please make the following considerations:

Dispatcher Service 
~~~~~~~~~~~~~~~~~~

The dispatcher service listens on port 3302. Provisioned (guest) virtual machines that have the agent installed upon them must be able to reach the dispatcher service on this port.

Agent
~~~~~

The DMCM agent listens on port 2003 and can be installed on guest virtual machines.
The server(s) upon which the subscriber, monitor, and dispatcher services run must be able to communicate to the agent on this port.

To summarize:

* Communication from guest virtual machines to the dispatcher server on port 3302

* Communication from the subscribers, monitors, and the dispatcher service to the guest virtual machines on port 2003

.. note::
   If the Agent Proxy is being used then only traffic to and from the agent proxy needs to be allowed in place of "guest virtual machines". In AP environments the subscriber/dispatcher/monitor communication to the DMCM agent happens **through** the agent proxy. Inbound communications from guest virtual machines to the dispatcher also then happen through the agent proxy.

DMCM Communications
~~~~~~~~~~~~~~~~~~~~~~~~~

The purpose of this section is to clarify the communication channels required by the individual DMCM components.
Ports listed are default and/or standard service ports and are subject to any customizations made by the installing engineers.

Enstratius Internal/External Communications
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tabularcolumns:: |l|l|p{5cm}|p{5cm}|

+--------------------+---------------------------+----------------------------------------------------+-----------------------------------------------------------------------------------------------+
| Component          | Listens Port              | Accepts From                                       | Initiates To                                                                                  | 
+====================+===========================+====================================================+===============================================================================================+
| Console            | 443 (https)               | Internet (or equivalent)                           | LDAP/AD, Dispatcher, MySQL, Riak (or Riak loadbalancer)                                       |
+--------------------+---------------------------+----------------------------------------------------+-----------------------------------------------------------------------------------------------+
| KM                 | 2013 (ssl)                | All DMCM Components                                | MySQL, Riak (or Riak loadbalancer)                                                            |
+--------------------+---------------------------+----------------------------------------------------+-----------------------------------------------------------------------------------------------+
| Dispatcher         | 3302 (ssl)                | Internet (guest vm), API, Monitor, Worker, Console | MySQL, Riak (or Riak loadbalancer), Rabbit MQ, KM, Cloud API, guest VMs                       |
+--------------------+---------------------------+----------------------------------------------------+-----------------------------------------------------------------------------------------------+
| MySQL              | 3306                      | All DMCM Components                                | N/A                                                                                           |
+--------------------+---------------------------+----------------------------------------------------+-----------------------------------------------------------------------------------------------+
| Riak               | 8098                      | All DMCM Compenents                                | N/A                                                                                           |
+--------------------+---------------------------+----------------------------------------------------+-----------------------------------------------------------------------------------------------+
| Monitor/Worker     | N/A                       | N/A                                                | Cloud API, Dispatcher, MySQL, Riak (or Riak loadbalancer), Rabbit MQ, KM, guest VMs           |
+--------------------+---------------------------+----------------------------------------------------+-----------------------------------------------------------------------------------------------+
| RabbitMQ           | 5672                      | Dispatcher, Monitor, Worker                        | N/A                                                                                           |
+--------------------+---------------------------+----------------------------------------------------+-----------------------------------------------------------------------------------------------+
| API                | 15443 (https)             | Internet (or equivalent)                           | KM, Dispatcher, MySQL, Riak (or Riak loadbalancer)                                            |
+--------------------+---------------------------+----------------------------------------------------+-----------------------------------------------------------------------------------------------+

