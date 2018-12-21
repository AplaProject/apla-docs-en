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

    1. Deploying the frontend

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

Install Go as described in the [official documentation](https://golang.org/doc/install#tarball).


1) Download the latest stable version of Go (1.11.2) from the [official site](https://golang.org/dl/) or via the command line:

.. code-block:: bash

    wget https://dl.google.com/go/go1.11.2.linux-amd64.tar.gz

2) Extract the package to `/usr/local`.

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

1) Download Centrifugo version 1.7.9 from [GitHub](https://github.com/centrifugal/centrifugo/releases/) or via command line:


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

In this guide, we will use `/opt/apla` directory, but you can use any directory. In this case, change all commands and configuration files accordingly.

1) Make a directory for Apla:

.. code-block:: bash

    sudo mkdir /opt/apla

2) Make your user the owner of this directory:

.. code-block:: bash

    sudo chown user /opt/apla/

3) Make subdirectories for Centrifugo, go-apla, frontend, apps and node data. In this guide, all node data is stored in the directories with ``nodeX`` name, where ``X`` is the node number. Depending on which node you are deploying, this will be ``node1`` for node 1, ``node2`` for node 2, and so on.

.. code-block:: bash

    mkdir /opt/apla/go-apla \
    mkdir /opt/apla/go-apla/node1 \
    mkdir /opt/apla/centrifugo \
    mkdir /opt/apla/apps \
    mkdir /opt/apla/apla-front


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

.. _latest release of go-apla: https://github.com/AplaProject/go-apla/releases
.. _default Go workspace: https://golang.org/doc/code.html#Workspaces

1) Download and build the `latest release of go-apla`_ from GitHub:

.. code-block:: bash

    go get -v github.com/AplaProject/go-apla

2) Copy the go-apla binary to the ``/opt/apla/go-apla`` directory. If you use the `default Go workspace`_ then the binary is located in the ``$HOME/go/bin`` directory:

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


