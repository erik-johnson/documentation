.. _two_node:

Two Node
--------

.. figure:: ./images/two_node.png
   :height: 400px
   :width: 600 px
   :scale: 95 %
   :alt: Enstratius Two Node Architecture
   :align: center

   Enstratius Two Node Architecture

In a two-node installation, Enstratius components are grouped into so-called frontend and
backend categories.

**Frontend**

1. Console

  * Console Database (MySQL)
  * Enstratius Console Database (MySQL)

2. API
3. Console Worker

**Backend**

1. Dispatcher

  * Provisioning Database (MySQL)
  * Analytics Database (MySQL)

2. Workers
3. Monitors
4. Key Manager

  * Credentials Database (MySQL)

5. Riak
6. Rabbit MQ
