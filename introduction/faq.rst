################################################################################
FAQ
################################################################################
1.	In a few words, how would you describe the platform?
 -	blockchain platform, designed to build digital ecosystems on the basis of an integrated application development environment with a multi-level system for management of access rights to data, interfaces and smart contracts.
2.	Does platform work on the Bitcoin, Ethereum, or some other blockchain?
 -	platform is build on the basis of an original blockchain.
3.	What are the main differences between platform and other public blockchain platforms with built-in mechanisms for execution of smart contracts, such as Ethereum, Qtum, and those still being designed, including Tezos and EOS?
 -	platform features unique functions that cannot be found in the aforementioned blockchain platforms: 
 - Integrated application development environment implemented in a single client software;
 - Specialized template language for design of interfaces, harmonized with the contract-building language;
 -	Multi-level system for management of access rights to data, contracts, and interfaces where rights can be granted to persons, member roles, and contracts;
 -	Ecosystems – autonomous software environments for creation of blockchain applications and user interaction with them;
 - 	Legal system – a set of regulations, codified in smart laws (specialized smart contracts), which regulate relations between the platform users, define the protocol parameters changing procedures, and are used to solve problems.
4.	Does platform has its own cryptocurrency? 
 -	The platform uses special tokens. Ecosystems can issue tokens of their own.
5.	What is a validating node?
 -	Validating node is a network node that is authorized to check transactions and create blocks.
6.	Which network nodes are eligible to become validating nodes?
 -	No, there can be only fixed number of validating nodes in the network.
7.	Who can maintain a validating node?
 -	Any network node with sufficient processing power and fail tolerance can become a validating one. A node can become a validating node based on voting of ecosystems, but only ecosystems that are proved by investors as genuinely functioning (by platform tokens owners) can participate in such voting. Therefore, platform network implements a new consensus algorithm – Delegated Proof of Value of Ecosystems (DPoVE). With this algorithm it is most likely that the validating nodes will be run by major ecosystems, since it is in their best interest to maintain the network operation.
8.	What are platform ecosystems?
 -	Ecosystems are virtually autonomous software environments for creation of blockchain applications and user operations with them. 
9.	Who can create an ecosystem?
 -	Any user of the  platform can create a new ecosystem.
10.	How can a user become a member of an ecosystem?
 -	Registration in the platform network is made in any of its existing ecosystems; there can be different procedures for admission to membership, which are defined by the ecosystems' policies: from posting information about a new ecosystem in a specialized catalog to sending out public keys. 
11.	Can one user create more than one ecosystem?
 -	Yes, every user can create any number of ecosystems, and be a member of any number of ecosystems at the same time.
12.	What is a platform application?
 -	Application is an integral software product that implements a function or a service. Applications are comprised of database tables, contracts and interfaces.
13.	Which programming language is used for creation of applications?
 -	Contracts are written using the Simvolio language, which was developed by the platform team (see contract language description).  
 -	Interfaces are written using Protypo – an original interface template language (see template language description). 
14.	Which software is used for creating applications and user interaction with them?
 -	Applications are written and executed in Molis – the single software client; no other software is required. 
15.	Can platform contracts access data using third-party API interfaces?
 -	No, contracts can directly access only the data stored in the blockchain. Specialized oracles are used to work with external data sources.
16.	Can a contract saved in the blockchain be edited later?
o	Yes, contracts are editable. Rights to edit contracts are established by their creators, who can deny any changes or grant rights to make changes to contracts to specific persons or configure a complex set of conditions in a specialized smart law.
 -	The Molis software client provides access to all contract versions.
17.	What is a smart law?
 -	Smart law is a contract that is created to control and restrict the operation of regular contracts, and thus the activities of ecosystems' members. A set of mart laws can be regarded as an ecosystem's legal system.
18.	Can a contract call/execute another contract?
 -	Yes, contracts can call other contracts by way of directly addressing another contract and providing parameters to it, or by way of calling a contract by link (name)  (see contract language description).
