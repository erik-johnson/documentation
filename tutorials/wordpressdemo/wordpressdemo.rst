Wordpress Demo
<<<<<<< HEAD
==============

.. toctree::
   :maxdepth: 1
   :hidden:

   overview
   prerequisites
   downloads
   create_firewall
   make_image
   upload_services
   upload_datasource
   design
   create_tiers
   add_services
   configure_ports
   configure_datasource
   dependencies
   launch_configuration
   prerequisites
   scaling_rules
   launch

.. figure:: ./images/demoLanding.png
=======
--------------

.. note:: Estimated time to complete this tutorial will vary based on several factors such
   as the speed of the cloud provider, and the skill level of the operator.

   New Guy, no SSH skills, no Linux: 1 mo.

   SSH/Firewall skills, cloud-savvy: 2 hrs, maybe less.

   Cloud god: 1 hr.

Purpose
~~~~~~~
The purpose of this document is to start from scratch and build a deployable enStratus
application architecture including a Wordpress web application backed by a MySQL database.

Prerequisites
~~~~~~~~~~~~~
Users seeking to successfully complete this tutorial must have the following items/skills.

#. A cloud account or cloud accounts connected to enStratus. EC2 is the cloud I will cover
   here.
#. Cloud files (object) storage
#. Familiarity with SSH.
   
   Users will need to be able to use the secure copy program (scp) or an equivalent to
   move the chef cookbook used to prepare the image.

#. Rudimentary familiarity with the Linux operating system command line. However, most of
   the instructions should be copy-and-paste.

Downloads
^^^^^^^^^

Chef solo cookbook: wordpress-demo-prep.tar.gz 
Wordpress service: wordpress.tar.gz
MySQL service: mysql.tar.gz
Datasource: wordpresscontent.sql

Overview
~~~~~~~~

.. note:: Some of the step are purposefully suboptimal to illustrate the process at hand.
   There are usually many ways to approach the processes we're describing, this is just
   one.

Create Firewall
~~~~~~~~~~~~~~~
Let's create a firewall (security group) for use in this tutorial.

.. figure:: ./images/firewall0.png
   :height: 400px
   :width: 800 px
   :scale: 50 %
   :alt: Create Firewall
   :align: center

   Create Firewall

Open ports 22 and 80 to your IP for testing. enStratus will automatically adjust the
firewall for the applications deployed according to the services > ports settings.

Make Image
~~~~~~~~~~
To make an image, start with a generic EC2 image from `Alestic <http://alestic.com/>`_.
Eric's team has recently begun publishing only 64-bit images. Any generic ubuntu image
*should* work, but this tutorial has only been tested with images from Alestic.

.. figure:: ./images/ami0.png
   :height: 400px
   :width: 1800 px
   :scale: 50 %
   :alt: Search for AMI
   :align: center

   Search for AMI

1. Start the image as a VM, choose a large product size, not t1.micro for best results.
   Also, generate and use an SSH key during the launch.

.. figure:: ./images/ami1.png
   :height: 700px
   :width: 800 px
   :scale: 50 %
   :alt: Launch, General Information
   :align: center

   Launch, General Information

Save the key and chmod it

.. code-block:: bash

   chmod 600 demoKey

.. figure:: ./images/ami3.png
   :height: 500px
   :width: 1200 px
   :scale: 50 %
   :alt: Launch, Create/Use Key
   :align: center

   Launch, Create/Use Key

2. While the image launches, open the firewall so you can access port 22 from your
   location.
3. Once the VM is started, copy the wordpress-demo-prep.tar.gz file to the instance.
   The command to do so will be something of the form:

.. code-block:: bash

   scp -i demoKey wordpress-demo-prep.tar.gz ubuntu@ip.of.running.instance:~

4. Next, ssh onto the running instance, and take root:

.. code-block:: bash

   ssh -i theKey ubuntu@ip.of.running.instance

   sudo su

5. Install the chef client:

.. code-block:: bash

   echo "deb http://apt.opscode.com/ `lsb_release -cs`-0.10 main" | sudo tee /etc/apt/sources.list.d/opscode.list
   sudo mkdir -p /etc/apt/trusted.gpg.d
   gpg --keyserver keys.gnupg.net --recv-keys 83EF826A
   gpg --export packages@opscode.com | sudo tee /etc/apt/trusted.gpg.d/opscode-keyring.gpg > /dev/null
   sudo apt-get update
   sudo apt-get -y upgrade
   sudo apt-get -y install chef

6. Extract the wordpress-demo-prep.tar.gz file:

.. code-block:: bash

   tar -xzf wordpress-demo-prep.tar.gz
	
7. Update the node.json and solo.rb files nameserver and cookbook location

.. code-block:: bash
	cd wordpress-demp-prep
	sudo vim node.json
	# Change nameserver and search in the 'resolver' node
	sudo vim solo.rb
	# Change the 'cookbook_path' to the extracted wordpress-demo-prep path
	# e.g. /tmp/wordpress-demo-prep/cookbooks
	

8. Execute the chef-solo run:

.. code-block:: bash

   chef-solo -j node.json -c solo.rb

During this step, some packages necessary for running a typical LAMP stack application
will be installed, along with the latest enStratus agent. Depending on your connection and
mirror speeds, this may take up to 5-7 minutes.

The purpose of this step is to prepare the image for running PHP and MySQL applications,
not to install the application itself, that comes later durin the launch and orchestration
steps of a deployment launch.

Once this step completes, initiate the build of the machine image from within the
enStratus console.

.. warning:: If the image is not built using the server actions > Make Image menu option
  in the enStratus console, it will not be available for use in the deployment. This measure
  is in place to protect users from attempting to use an image that does not have the agent
  on it for automation.

