################################################################################
General information
################################################################################

.. contents::
  :local:
  :depth: 3
  
********************************************************************************
Distinctive features
********************************************************************************

The blockchain platform is developed for building digital ecosystems. The platform includes an integrated application development environment with a multi-level system of access rights to data, interfaces and smart contracts. The platform is built based on software developed by EGAAS Corporation (EGAAS S.A.).

In terms of its structure and functioning, Genesis is fundamentally different from most of the existing blockchain platforms: 

* The development and use of blockchain applications is carried out in autonomous software environments , called *ecosystems*. Every ecosystem has its own membership rules, initially established by the ecosystem founder. 
* Activities in ecosystems are based on creating *registers* and recording/modifying the data involved using *smart contracts*, whereas in most other blockchain platforms activities are based on exchanging transactions or messages between accounts. 
* Management of access permissions to registers, and relation management between ecosystem members are regulated by a set of rules called *smart laws*.

********************************************************************************
Genesis Blockchain Platform
********************************************************************************
Network
==========================
The Genesis blockchain platform is built based on a peer-to-peer network. Full nodes of the network store the up-to-date version of the blockchain and the database, in which the current state of the platform is recorded. The network users receive data by requesting it from databases of full nodes using the software client (or REST AP commands). New data is sent to the network in the form of transactions signed by users. Such transactions are in essence commands for modification of information in the database. Transactions are aggregated in blocks, which are then added to the blockchain on the network nodes. After a new block is added to the blockchain, each full node processes the transactions in this block, thus making changes to data in its database accordingly.

Validating Nodes
==========================
The network's full nodes that have the right to form blocks, are called Validating Nodes. The number of Validating Nodes is limited and is defined with the count_of_nodes parameter in the platform's configuration settings. 

The list of Validating Nodes is stored in the full_nodes parameter in the following format: 

