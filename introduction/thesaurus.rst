################################################################################
Terms and Definitions
################################################################################
********************************************************************************
General Blockchain Technology Terms 
********************************************************************************
- *blockchain* - an information system that stores data, secures this data from falsification and loss, transfers and processes data inside the system while preserving data reliability; the protection of data is achieved by: (1) writing the data into a chain of cryptographically linked blocks, (2) decentralized storage of copies of the chains of blocks on nodes of a peer-to-peer network, and (3) synchronization of chains of blocks on all full nodes using a consensus algorithm; preserving data reliability when performing operations with data within the network is ensured by storing the algorithms of data transfer and processing (contracts) inside the blockchain; the chain of blocks itself is also referred to as the blockchain.
- *peer to peer network* - a computer network, consisting of equally privileged nodes (without a central server).
- *block* - a collection of transactions grouped by a validating node into a special data structure after their format and signatures are verified; a block contains a hash pointer as a link to the previous block, which is one of the measures that ensure the cryptographic security of the blockchain; a block is added to the blockchain after the consensus with other validating nodes on the network is achieved.  
- *hash* - a uniquely reproducible cryptographic representation of a file or any other set of digital data; it ensures the invariability of data – any modification of data changes its hash.
- *block validation* - verification of the correctness of a block's structure, its creation time, its compatibility with the previous block, the transaction signatures, and the correspondence of transactions to the data in the blockchain.
- *validating node* - a node on the network that has the right to create and validate blocks.
- *consensus* - an agreement between validating nodes on the procedure of adding new blocks to the blockchain or an algorithm of such agreement.
- *transaction* - a single operation of data transfer on a blockchain network, or a record of such transaction in the blockchain.
- *token* - a unit that represents a share of rights, fixed as a set of identifiable numerical records in the register that includes a mechanism for exchanging shares of rights between these records.
- *identification* - a cryptographic procedure for recognition of a user in the system.
- *unique identification* - a procedure that links a user with a unique person; it requires legal and organizational efforts to implement biometric or other procedures for association of usernames with real persons.
- *private key* - a string of characters kept in secret by its owner and used for accessing this person's Virtual Account on the network and signing transactions.
- *public key* - a string of characters which can be used to check the authenticity of a signature made with a private key; public keys can be uniquely derived from private keys, but not vice versa.
- *digital signature* - an attribute of a digital document or message, obtained as a result of cryptographic data processing; a digital signature is used to check the integrity (absence of modifications) and authenticity of a document (verify sender's identity).  
- *contract* - a program that performs operations with data stored in the blockchain; all contracts are stored in the blockchain. 
- *transaction fee* - payment to a validating node for execution of a transaction.
- *double spend* - an attack on a blockchain network with the purpose of using the same tokens for two different transactions; this attack is implemented by forming and maintaining a fork (alternate version) of the blockchain; such an attack can be executed only in the event the attacker controls 50% or more of the network's validating power.
- *encryption* - transformation of digital data in such a way that only the party that possesses the appropriate decryption key is able to read it.
- *private blockchain* - a blockchain network where all nodes and access rights to data are under the centralized control of a single organization (government, corporation, or private individual).
- *public blockchain* - a blockchain network which is not controlled by any organization, all decisions are made by reaching a consensus among the network's participants, and data is available to everyone for read access. 
- *delegated proof of stake (DPoS)* - a blockchain network consensus algorithm, where validating nodes are assigned by delegates (usually, token owners), who vote using their shares of rights.

********************************************************************************
Terms of the platform
********************************************************************************
- *testnet* - a version of the network that is used for testing software.
- *mainnet* - main version of the network.
- *Platform token* - tokens of the platform, which are used for payments for the use of network resources (fees).
- *Platform transaction* - commands that call contracts and pass parameters to them; the result of execution of a transaction by a node is the update of the platform's database.
- *fuel* - a conventional unit used to calculate the fee for execution of certain operations on the network; fuel to exchange rate is decided on by the voting of validating nodes.
- *account* - a storage record for tokens, which can be accessed with a pair of keys – a private and a public one. 
- *address* - a character-coded identifier of a user on the network, regarded as the name of this user's virtual account.
- *associated virtual account* - a virtual account from which the payment for execution of a contract is debited; an association of a contract with a virtual account is established upon contract creation and can be changed at any time; by default (before an association is established) payment is debited from the virtual account of the user who executed the contract.
- *Molis* - a software client used to connect to the network; Molis enables users to work with their virtual accounts, build ecosystems, and create applications in an integrated development environment (creating and editing tables, interface pages, and contracts).
- *web-Molis* - a fully-functional software client that works as a web-application. 
- *platform ecosystem* - a relatively closed programming environment, which includes large numbers of applications and users who create and/or use these applications; ecosystem members can initiate the emission of the ecosystem's own token, use a system of smart contracts to establish the rules of interaction between its members, and set members' permissions to access the ecosystem's elements.
- *ecosystem parameters* - a set of configurable ecosystem attributes (name, description, logo, name of the ecosystem’s token and its emission parameters, etc.); these attributes are stored and can be edited in a dedicated configuration table. 
- *ecosystem members* - users who have access to functions and applications of a particular ecosystem. 
- *dedicated ecosystem* - an ecosystem that has all the functions of a standard ecosystem, but works outside the blockchain (no data is saved in the  blockchain); in dedicated ecosystems contracts are able to access any web resources over HTTP/HTTPS protocols, and rights to read data can be configured.
- *delegated Proof of Value of Ecosystem (DPoV(E))* - consensus algorithm, where validating nodes are assigned by the voting of ecosystems whose significance to the platform is confirmed (Valued Ecosystems), since it is in their best interest to maintain the smooth operation of the network; the approval of ecosystems that satisfy a number of formal indicators (number of transactions, number of members) to become Valued Ecosystems is implemented by the voting of token owners (in order to avoid fake ecosystems with bot-generated activities from taking part in the approval of Validating Nodes). 
- *Simvolio* - a script language for building contracts; Simvolio contains functions for processing data received from interface pages, and for performing operations with values in database tables; contracts can be created and edited in the editor of the Molis software client.
- *Protypo* - a template language that includes functions required for obtaining values from database tables, and conditional statements/operators for building interface pages and forwarding user input data to contracts. 
- *integrated development environment* - a set of software tools used for creating applications; the Molis software client's integrated development environment includes a contract editor, pages editor, tools for work with database tables, language resource editor, and application export and import functions; the integrated development environment will soon be complemented with visual editors based on semantic tools.
- *interface designer* - a tool in the Molis software client used for creating interfaces of application pages by arranging basic application elements (HTML containers, form fields, buttons, etc.) directly on the screen.
- *visual interface editor * - a tool in the Molis software client used for creating interfaces of application pages, which includes an interface designer and a generator of page code in Protypo language.
- *visual contract editor* - a tool in the Molis software client used for creating contracts using a visual interface.
- *language resources* - a module of the Molis software client used for localization of application interfaces; it associates a label on a page in an application with a text value in a selected language.
- *export of applications* - saving the source code of an application (any set of its tables, pages, and contracts) as a file.
- *import of applications* - uploading an application (all tables, pages, and contracts included in an exported file) into an ecosystem.
- *smart law* - a record in the blockchain that contains regulatory information, which is used for controlling the operation of contracts and management of access rights to registers; smart laws are specialized smart contracts.
- *legal system* - a set of regulations established in smart laws; a legal system regulates the relations between the platform users, defines procedures for changing protocol parameters and includes mechanisms that provide solutions to various challenges.
- *application* - a functionally complete software product created in the Molis software client's integrated development environment; an application consists of database tables, contracts, and interface pages. 
- *application interface page* - a program code, written using the Protypo template language, that forms an interface on the screen.
- *interface block* - a program code, written using the Protypo template language, that can be included in application interface pages.
- *contract association* - linking a contract with a Virtual Account, from which the fee for  performing contract operations will be debited. 
- *access rights* - conditions for obtaining access to creating and editing tables, contracts and interface pages; access rights to tables can be specifically set for adding rows and columns, and for editing values in columns; 
- *full node* - a node on the platform network that stores the full up-to-date version of the blockchain.
- *partial node* - a node on the platform network that stores only the blocks with data related to one ecosystem.  
- *concurrent transactions processing* - a method for increasing the processing speed of transactions by simultaneously processing data from different ecosystems.
