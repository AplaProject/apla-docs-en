Blockchain in general terms
###########################

This section explains, in general terms, how blockchain technology works. 

Blockchain
==========

A blockchain is a distributed *ledger* of *operations* performed on a *global state* by *members* of a blockchain network. 


Nodes
=====

Members of a blockchain network are called *nodes*. Each member owns a copy of the global state and a copy of the ledger. There is no central server or authority that holds master copies. Instead, nodes use the  ledger to agree upon what the global state must be.


Global state
============

The *global state* is a set of data. Most common implementation of a global state is a database. 

The global state is mutable. It can be changed with operations that members write to the ledger. For example, a database entry with the asset owner name can be changed, so that a new person owns the asset.


Ledger
======

The *ledger* (or *blockchain*) is a list of operations that were performed on a global state. 

The ledger is immutable. The blockchain technology makes it so that operations written to the ledger cannot be modified or removed at a later date. For example, there is no way to back-out the operation that states that the asset owner name was changed at some point.


Transactions and blocks
=======================

The operations that members write to the ledger are called *transactions*.

Every member can write transactions to the ledger. A selected member (*validating node*) checks that this transaction is valid and can be written to the ledger. If it is so, this member generates a new *block* in the blockchain and signs it. A block usually holds several transactions.

This block is passed to other network nodes that validate the block. If the block is valid, they include it in their ledger and perform all the transactions from this block on their copy of the global state. Once all members have validated the block, the network have reached *consensus*. This block is considered valid by all the nodes of a blockchain network, and the global state is the same for all the nodes.

.. note::

    |platform| uses a proof-of-authority consensus mechanism. Only the selected nodes called *validating nodes* can generate new blocks. 

    Other blockchain-based projects use different consensus mechanisms. For example, bitcoin cryptocurrency uses prof-of-work mechanism, where a node that solved a complex equation is awarded a right to create a new block. 


Example
=======

For example, three friends own a collection of expensive automobiles, which are stored in a hangar. Each friend has a list of automobiles in the hangar and a log of operations for the hangar. 

In blockchain terms, three friends are network nodes, the hangar is a global state, the automobiles in the hangar are data stored in the global state, the log of operations is a ledger, and entries in this log are transactions.

Now, for example, one of the friends buys a new automobile and decides to add it to the hangar. He sends a message to his friends. The message says that he has added a Buick 1961 to the hangar. One of the friends (appointed for this task) validates this claim, for example, by contacting the hangar personnel and checking if the new automobile is really there. If this message is valid, he adds a new entry to his copy of the operations log and adjusts his list of automobiles. He also tells all other friends what he has changed in his operations log. Other friends check that the change was correct and adjust their list of automobiles in the hangar. When all three friends have accepted the change in their ledger and adjusted their list of automobiles, the change was finalized.

In blockchain terms, one of the network nodes generated a transaction and sent it to other nodes. One of the validating nodes then validated this transaction and added a new block to the ledger. This block was sent to other network nodes who validated it, added it to the blockchain, and changed their global state accordingly. Once all nodes have made the changes to the ledger and to the global state, it is said that the network have reached the consensus.