*	[[“host1:port",”-1222","nodepub1"], ["host2:ip","-1222", "nodepub2"]], where 
*	host1:port – is the address of the host, to which the transactions and new blocks are sent; also, the whole chain of blocks (starting from block #1) can be requested and received from this address
*	-1222 – the node's account to which the fee for transaction processing is sent; if this account does not exist, the fee will not be charged
*	nodepub1 – the node's public key used for verification of signatures of blocks, created by this node

Transactions
==========================
A transaction is formed by the software client (or by the contract REST API command) and includes data for execution of a special program controller – contract ("smart contract"), called by a user. Each transaction has the following format: 

* Type - ID of the executed contract,                                   
* Data - parameters passed to the contract,                           
* KeyID - ID of the user who sent the transaction,          
* PublicKey - user's public key (optional),              
* BinSignatures - transaction signature,                         
* Time - transaction timestamp,                                
* EcosystemD - ID of the ecosystem, where the transaction was initiated,          
* ТokenEcosystem - ID of the ecosystem, the tokens of which should be used for the transaction payments, 
* MaxSum - maximum transaction fee,
* PayOver - additional payment for priority processing in the transaction queue.
 
A transaction is signed by the private key of an account holder. Both the key and the signing function can be stored in a browser, in the software client, on a SIM card, or on a specialized physical device. In the current implementation, private keys are kept in the Molis software client encrypted by the AES algorithm. Transactions are signed using the ECDSA algorithm.

Network Protocol
==========================
A transaction is sent by a user to one of the validating nodes, where it undergoes a basic verification to ensure the correctness of its format and then is added to the transaction queue. This transaction is also sent to other Validating Nodes on the network, where it's also added to the transactions queue. 

The Node, at a particular moment of time that has the right to generate a new block (according to the full_nodes parameter), retrieves transactions from the queue and sends them to the block generator. Simultaneously with the formation of a new block, the processing of transactions which are added to this block is carried out: each transaction is sent to the virtual machine that executes a corresponding contract with parameters, passed in the transaction, resulting in modification of the information in the database.
 
A new block is checked for errors, and if it is recognized as valid, it is sent to other Validating Nodes on the network. 

Validating Nodes add this newly received block to the blocks queue. After having been validated, a new block is added to the blockchain, and the transactions in this block are processed, thus updating the database.

Block and Transaction Verification
==========================
The verification of a new block, carried out by a Validating Node after it has created a new block, and the verification of such block on all other Validating Nodes after they receive this block, includes the following checks:

*	The first byte should be 0; if not, the received data is not considered a block
*	Received block's generation timestamp should be before the current time
*	The block's generation timestamp should correspond to the time interval when the Validating Node had the right to sign a new block
*	The new block's number should be greater than that of the last block in the existing chain
*	The total fee limit for transactions in the block should not be exceeded
*	The block should be correctly signed with the key of the Node that created it; the following data should be signed: BlockID, Hash of the previous block, Time, Position in full_nodes, MrklRoot from all transactions in the block
*	Each transaction in the block is checked for correctness in the following ways: 
  
  *	Each transaction's hash should be unique;
  *	The limit of transaction signed with one key should not be exceeded (max_block_user_tx);
  *	The transaction size should not be exceeded (max_tx_size);  
  *	The time when the transaction was sent should not be greater than the time of the block formation and not less than the block formation time minus 86400 seconds;
  *	Transactions should be correctly signed;
  *	The tokens which are assigned to be used for payment of transaction fees should exist in the sys_currencies list;
  *	The user who executed the contract should have a sufficient number of tokens in their account to pay for resources required for execution of the transaction.

Platform's Database
==========================
The platform's unified database, copies of which are stored and maintained up-to-date on every full node of the network, is used for storing large volumes of data (registers) and quick retrieval of data by contracts and interfaces. In the formation of a new block and its addition to the blockchain, all full nodes of the platform carry out a simultaneous update of database tables. Thus, the database stores the current (up-to-date) state of the blockchain, which ensures the equivalence of data on all full nodes and unambiguousness of contract execution on any Validating Node. When a new full node is added to the network, the up-to-date status of its database is reached by way of subsequent execution of all transactions recorded in the blocks of the blockchain. 

Currently, the Genesis platform uses PostgreSQL as its database management system. 

********************************************************************************
Platform's Ecosystems
********************************************************************************
The data space of the Genesis platform is divided into many relatively independent clusters – *ecosystems*, in which the activities of the network's users are implemented. An Genesis ecosystem is an autonomous software environment that consists of a certain number of applications and users, who create these applications and work with them. Any holder of an Genesis account can create a new ecosystem.

The software basis of an ecosystem is a collection of applications, which are systems of interfaces, contracts, and database tables. The specific ecosystem to which application elements belong is indicated by prefixes in their name (for example, @1name), where the ecosystem's ID is indicated after the “@” sign. When addressing application elements within the current ecosystem, the prefix can be omitted. 

The Molis software client provides access to database management tools, contracts editor, interface editor, and other functions required for the creation of applications in an ecosystem, without resorting to any additional software modules. 

A person can become a user of the Genesis platform only after receiving a private key for accessing one of the ecosystems (by default, ecosystem #1). A user can be a member of any number of ecosystems. Switching between ecosystems is carried out using a specialized menu of the software client.

Integrated Development Environment
==========================
The Molis software client includes a full-scale integrated development environment (IDE) for creation of blockchain applications. Working with this IDE does not require the software developers to have profound knowledge of blockchain technology. The IDE is comprised of:

-	Ecosystem parameters table,
-	Contracts editor, 
-	Database tables administration tools,
-	Interface editor and a visual interface designer,
-	Language resource editor,
-	Application import / export service.

Applications on Genesis 
==========================
An application on the Genesis platform is a system of tables, contracts and interfaces with configured access rights. Such applications perform useful functions or implement various services. 

Each ecosystems creates its own set of tables for development of applications. This, however, does not exclude the possibility of accessing tables from other ecosystems by specifying those ecosystems' prefixes in table names. Tables are not in any way bound (nor belong) to specific contracts, and can be used by all applications. The permissions for entering data into tables are set by way of configuring the access rights. Specialized contracts – smart laws – can be used for rights management. 

It should be noted, that the design and creation of applications on Genesis does not require the software developers to know the structure of the network and its protocols, nor to understand the algorithm of blockchain formation and synchronization of databases on full nodes. Work in the Molis software client, including the creation of application elements, reading data from tables, execution of contracts and displaying results on the screen, looks and feels like operations with modules of a software environment on a local computer.

Ecosystem's Tables
==========================
An unlimited number of tables can be created for each ecosystem on the platform's database. As mentioned earlier, tables belonging to a specific ecosystem can be identified by a prefix that contains the ecosystem ID, which is not displayed in the software client while working within that specific ecosystem. Making records in tables of other ecosystem's tables is possible in cases where the access rights are configured to allow such actions.

Tools for Tables Administration
--------------------------
Tools for administration of an ecosystem's tables are available from the Tables menu of the administrative tools in the Molis software client. The following functions are implemented:

•	Viewing the list of tables and their contents, 
•	Creation of new tables,
•	Adding new table columns and specifying the data type in columns: Text, Date/Time, Varchar, Character, JSON, Number, Money, Double, Binary, 
•	Management of permissions for entering data and changing the table structure.

Operations with Data in Tables
--------------------------
To organize the work with the database, the Simvolio contract language and the Protypo template language both have the DBFind function, which provides for retrieving values and data arrays from tables. The contract language has a function for adding rows to tables, DBInsert, and a function for changing values in existing entries, DBUpdate (when a value is changed, only the data in the database table is rewritten, whereas the blockchain is appended with a new transaction while preserving all previous transactions). Data in tables can be modified but not deleted.

In order to minimize the time of contracts execution, the DBFind functions cannot address more than one table at the same time, thus the requests with JOIN are not supported. That is why it is not advisable to normalize the application tables, but rather include all available information to the rows, thus duplicating data available in other tables. This, however, is not just a coercive measure, but a necessary requirement for blockchain applications, where what is saved (signed by a private key) should be a full, complete, up-to-date for a specific moment in time set of data (document), which cannot be modified due to the change of values in other tables (which is inevitable in relational databases).

Ecosystem Parameters
==========================
The ecosystem parameters are available for viewing and editing from the Ecosystem parameters section in the administrative tools of the Molis software client. Ecosystem parameters can be divided into the following groups:

•	General parameters: name of the ecosystem (ecosystem_name), its description (ecosystem_description), account of its founder (founder_account), and other information,
•	Access parameters, which define exclusive rights to access application elements (changing_tables, changing_contracts, changing_page, changing_menu, changing_signature, changing_language)
•	Technical parameters: for example, user stylesheets (stylesheet),
•	User parameters of the ecosystem, where constants or lists (separated by commas), required for the work of applications are stored.

Rights to edit can be specified for every ecosystem's parameter.

In order to retrieve values of certain ecosystem parameters, both the contracts language Simvolio and the template language Protypo have the EcosysParam function, where an ecosystem parameter name can be specified as an argument. To retrieve an element from a list (entered as an ecosystem parameter and separated by commas), you should specify you desired element's counting number as a second argument for the function. 

Parameters of the Platform Ecosystem
--------------------------
All parameters of the Genesis blockchain platform are stored in the parameters table of the platform configuration ecosystem. These are the following parameters:

-	Time period for creation of a block by a Validating Node,
-	Source codes of pages, contracts, tables, and menus of new ecosystems,
-	List of validating nodes,
-	Maximum transaction and block sizes, and the maximum number of transactions in one block,
-	Maximum number of transactions sent by the same account in one block,
-	Maximum amount of Fuel spent on one transaction and one block,
-Fuel to APL exchange rate, and other parameters.

Managing the parameters of the platform configuration ecosystem on the program level is the same as managing the parameters of any other ecosystem. Unlike in other ecosystems, where all rights to manage ecosystem parameters belong to the ecosystem founder, changing the parameters of the platform configuration ecosystem can only be performed using the UpdSysContract contract, the management of which is defined in the platform's Legal System. Contracts (smart laws) of the Legal System are created before the network is launched and implement the rights and standards, stipulated in the “Platform's Legal System” section of the White Paper.  

********************************************************************************
Access Rights Control Mechanism 
********************************************************************************
Genesis has a multi-level access rights management system. Access rights can be configured to create and change any element of an application: contracts, database tables, interface pages, and ecosystem parameters. Permissions to change access rights can be configured as well.

By default, all rights in an Genesis ecosystem are managed by its founder (this is defined in the MainCondition contract, which every ecosystem has by default). However, after specialized smart laws are created, access rights control can be transferred to all ecosystem members or a group of such members.

Controlled Operations
==========================
Permissions can be defined in the Permissions field of contracts, tables and interface (pages, menus, and page blocks) editors, available from the Molis administrative tools section. Permissions for the following operations can be configured:

1.	Table column permission – permission to change values in the table column,
2.	Table Insert permission – permission to add a new row to the table,
3.	Table New Column permission – permission to add a new column,
4.	Conditions for changing of Table permissions – permission to change rights, listed in items 1-3,
5.	Conditions for change smart contract – permission to edit the smart contract,
6.	Conditions for change page – permission to edit the interface page,
7.	Conditions for change menu – permission to edit the menu,
8.	Conditions for change of ecosystem parameters – permission to change a certain parameter in the ecosystem configuration table.

Ways to Manage Permissions
==========================
Rules, that define the access rights, should be entered in the *Permissions* fields as arbitrary expressions in Simvolio language. Access will be granted in the event that at the moment of request the expression was true. If the *Permissions* field is left blank, it is automatically set to *false*, and the execution of related actions is blocked.

The easiest way to define permissions is to enter a logical (boolean) expression in the *Permissions* field. For example, $member == 2263109859890200332, where the ID of a certain ecosystem member is given. 

The most versatile and recommended method for defining permissions is the use of the *ContractConditions* function, to which a contract name can be passed as a parameter. This contract should include the conditions, in which formulation of the table values (for example, user roles tables) and ecosystem parameters can be used. 

Another method of permissions management is the use of the ContractAccess function. The list of contracts that are eligible to implement a corresponding action can be passed to the ContractAccess function as parameters. For example, if we take the table that lists the accounts in the ecosystem's tokens, and put ``ContractAccess(“TokenTransfer”)`` function in the *Permissions* field of the amount column, then the operation of changing the values in the amount column will be allowed only to the *TokenTransfer* contract (all contracts that perform token transfer operations between accounts, will be able to perform such operations only by calling the *TokenTransfer* contract). Conditions for accessing the contracts themselves can be managed in the conditions section. They can be rather complex and can include many other contracts.

Exclusive Rights
==========================
To resolve conflict situations or those critical for the operation of an ecosystem, the Ecosystem parameters table has a number of special parameters (*changing_smart_contracts, changing_tables, changing_pages*), where the conditions for obtaining exclusive rights to access any smart contracts, tables and pages are defined. These rights are set using special smart contracts, for example, executing a voting of ecosystem members or requesting the availability of a number of signatures of different user roles.

********************************************************************************
Virtual Dedicated Ecosystems
********************************************************************************

Genesis allows for creation of Virtual Dedicated Ecosystems (VDE), which have the full set of functions of standard ecosystems, but work outside the blockchain. In VDE full-scale applications can be created using the contract and template languages, database tables and other software client functions. Contracts from blockchain ecosystems can be called using API.

Requests to Web-Resources
==========================
The main difference between VDE and standard ecosystems is the possibility to make requests from its contracts to any web-resources via HTTP/HTTPS using the HTTPRequest function. Arguments passed to this function should be: URL, request method (GET or POST), header, and request parameters.

Rights to Read Data 
==========================
Since data in VDE are not saved to the blockchain (which, however, is available for reading), they have an option to configure rights to read tables. Read rights can be set for separate columns, and for any rows using a special contract.

Using VDE
==========================
VDE can be used for the creation of registration forms and sending verification information to users’ emails or phones, storing data out of public access, and writing and testing the work of applications with their further export and import to blockchain ecosystems. Also, in VDE you can schedule contract execution, which allows for the creation of oracles, which are used for receiving data from the web and sending it to the blockchain.

Creating a VDE
==========================
VDE can be created on any full node on the network. Node Administrator defines the list of ecosystems that are allowed to use the functions of dedicated ecosystems, and assigns a user who will have the rights of the ecosystem founder and will be able to: install applications, accept new members to the ecosystem, and configure access rights to the ecosystem's resources.

