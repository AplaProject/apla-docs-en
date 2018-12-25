Apla blockchain deployment
^^^^^^^^^^^^^^^^^^^^^^^^^^

This howto shows how to deploy your own blockchain network.

.. todo::
    
    If you want to join the Apla blockchain network, see FIXME


Example deployment
##################

This guide deploys the example blockchain network of three nodes. 

The network has three nodes: 

    - Node 1 is the first node in the blockchain network. It can generate new blocks and send transactions from the clients connected to it.

    - Node 2 is an extra validating node. It can generate new blocks and send transactions from the clients connected to it.
    
    - Node 3 is an extra node. It cannot generate new blocks but can send transactions from the clients connected to it. 

Nodes are deployed in this configuration: 

    - Each node will use its own instance of PostgreSQL database system.

    - Each node will use its own instance of Centrifugo service.

    - The go-apla services will be deployed on the same host with other backend components.


Addresses and ports used by nodes are described in the following table:

.. list-table::
   :header-rows: 1
   :widths: auto

   * - Node
     - Component
     - IP and Port
   * - 1
     - PostgreSQL
     - 127.0.0.1:5432
   * - 1
     - Centrifugo
     - 10.10.99.1:8000
   * - 1
     - go-apla (TCP-server)
     - 10.10.99.1:7078
   * - 1
     - go-apla (API-server)
     - 10.10.99.1:7079
   * - 2
     - PostgreSQL
     - 127.0.0.1:5432
   * - 2
     - Centrifugo
     - 10.10.99.2:8000
   * - 2
     - go-apla (TCP-server)
     - 10.10.99.2:7078
   * - 2
     - go-apla (API-server)
     - 10.10.99.2:7079
   * - 3
     - PostgreSQL
     - 127.0.0.1:5432
   * - 3
     - Centrifugo
     - 10.10.99.3:8000
   * - 3
     - go-apla (TCP-server)
     - 10.10.99.3:7078
   * - 3
     - go-apla (API-server)
     - 10.10.99.3:7079


Deployment stages
#################

Your own blockchain network must be deployed in several stages:

.. todo::
    
    Links

1. Backend depolyment

    1. Deploying the first node
    2. Deploying other nodes

2. Frontend deployment

3. Blockchain network configuration

    1. Creating the founder's account
    2. Importing apps, roles, and templates
    3. Adding first node to the list of nodes

4. Adding extra validating nodes

    1. Creating the node owner's account
    2. Adding the validating node via voting 


Backend deployment
##################

Deploying the first node
========================

.. _dependencies:

Dependencies and environment setup
----------------------------------

sudo
""""

All commands for Debian 9 must be run as a non-root user. But some system commands need superuser privileges to be executed. By default, sudo is not installed on Debian 9, and you must install it first.

1) Become the root superuser.

.. code-block:: bash

    su -


2) Upgrade your system.

.. code-block:: bash
    
    apt update -y && apt upgrade -y && apt dist-upgrade -y

3) Install sudo.

.. code-block:: bash

    apt install sudo -y


4) Add your user to the sudo group.

.. code-block:: bash
    
    usermod -a -G sudo user

5) After the reboot, the changes take effect.


Go language
"""""""""""

Install Go as described in the `official documentation <https://golang.org/doc/install#tarball>`_).


1) Download the latest stable version of Go (1.11.2) from the `official site <https://golang.org/dl/>`_ or via the command line:

.. code-block:: bash

    wget https://dl.google.com/go/go1.11.2.linux-amd64.tar.gz

2) Extract the package to ``/usr/local``.

.. code-block:: bash

    tar -C /usr/local -xzf go1.11.2.linux-amd64.tar.gz


3) Add ``/usr/local/go/bin`` to the PATH environment variable (either to ``/etc/profile`` or ``$HOME/.profile``).

.. code-block:: bash

    export PATH=$PATH:/usr/local/go/bin


4) For changes to take effect, ``source`` this file. For example:

.. code-block:: bash
    
    source $HOME/.profile


