.. _single_node_install:

Single Node
-----------

This is the Enstratius on-premise install built using Opscode chef-solo

.. warning:: 

   By default, this repository will work **OUT OF THE BOX** to install a standalone server
   hosting all the Enstratius components.  This is by design as the initial and primary use
   case is for testing and POC.

   This installer **CAN** be used to install multi-node production installs but it
   requires more manual configuration.


.. _install_quick_start:

Quick Start
~~~~~~~~~~~

Requirements
^^^^^^^^^^^^

#. Enstratius license key
#. Enstratius download password
#. Enstratius bootstrap tool
#. A domain name for the installation

For the quick start, let's assume:

license_key=abcd1234
download_password=wxyz6789
domain_name=cloud.mycompany.com

To perform the installation, as root:

.. code-block:: bash

   ./setup -p wxyz6789 -l abcd1234 -c cloud.mycompany.com

The install will take a few moments to complete, the installation. Once completed, you
should be able to :ref:`register`
   
Components
~~~~~~~~~~

Enstratius software can be installed on a single server, combining all software components:

1. Dispatcher

  * Provisioning Database (MySQL)
  * Analytics Database (MySQL)

2. Workers
3. Monitors
4. Key Manager

  * Credentials Database (MySQL)

5. Riak
6. Rabbit MQ

7. Console

  * Console Database (MySQL)

8. API
9. Console Worker
10. LDAP


System Requirements
~~~~~~~~~~~~~~~~~~~

Provision a server (can be virtual, we often test using an m1.large instance in EC2) for
the installation. Installation can also be done using local virtualization. These specs
are for a single node installation

Recommended Requirements
~~~~~~~~~~~~~~~~~~~~~~~~

* 4 VCPU
* 15GB of memory
* 60GB of storage (this is primarly logging and data)
* 64-bit Ubuntu 12.04

External connectivity
~~~~~~~~~~~~~~~~~~~~~

**This installation requires external Internet connectivity.**

The installer pulls all requirements from the Internet, including vendor OS repositories.
You only need to download the packages once. They will be cached the first run in the
`cache` directory.  

Chef will reuse those packages if the checksums match the defined
ones.

Installation
~~~~~~~~~~~~

The following assumes competence with Linux, and command line utilities like ssh. If you
don't use these utilities on a near daily basis, this installer is probably not for you.

Extract Chef-Solo Package
~~~~~~~~~~~~~~~~~~~~~~~~~

The installation package will be provided to you by an Enstratius engineer.
Extract it.

The contents of the directory will be similar to:

.. hlist::
   :columns: 3

   * README.md
   * TODO.md
   * VAGRANT.md
   * Vagrantfile
   * backup
   * cache
   * checksums
   * classes
   * cookbooks
   * data_bags
   * docs
   * enstratus-utilities.jar
   * json_templates
   * local_settings
   * roles
   * setup.sh
   * solo.rb

.. _running_setup:

Running the setup
~~~~~~~~~~~~~~~~~

The setup script is designed to work out of the box with the single-node
installation. There is a `setup.sh` script provided that will do configuration for
you. At a minimum, `setup.sh` needs two settings passed to it:

#. Your license key 
#. Download password for the Enstratius software. 

These should have been provided to you by Enstratius.

Optionally, you can specify a `savedir` where you would like to save your settings.

Help output
^^^^^^^^^^^

.. code-block:: bash

   Usage: setup.sh [-h] [-e] -p <download password> -l <license key> [-s savename] [-c <console hostname>] 
   [-n <number of nodes>] [-m <mapping string>] [-a <optional sourceCidr string>]

   -p: The password for downloading Enstratius
   -l: The license key for Enstratius
   
   For most single node installations, specify the download password and license key.
   
   optional arguments
   ------------------
   -h: This text
   -e: extended help
   -c: Alternate hostname to use for the console. [e.g. cloud.mycompany.com] (default: fqdn
   of console node)
   -a: Alternate string to use for the sourceCidr entry. You know if you need this.
   -s: A name to identify this installation
   -n: Number of nodes in installation [1,2,4] (default: 1)
   -m: Mapping string [e.g. frontend:192.168.1.1,backend:backend.mydomain.com]

For a single node, most users should run something similar to

.. code-block:: bash

  ./setup -p <the_password_here> -l <license_key_here> -c cloud.mycompany.com


Running without a savedir
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   root@host# ./setup.sh -l XXXX -p YYYYY
   Savedir not specified. Using temporary directory
   Generating Keys
   Creating local_settings//tmp/tmp.KZ1vPP28lG/genkeys.txt file
   Writing JSON files to 'local_settings//tmp/tmp.KZ1vPP28lG/'
   #Ready to run :
   #
   chef-solo -j local_settings//tmp/tmp.KZ1vPP28lG/single_node.json -c solo.rb

Running with a savedir
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   root@host# ./setup.sh -s my_local_install -l XXXX -p YYYYY
   Savedir my_local_install not found. Assuming new run...
   
   Generating Keys
   Creating local_settings/my_local_install/genkeys.txt file
   Writing JSON files to 'local_settings/my_local_install/'
   #Ready to run :
   #
   chef-solo -j local_settings/my_local_install/single_node.json -c solo.rb

Running the install with a previous savedir
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   root@host# ./setup.sh -s my_local_install -l XXXX -p YYYYY
   Savedir my_local_install found..
   
   Existing config in use. Skipping password generation
   Reading existing keys from ./local_settings/my_local_install/
   Writing JSON files to 'local_settings/my_local_install/'
   #Ready to run :
   #
   chef-solo -j local_settings/my_local_install/single_node.json -c solo.rb

This is CRITICAL if you want to be able to rerun the installation on the same machine.
The installer uses chef-solo. Chef-solo does not persist any state between invocations in
the same way that chef with a Chef server does. The `setup.sh` script is designed to allow
you to persist that state between runs.