19.	Is master contract required for work of applications?
 -	No, it's not. Contracts are autonomous program modules that execute some functions. Each contract is configured to receive specific data, properly check this data, and execute some action, which will be recorded as a transition in the database.
20.	Can applications be localized to different languages?
 -	Yes, the software client has a built-it mechanism for localization support, allowing for creation of interfaces on any languages. 
21. Can interfaces be created without using the Protypo template language?
 - Yes, platorm API can be used for that.
22. Are interface pages stored in the blockchain?
 -	Yes, pages and contracts are stored in the blockchain, which protects them from falsification.
23. What types of databanks can be used for operation of contracts?
 -	The Molis software client includes instruments for creation of database tables (PostgreSQL is used at the moment, but we may change that later), and the Simvolio contracts language has all functions required for reading and writing of data; Protypo template language includes functions for reading data from tables.
24. How is the access to data in tables regulated?
 -	Rights to add a column, a row, or to edit data in a column can be provided to ecosystem members, roles, or specific contracts (with the prohibition to contracts, other than those created to carry out specified operations).
25. Can applications inside an ecosystem exchange data with applications from another ecosystem?
 - 	Yes, data exchange can be organized through global (available for all ecosystems) tables.
26. Should all applications in a new ecosystem be written from scratch?
 - No, each new ecosystem as a number of applications available out-of-the-box: a mechanism for management of members and roles in an ecosystem, an application for configuration and emission of tokens, a voting system, a social news system with incentives for activity, and a messenger for ecosystem members. These applications can be edited and configured to meet the specific requirements of any ecosystem.
27. Is there any payment for operation of applications?
 - 	Yes, the use of resources of validating nodes should be paid in platform tokens.
28. Who pays for operation of applications?
 - 	An account (binding account), which the tokens for payment for resources are debited from, is set by the contract creator on its activation; there is an algorithm to change the wallet. It can be defined using ecosystem's smart laws whether or not the ecosystem members will pay for work with the application, and if yes, than what way of payment it will be (contributions or otherwise). 
29.  How are applications within ecosystems protected from exploit of their vulnerabilities?
 -	  The platform team understands that there is no way to completely avoid mistakes in the program code of applications, especially given that applications can be written by any user. That's why we decided to create a mechanism that eliminates the consequences of exploit of vulnerabilities. The platform has a legal system (a set of smart laws), that allow for stopping the operation of an attacking application and make a number of transactions restoring the status quo. The rights to execute such contracts and voting procedures to grant these rights are defined in the smart laws of the platform's legal system.   
30.  Which new functions are planned to be implemented in platform in the future?
 -	 Visual interface designer,
 -	 Visual smart contract designer,
 -	 Support of hybrid (SQL and NoSQL) databases,
 -	 Parallel multi-threaded processing of transactions coming from different ecosystems.
 -	 Execution of resource-intensive calculations on the client side.
 -	 Hosting for ecosystems and a computing power exchange.
 -	 Partial nodes that store only a part of blocks on the server.
 -	 Semantic reference (ontology) for unification of operations with data within the platform.
31.  Are there any proofs of platform's operability?
 -	 A number of proof of concept projects have been implemented on the platform during the last months: a polling and voting system for a political party (Netherlands), new businesses registration (UAE), trading financial instruments (Luxembourg), register of property (India), and a contracts management system (UAE).
32.  Does platform have any obvious drawbacks?
 -  The biggest drawback of platform, compared to, say, Ethereum, is that platform is just in the launch mode. But this drawback will transform into a big advantage over time.
33.  How do you see the future of Alpa?
 -	 Platform was designed based on the assumption that the full effect of the blockchain technology can be achieved only when all activities, operations, registers and contracts to one blockchain. Just as there can't be many co-existing Internets, there ultimately can't be many co-existing blockchain networks. We see platform as a unified platform, which in the future will run all operations of all governments in the world.
