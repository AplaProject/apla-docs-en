.. -- Conditionals Genesis / Apla -------------------------------------------------

.. backend GitHub repo name and link
.. |backend_gh| replace:: `go-genesis`_
.. _go-genesis: https://github.com/GenesisKernel/go-genesis 
.. .. |backend_gh| replace:: `go-apla`_
.. .. _go-apla: https://github.com/AplaProject/go-apla


.. frontend GitHub repo name and link
.. |frontend_gh| replace:: `genesis-front`_
.. _genesis-front: https://github.com/GenesisKernel/genesis-front   
.. .. |frontend_gh| replace:: `apla-front`_
.. .. _apla-front: https://github.com/AplaProject/apla-front 



About |platform| blockchain
###########################

This section explains how |platform| works with blockchain. 


Tip of the iceberg
==================

If you are interested in developing |platform| apps, using them, or managing ecosystems, then you probably don't need to know anything about the |platform| blockchain at all.

In |platform|, the blockchain and the blockchain network are hidden from ecosystem members, administrators, and even app developers. Instead, |platform| provides interfaces for all these groups of users. These interfaces provide access to the top layer of a blockchain: its tamper-proof distributed *global state*. 


App developers
--------------

In technical terms, the *global state* is a set of data. |platform|'s implementation of its global state is a database. From the app developer standpoint, applications interact with this database by querying, inserting, and updating the database tables.

|platform| applications are a collection of contracts and pages that interact with tables. 

Under the hood, contracts are executed by writing transactions to the blockchain. These transactions invoke contract code, which is executed by the blockchain network nodes, which leads to changes in the global state, the database. For an app developer, a contract is a function. When it is executed, data is written to the database. Pages are like scripts. Page code is a set of :doc:`Protypo </topics/templates2>` functions. Some of these functions display page elements, other fetch data from the database. No knowledge of transactions, block generation or consensus algorithms is required from an application developer to work with |platform| blockchain.


Ecosystem members
-----------------

In |platform|, apps written by app developers work inside autonomous software environments called *ecosystems*. An ecosystem typically serves a certain purpose and combines many apps created to support different aspects of this purpose.

To get access to the apps of an ecosystem, a user must become an *ecosystem member*. One user can be a member of many ecosystems.

Ecosystem members can view and modify the database from application pages, like they would do in a common web application: by filling forms, pressing buttons, and navigating pages.


Ecosystem and platform apps
---------------------------

Apps can be divided by scope into *ecosystem apps* and *platform apps*.

*Ecosystem apps* implement some certain functionality or business process specific to an ecosystem. An ecosystem app is available only in its ecosystem.

*Platform apps* are available in all ecosystems. Any app can be developed as a platform app. |platform| developers provide platform apps that support core functionality for ecosystem governance, such as apps for votings, notifications, and ecosystem member roles management.

.. todo::
    
    Move this to ecosystem intro


Under the hood
==============

The layers
----------

Under the top layer that is visible to ecosystem members and app developers, lies the "engine" of |platform|, its node network and blockchain protocols.

You can think of |platform| as having several layers: 

    - User interfaces layer

        Ecosystem members interact with applications via pages and page elements.

    - Apps layer

        App developers interact with global state (database tables) via contract and page code.

    - Global state layer

        Global state (database) is updated and synchronized based on the operations written to the distributed ledger of operations (blockchain).

    - Blockchain layer

        Distributed ledger of operations is updated with new blocks. New blocks hold the operations (transactions) that must be performed on the global state.

    - Node network layer

        Network nodes implement |platform| blockchain protocol. They distribute transactions in the node network, validate these transactions and generate new blocks. Blocks are, in turn, distributed and validated by network nodes. The distributed ledger is kept synchronized for all nodes in a network. If conflicts occur, nodes agree upon which block chains are considered valid and rollback the invalid block chains.

    - Transactions layer

        Transactions that are basis for block generation and blockchain protocol itself are a result of actions performed at the top layer. As explained in :ref:`implementation`, transactions are automatically generated by |platform| frontend, Molis client. 

        When a user or a developer makes an action such as clicking a button on a page or executing a contract from the code editor, Molis converts this action into a transaction and sends it to the network node that it is connected to. 


Thus, the top layer is connected to the bottom layer and the transaction flow goes in the opposite direction: 

    - A user action in the user interface creates a transaction.

    - The transaction gets included in a block.

    - The block is included in the blockchain.

    - Changes in the blockchain cause changes in the global state, the action is applied to the database.

    - The database changes are displayed in the app.


.. _implementation:

The implementation
------------------

.. todo::

    When detailed docs about implementation (daemons) are ready, link them here and to the blockchain chapter below. 


Two main components of |platform| are its backend, |backend_gh|, and Molis client, |frontend_gh|.


Molis client: 

    - Provides a user interface for |platform|.

    - Provides an IDE for app development.

    - Stores private keys of user accounts and performs authorization.

    - Requests app page data from the database, and displays app pages to users.

    - Sends transactions to the backend via :doc:`REST API</reference/api2>`. 

        Transactions are created automatically for user actions that require a transaction. For example, when an app developer executes a contract from the IDE, Molis converts this action into a transaction.


The backend: 

    - Keeps the global state (the database) of the node.
    - Implements all |platform| blockchain protocols.
    - Executes contract code in a :doc:`virtual machine </topics/vm>`.
    - Executes page code in a :doc:`template engine </topics/templates2>`. The result is page data that can be used by Molis client.
    - Implements :doc:`REST API</reference/api2>`.

.. todo::

    Make the templates link refer to a more specific heading: Interface Template Engine.