.. note:: As a sanity check that the agent is working, you should see an expanded list of
  options in the actions menu as shown.

.. figure:: ./images/makeImage1.png
   :height: 700px
   :width: 2200 px
   :scale: 45 %
   :alt: Server, Make Image
   :align: center

   Server, Make Image

Once this process completes, select action > Make Image from the server's action menu.

.. figure:: ./images/makeImage0.png
   :height: 300px
   :width: 700 px
   :scale: 50 %
   :alt: Make Image
   :align: center

   Make Image

While the image builds, it's time to upload the service images for use by enStratus.

Upload Services
~~~~~~~~~~~~~~~

Using Automation > Service Images, upload wordpress.tar.gz and mysql.tar.gz as service
image files. enStratus stores these files in cloud files storage and will initiate a
download of these files at launch time by the enStratus agent.

.. figure:: ./images/serviceUpload0.png
   :height: 300px
   :width: 700 px
   :scale: 50 %
   :alt: Service Upload, Wordpress
   :align: center

   Service Upload, Wordpress

Repeat this action for the mysql service.

.. figure:: ./images/serviceUpload1.png
   :height: 300px
   :width: 700 px
   :scale: 50 %
   :alt: Service Upload, MySQL
   :align: center

   Service Upload, MySQL

Once this action is complete, there will be (minimally) 2 services.

.. figure:: ./images/serviceUpload2.png
   :height: 500px
   :width: 2100 px
   :scale: 45 %
   :alt: Services Uploaded
   :align: center

   Services Uploaded

Upload DataSource
~~~~~~~~~~~~~~~~~

Using Automation > Datasource, upload wordpresscontent.tar.gz as datasource files.
enStratus stores datasource files in cloud files storage and will initiate a download of
this file to services that are configured to have datasources.

.. figure:: ./images/dataSourceUpload0.png
   :height: 300px
   :width: 700 px
   :scale: 50 %
   :alt: Data Source, Upload
   :align: center

   Data Source, Upload

After upload, the datasource will appear as shown:

.. figure:: ./images/dataSourceUpload1.png
   :height: 400px
   :width: 2100 px
   :scale: 45 %
   :alt: Data Source, Upload
   :align: center

   Data Source, Upload

Design Deployment
~~~~~~~~~~~~~~~~~

Now that those steps are complete, it's time to start building the application
architecture. 

First, create a new deployment by navigating to Automation > Designer. If you have not yet
created a deployment, a placeholder image greets you:

.. figure:: ./images/deployment0.png
   :height: 800px
   :width: 1500 px
   :scale: 50 %
   :alt: Deployment Designer
   :align: center

   Deployment Designer

To create a new deployment, select the actions menu and choose Create a New Deployment.

.. figure:: ./images/deployment1.png
   :height: 800px
   :width: 1500 px
   :scale: 50 %
   :alt: Deployment Designer, Create New Deployment
   :align: center

   Deployment Designer, Create New Deployment

And that's it, the deployment is created.

.. figure:: ./images/deployment2.png
   :height: 800px
   :width: 1500 px
   :scale: 50 %
   :alt: Deployment Designer, Create Tier
   :align: center

   Deployment Designer, Create Tier


Create Tiers
~~~~~~~~~~~~

1. Use the designer diagram to add a tier. Give the tier a high-level generic name like
   Application Tier. Tiers can hold many services, and we'll give a more specific name for
   the services. This tier will house only one service for this tutorial, the wordpress
   application.

.. figure:: ./images/deployment3.png
   :height: 750px
   :width: 1100 px
   :scale: 50 %
   :alt: Deployment Designer, Add Tier
   :align: center

   Deployment Designer, Add Tier

When the tier is added, the designer diagram is updated.

.. figure:: ./images/deployment4.png
   :height: 600px
   :width: 1300 px
   :scale: 50 %
   :alt: Deployment Designer, Add Tier
   :align: center

   Deployment Designer, Add Tier


2. Use the desginer diagram to add another tier. Give the tier a name like Database Tier.
   This tier will hold the MySQL service.

.. figure:: ./images/deployment5.png
   :height: 750px
   :width: 1100 px
   :scale: 50 %
   :alt: Deployment Designer, Add Tier
   :align: center

   Deployment Designer, Add Tier

When the tier is added, the designer diagram is updated.

.. figure:: ./images/deployment6.png
   :height: 600px
   :width: 1300 px
   :scale: 50 %
   :alt: Deployment Designer, Add Tier
   :align: center

   Deployment Designer, Add Tier

Add Services
~~~~~~~~~~~~
Adding services to tiers means telling enStratus what service should be installed on
servers running in the tier. This action only needs to be completed once, no matter how
many regions/clouds the tier spans.

To add a service to a tier, select the tier by clicking on it in the designer diagram and
choose +add service. enStratus will present the list of the services available for
attaching to the tier. This list should have at a minimum the two services uploaded above. 

.. figure:: ./images/addService0.png
   :height: 400px
   :width: 700 px
   :scale: 50 %
   :alt: Add Service, Wordpress
   :align: center

   Add Service, Wordpress

.. note:: If there was a previously running deployment where automated backups were made,
  it is also possible to tell enStratus to use a previously generated backup as a service.

  Pretty cool.

.. figure:: ./images/addService1.png
   :height: 400px
   :width: 700 px
   :scale: 50 %
   :alt: Add Service, MySQL
   :align: center

   Add Service, MySQL

Associate the wordpress service with the application tier and the mysql service with the
database tier.

.. figure:: ./images/addService2.png
>>>>>>> Added instructions to change solo.rb and nodes.json
   :height: 600px
   :width: 1150 px
   :scale: 70 %
   :alt: Wordpress Landing Page
   :align: center

   Wordpress Landing Page
