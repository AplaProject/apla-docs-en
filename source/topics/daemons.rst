.. -- Conditionals Genesis / Apla -------------------------------------------------

.. backend binary name and GitHub link
.. |backend| replace:: `go-genesis`_
.. _go-genesis: https://github.com/GenesisKernel/go-genesis	
.. .. |backend| replace:: `go-apla`_
.. .. _go-apla: https://github.com/AplaProject/go-apla



Backend daemons
###############

This document describes how |platform| nodes work from the technical standpoint.


Backend structure
=================


|platform| backend, |backend| operates at every network node. 

The backend supports the blockchain network: 

- It receives transactions from clients and distributes these transactions to other nodes. 
- It receives new blocks from other nodes, generates new blocks, synchronizes blockchain between nodes.


Daemons
-------

Backend daemons perform individual functions of the backend. A *full node* (a node that can generate new blocks) runs the following backend daemons:

- *BlockGenerator*

	Generates new blocks

- *BlockCollections*

	Downloads new blocks from other nodes

- *Confirmation*

	Confirms that blocks present at the node are also present at the majority of other nodes.

- *Disseminator*

	Distributes transactions and blocks to other full nodes.


- *QueueParseBlock*

 	Downloads and parses new blocks from the :ref:`block queue`.

- *QueueParseTx* 

	Parses transactions from the :ref:`transaction queue`.