5) Remove the temporary file:

.. code-block:: bash

    rm go1.11.2.linux-amd64.tar.gz


PostgreSQL
""""""""""

1) Install PostgreSQL and psql:

.. code-block:: bash

    sudo apt install -y postgresql


Centrifugo
""""""""""

1) Download Centrifugo version 1.7.9 `from GitHub <https://github.com/centrifugal/centrifugo/releases/>`_ or via command line:


.. code-block:: bash

    wget https://github.com/centrifugal/centrifugo/releases/download/v1.7.9/centrifugo-1.7.9-linux-amd64.zip \
    && unzip centrifugo-1.7.9-linux-amd64.zip \
    && mkdir centrifugo \
    && mv centrifugo-1.7.9-linux-amd64/* centrifugo/


2) Remove temporary files:

.. code-block:: bash

    rm -R centrifugo-1.7.9-linux-amd64 \
    && rm centrifugo-1.7.9-linux-amd64.zip


Directories
"""""""""""

For Debian 9 OS, it is recommended to store all software used by Apla blockchain platform in a separate directory.

In this guide, we will use ``/opt/apla`` directory, but you can use any directory. In this case, change all commands and configuration files accordingly.

1) Make a directory for Apla:

.. code-block:: bash

    sudo mkdir /opt/apla

2) Make your user the owner of this directory:

.. code-block:: bash

    sudo chown user /opt/apla/

3) Make subdirectories for Centrifugo, go-apla, and node data. In this guide, all node data is stored in the directories with ``nodeX`` name, where ``X`` is the node number. Depending on which node you are deploying, this will be ``node1`` for node 1, ``node2`` for node 2, and so on.

.. code-block:: bash

    mkdir /opt/apla/go-apla \
    mkdir /opt/apla/go-apla/node1 \
    mkdir /opt/apla/centrifugo \


.. _database:

Creating the database
---------------------

1) Change user's password postgres to Apla's default. You can set your own password, but then you also must change it in the node configuration file config.toml.

.. code-block:: bash

    sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'apla'"


2) Create a node current state database, for example 'apladb':

.. code-block:: bash

    sudo -u postgres psql -c "CREATE DATABASE apladb"

.. _centrifugo:

Configuring Centrifugo
----------------------

1) Create Centrifugo configuration file:

.. code-block:: bash

    echo '{"secret":"CENT_SECRET"}' > /opt/apla/centrifugo/config.json

You can set your own "secret", but then you also must change it in the node configuration file config.toml.

.. _go-apla-install:

Installing go-apla
------------------

1) Download and build the `latest release of go-apla <https://github.com/AplaProject/go-apla/releases>`_ from GitHub:

.. code-block:: bash

    go get -v github.com/AplaProject/go-apla

2) Copy the go-apla binary to the ``/opt/apla/go-apla`` directory. If you use the `default Go workspace <https://golang.org/doc/code.html#Workspaces>`_ then the binary is located in the ``$HOME/go/bin`` directory:

.. code-block:: bash

    cp $HOME/go/bin/go-apla /opt/apla/go-apla


Configuring the first node
--------------------------

1) Create the node 1 configuration file:

.. code-block:: bash

    /opt/apla/go-apla/go-apla config \
        --dataDir=/opt/apla/go-apla/node1 \
        --dbName=apladb \
        --centSecret="CENT_SECRET" --centUrl=http://10.10.99.1:8000 \
        --httpHost=10.10.99.1 \
        --httpPort=7079 \
        --tcpHost=10.10.99.1 \
        --tcpPort=7078

4) Generate node 1 keys:

.. code-block:: bash

    /opt/apla/go-apla/go-apla generateKeys \
        --config=/opt/apla/go-apla/node1/config.toml

5) Generate the first block:

.. note:: 
    
    If you are creating your own blockchain network. you must use the ``--test=true`` option. Otherwise you will not be able to create new accounts.

.. code-block:: bash

    /opt/apla/go-apla/go-apla generateFirstBlock \
        --config=/opt/apla/go-apla/node1/config.toml \
        --test=true

6) Initialize the database:

.. code-block:: bash

    /opt/apla/go-apla/go-apla initDatabase \
        --config=/opt/apla/go-apla/node1/config.toml


Starting the first node backend
-------------------------------

.. _services: https://wiki.debian.org/systemd/Services

To start the first node backend, you must start two services:

-   centrifugo
-   go-apla

If you did not create these as `services`_, you can just execute binary files from their directories in different consoles.

1) Run centrifugo:

.. code-block:: bash

    /opt/apla/centrifugo/centrifugo \
        -a 10.10.99.1 -p 8000 \
        --config /opt/apla/centrifugo/config.json


2) Run go-apla:

.. code-block:: bash

    /opt/apla/go-apla/go-apla start \
        --config=/opt/apla/go-apla/node1/config.toml


Deploying additional nodes
==========================

All other nodes are deployed like the first node with three differences:

- You do not need to generate the first block. Instead, it must be copied to the node data directory from node 1.
- The node must be configured to download blocks from node 1 via ``--nodesAddr`` option.
- The node must be configured to use its own addresses and ports.


Follow this sequence of actions:

    1. :ref:`dependencies`
    2. :ref:`database`
    3. :ref:`centrifugo`
    4. :ref:`go-apla-install`


    5. Create the node 2 configuration file:

        .. code-block:: bash

            /opt/apla/go-apla/go-apla config \
                --dataDir=/opt/apla/go-apla/node2 \
                --dbName=apladb \
                --centSecret="CENT_SECRET" --centUrl=http://10.10.99.2:8000 \
                --httpHost=10.10.99.2 \
                --httpPort=7079 \
                --tcpHost=10.10.99.2 \
                --tcpPort=7078 \
                --nodesAddr=10.10.99.1

    6. Copy the first block file to Node 2. For example, you can do it via ``scp`` on Node 2:

        .. code-block:: bash
            
            scp user@10.10.99.1:/opt/apla/go-apla/node1/1block /opt/apla/go-apla/node2/


    7. Generate node 2 keys:

        .. code-block:: bash

            /opt/apla/go-apla/go-apla generateKeys \
                --config=/opt/apla/go-apla/node2/config.toml

    8. Initialize the database:

        .. code-block:: bash
        
            ./go-apla initDatabase --config=node2/config.toml

    9. Run centrifugo:

        .. code-block:: bash

            /opt/apla/centrifugo/centrifugo \
                -a 10.10.99.2 -p 8000 \
                --config /opt/apla/centrifugo/config.json


    10. Run go-apla:

        .. code-block:: bash

            /opt/apla/go-apla/go-apla start \
                --config=/opt/apla/go-apla/node2/config.toml


Frontend Deployment
###################

Molis client can be build by the yarn package manager only on Debian 9 (Stretch) 64-bit [official distributive](https://www.debian.org/CD/http-ftp/#stable) with **installed GNOME GUI**.


Software Prerequisites
======================

Node.js
-------

1) Download Node.js LTS version 8.11 from the [official site](https://nodejs.org/en/download/) or via the command line:

.. code-block:: bash

    curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash


2) Install Node.js:

.. code-block:: bash

    sudo apt install -y nodejs


Yarn
----

1) Download Yarn version 1.7.0 from [GitHub](https://github.com/yarnpkg/yarn/releases) or via command line:

.. code-block:: bash

    cd /opt/apla \
    && wget https://github.com/yarnpkg/yarn/releases/download/v1.7.0/yarn_1.7.0_all.deb

2) Install Yarn:

.. code-block:: bash

    sudo dpkg -i yarn_1.7.0_all.deb && rm yarn_1.7.0_all.deb


Building Molis App
==================

1) Download latest release of Molis from [GitHub](https://github.com/AplaProject/apla-front) via git:

.. code-block:: bash

    cd /opt/apla \
    && git clone https://github.com/AplaProject/apla-front.git

2) Install Molis dependencies via Yarn:

.. code-block:: bash

    cd /opt/apla/apla-front/ \
    && yarn install


.. _front-connections:

Adding the blockchain network configuration
-------------------------------------------

1) Create settings.json file that contains connections information about full nodes:

.. code-block:: bash

    cp /opt/apla/apla-front/public/settings.json.dist \
        /opt/apla/apla-front/public/public/settings.json

2) Edit settings.json file in any text editor and add required settings in this format:

.. code-block:: text

    http://Node_IP-address:Node_HTTP-Port


**Example** settings.json file for three nodes:

.. code-block:: json

    {
        "fullNodes": [
            "http://10.10.99.1:7079",
            "http://10.10.99.2:7079",
            "http://10.10.99.3:7079"
        ]
    }


Building as Molis Desktop App
-----------------------------

1) Build the desktop app with Yarn:

.. code-block:: bash
    
    cd /opt/apla/apla-front \
    && yarn build-desktop

2) The desktop app must be packed to the AppImage:

.. code-block:: bash

    yarn release --publish never -l


After that, your application will be ready to use, but its :ref:`connection settings <front-connections>` cannot be changed in the future. If these settings will change, you must build a new version of the application.


Building as Molis Web App
-------------------------

1) Build the web app:

.. code-block:: bash
    
    cd /opt/apla/apla-front/ \
    && yarn build


After building, redistributable files will be placed to the '/build' directory. You can serve it with any web-server of your choice. Settings.json file must be also placed there. Note that you do not need to build your application again if your connection settings will change. Instead, edit the settings.json file and restart web-server.

2) For development or testing purposes, you can build Yarn's web-server:

.. code-block:: bash

    sudo yarn global add serve \
    && serve -s build

After this, your Molis Web App will be available at: ``http://localhost:5000``


Blockchain network configuration
################################

Creating the founder's account
==============================

Create an account for the first node owner. This account is the founder of the new blockchain platform and will have admin access.

1) Run Molis (frontend)

2) Import an existing account using the following data:

    - Backup payload is the node owner's public key located in the ``/opt/apla/go-apla/node1/PublicKey`` file.

        .. note::

            There are two public key files in this directory. ``PublicKey`` file is for node owner's account, ``NodePublicKey`` is the public key of the node itself. It is used later in this guide.

3) Login under this new account. Because roles haven't been created at this moment, use the *Without role* login option.

Importing apps, roles, and templates
====================================

At this moment, the blockchain platform is in the blank state. You can configure it by adding the framework of roles, templates and apps that support the basic ecosystem functions.

1) Clone the applications repository.

.. code-block:: bash

    cd /opt/apla \
    && git clone https://githu

2) In Molis, navigate to *Developer* > *Import*.

3) Import apps in this order:

    A. /opt/apla/apps/lang_res.json
    B. /opt/apla/apps/system.json
    C. /opt/apla/apps/basic.json
    D. /opt/apla/apps/conditions.json

4) Navigate to *Admin* > *Roles* and click *Install default roles*.

5) Sign out of the system via the profile menu.

6) Log into the system under the *Admin* role.

7) Navigate to *Home* > *Votings* -> *Templates list* and click *Install default templates*.

Adding first node to the list of nodes
======================================


1) Navigate to *Admin* > *Platform parameters* and click the cogwheel icon for the *full_nodes* parameter.

2) Specify the parameters for the first blockchain network node. Node's public key is located in the ``/opt/apla/go-apla/node1/NodePublicKey`` file. Node owner's key indentifier is located in the ``/opt/apla/go-apla/node1/KeyID`` file.

.. code-block:: json

    {"api_address":"http://10.10.99.1:7079","key_id":"%node_owner_key_id%","public_key":"%node_public_key%","tcp_address":"10.10.99.1:7078"}


Adding extra validating nodes
#############################


Creating the node owner's account
=================================

.

Adding the validating node via voting
=====================================

.
