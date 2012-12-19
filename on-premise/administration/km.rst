Key Manager
===========

The enStratus Key/Credentials Management service is a jetty service installed to
/services/km. 

The enStratus KM service is very stable and will run for very long periods of time
without requiring attention.

.. figure:: ./images/km.png
   :height: 250 px
   :width: 450 px
   :scale: 70 %
   :alt: KM Service
   :align: center

   KM Service Connections

Overview
--------

The enStratus KM service is a jetty process that listens on port 2013, configurable. It
accepts connections from enStratus components that have as an operational requirement to
perform activities that require access to sensitive information.

Sensitive information can be thought of as any information required to interact with
cloud providers, for example, the credentials necessary to perform the action of
initiating the start of a VM in a cloud provider.

The KM system also handles information entered by users via the enStratus console or API.
An example of this type of information is a public ssh key that may be a part of an
enStratus user profile or an SSL certificate that was loaded into enStratus for use in an
automation routine.

The KM system stores this information in an encrypted, de-identified database called the
credentials database. 

Credentials Database
--------------------

The credentials database stores the encrypted, de-identified information described above.
The credentials database is currently a MySQL database, but will be migrated to Riak in a
future release.

The enStratus KM system is the **only** enStratus component to interact with the
credentials database.

The credentials database should be considered to be the only enStratus component that,
should it be compromised or destroyed, would constitute an irrecoverable failure. As such,
care should be taken to ensure the data contained therein is backed up and safeguarded in
accordance with industry standard procedures.

Installation
------------

Installation of the enStratus KM service is best handled by using a configuration
management system such as Chef or Puppet.

Software Requirements
---------------------

The KM service depends on the Open JDK and runs as a Jetty service.

Incoming Connections
--------------------

#. Dispatcher

   The enStratus dispatcher component connects to the KM service via a webservices
   endpoint, defined on the dispatcher service in:
  
   ``/services/dispatcher/resources/enstratus-km-client.cfg``

#. Monitor

   The enStratus monitor component connects to the KM service via a webservices
   endpoint, defined on the monitor service in:

   ``/services/monitor/resources/enstratus-km-client.cfg``

#. Worker

   The enStratus worker component connects to the KM service via a webservices
   endpoint, defined on the worker service in:

   ``/services/worker/resources/enstratus-km-client.cfg``

Outgoing Connections
--------------------

None.

Customizing
-----------

The service port upon which the enStratus KM service listens and the Java options it uses
to start the jetty service can be modified.

Service Port
~~~~~~~~~~~~

The service port upon which the KM service operates is defined in:

``/services/km/bin/km``

Logging
-------

Logging for the km service is done to:

``/services/dispatcher/logs/YYYY_MM_DD.request.log``

``/services/dispatcher/logs/km.log``

Monitoring
----------

Backups
-------

Service
~~~~~~~

The enStratus KM service files should be backed up before and after any changes, and
once/day during steady-state operations.

.. code-block:: bash

   #!/bin/bash
   
   TAR=/bin/tar
   GZIP=/bin/gzip
   
   DIR=/var/enstratus/backups
   BASE=km
   DA=`date +%Y%m%d-%H%M%S`
   
   FILE=${DIR}/${BASE}-${DA}.tar.gz
   
   find ${DIR} -type f -iname "*.gz" -mtime +2 | xargs rm -f
   
   cd /services/${BASE}
   $TAR -czf ${FILE}  --exclude='log/*' . > /dev/null 2>&1
   chmod 700 ${FILE}


Database
~~~~~~~~

The frequency with which the enStratus credentials database is backed up is determined
primarily by the number of writes being made to the database. enStratus environments where
there are many new accounts being joined to enStratus, many new users being added or
modified should conduct backups more frequently than environments where these events are
less frequent.

As a general best practice guideline, backups should be done no less frequent than twice
daily, every four hours in heavily utilized systems, or more frequently as the situation
dictates.

Backups should be encrypted and stored in a geographically unique location from the
primary data source.

The expected time to run a backup of the credentials database is less than one minute. 

The expected time to restore the credentials database from backup less than one minute.

Starting KM
-----------

To start the Key Management service:

.. code-block:: bash

	/etc/init.d/enstratus-km start

The KM service will start, and you should see a java service running on port 2013.

.. code-block:: bash

	netstat -tnlup | grep 2013
	tcp6       0      0 :::2013                 :::*                    LISTEN 7159/java  

Stopping KM
-----------

To stop the Key Management service:

.. code-block:: bash

	/etc/init.d/enstratus-km stop

Configuration Files
-------------------

The KM service has two configuration files in /services/km/resources/

#. enstratus-context.xml
#. log4j.properties

enstratus-context.xml
~~~~~~~~~~~~~~~~~~~~~

The full path to the context xml configuration file is:

``/services/km/resources/enstratus-context.xml``

This file is responsible for controlling how the KM service connects to the credentials
database.

log4j.properties
~~~~~~~~~~~~~~~~

The full path to the server.xml configuration file is:

``/services/km/resources/log4j.properties``

This file is responsible for controlling the logging behavior for the KM service.
