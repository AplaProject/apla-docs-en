.. _JWT token: https://jwt.io

REST API v2
###########

All functions, available from the Molis software client, including authentication, receipt of data about ecosystems, error handling, operations with database tables, interface pages, and execution of contracts (network transactions) are available though REST API of the platform. Thus, by using REST API, developers can access any function of the platform without using the Molis software client.

Command calls are performed by addressing ``/api/v2/command/[param]``, where **command** is a command name, and **param** is an additional parameter (for example, the name of the resource to change or receive). Request parameters must be sent with ``Content-Type: x-www-form-urlencoded``. The server response will be sent in JSON format.

.. contents::
   :depth: 2
   :local:



Error handling
==============

Status 200 is returned in case of a successful request execution. In case of an error, apart from an error status, a JSON object is returned with the following fields:

* **error**
    
    Error identifier.

* **msg**
    
    Error text.

* **params**
    
    Array of additional parameters of the error, which can be put into the error message


.. code-block:: default
    :caption: Response example

    400 (Bad Request)
    Content-Type: application/json
    {
        "err": "E_INVALIDWALLET",
        "msg": "Wallet 1111-2222-3333 is not valid",
        "params": ["1111-2222-3333"]
    }


Error List
----------

.. todo::
    
    https://github.com/GenesisKernel/go-genesis/blob/master/packages/api/errors.go
    `E_DELETEDKEY`: `The key is deleted`,
    `E_PERMISSION`: `Permission denied`,
    `E_UNKNOWNSIGN`: `Unknown signature`,
    `E_REQUESTNOTFOUND`: `Request %s doesn't exist`,
    `E_UPDATING`: `Node is updating blockchain`,
    `E_STOPPING`: `Network is stopping`,

.. describe:: E_CONTRACT

    There is no %s contract.

.. describe:: E_DBNIL

    DB is nil.

.. describe:: E_ECOSYSTEM

    Ecosystem %d doesn't exist.

.. describe:: E_EMPTYPUBLIC

    Public key is undefined.

.. describe:: E_EMPTYSIGN

    Signature is undefined.

.. describe:: E_HASHWRONG

    Hash is incorrect.

.. describe:: E_HASHNOTFOUND

    Hash has not been found.

.. describe:: E_HEAVYPAGE

    This page is heavy.

.. describe:: E_INSTALLED

    Platform is already installed.

.. describe:: E_INVALIDWALLET

    Wallet %s is not valid,

.. describe:: E_NOTFOUND

    Content page or menu has not been found.

.. describe:: E_NOTINSTALLED

    Platform is not installed. 

    In this case, you need to run the installation with the *install* command.

    The **E_NOTINSTALLED** error should be returned by any command except for install, in case the system is not yet installed. 

.. describe:: E_PARAMNOTFOUND

    Parameter has not been found.

.. describe::  E_QUERY

    DB query is wrong.

.. describe:: E_RECOVERED

    API was recovered.

    Returned if there is a panic error.

    This error means that you have encountered a bug that needs to be found and fixed. 

.. describe:: E_REFRESHTOKEN

    Refresh token is not valid.

.. describe:: E_SERVER

    Server error. 

    Returned if there is an error in the golang library functions. The *msg* field contains the error text.

    The **E_SERVER** error may appear in response to any command. If it appears as a result of incorrect input parameters, it can be changed to relevant errors. In the other case, this error reports of invalid operation or incorrect system configuration, which requires a more detailed investigation. 

.. describe:: E_SIGNATURE

    Signature is incorrect.

.. describe:: E_STATELOGIN

    %s is not a membership of ecosystem %s.

.. describe:: E_TABLENOTFOUND

    Table %s has not been found.

.. describe:: E_TOKEN

    Token is not valid.

.. describe:: E_TOKENEXPIRED

    Token is expired by %s.

.. describe:: E_UNAUTHORIZED

    Unauthorized.

    The **E_UNAUTHORIZED** error can be returned for any command except for *install, getuid, login* in cases where login was not performed or the session has expired.

.. describe:: E_UNDEFINEVAL

    Value %s is undefined.

.. describe:: E_UNKNOWNUID

    Unknown uid.

.. describe:: E_VDE

    Virtual Dedicated Ecosystem %s doesn't exist.

.. describe:: E_VDECREATED

    Virtual Dedicated Ecosystem is already created.


Routes unavailable in VDE
=========================

Requests that are not available in VDE.

- txstatus
- txstatusMultiple
- appparam
- appparams
- history
- balance
- block
- maxblockid
- ecosystemparams
- ecosystemparam
- systemparams
- ecosystems


Authentication
==============

The `JWT token`_ is used for authentication. After receiving a JWT token, you must put it in the header of every request: ``Authorization: Bearer TOKEN_HERE``. 


getuid
------

**GET**/ Returns a unique value, which needs to be signed with your private key and sent back to server using the **login** command. 

A temporary JWT token is created, which needs to be passed to **Authorization** when calling **login**.


Request
"""""""

.. code-block:: default
    
    GET
    /api/v2/getuid
    


Response
""""""""

* *uid*

    Signature line.

* *token*

    Temporary token to pass in login. 

    The lifetime of a temporary token is 5 seconds.

In cases where authorization is not required, the following information is returned:

* *expire*

    Number of seconds before the time runs out.

* *ecosystem*

    Ecosystem ID.

* *key_id*

    Wallet ID.

* *address*

    Wallet address in the ``XXXX-XXXX-.....-XXXX`` format.


Response example
""""""""""""""""

.. code-block:: default
    
    200 (OK)
    Content-Type: application/json
    {
        "uid": "28726874268427424",
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........AU3yPRp64SLO4aJqhN-kMoU5HNYTDplQXbVu0Y"
    }
    


Errors
""""""

*E_SERVER*


login
-----

**POST**/ User authentication.

 The **getuid** command should be called in the first place in order to receive a unique value and sign it. A temporary JWT token, which was received along with getuid, should be passed in the header. 

 In case of success, the token that was received in the response should be included in all queries in the *Authorization* header.


Request
"""""""

.. code-block:: default

    POST
    /api/v2/login
    

* *[ecosystem]*

    Ecosystem ID. 

    If not specified, the command will work with the first ecosystem.

* *[expire]*

    Lifetime of the JWT token in seconds (36000 by default).

* *[pubkey]*

    Public hex key, if the blockchain already stores a key, then the wallet number should be passed with the *key_id* parameter.

* *[key_id]*

    Account id as a number or in the ``XXXX-...-XXXX`` format. 

    Use this in cases where the public key is already stored in the blockchain. Can't be used together with *pubkey*.

* *signature*

    A uid signature received though getuid hex.


Response
""""""""

* *token*

    JWT token.

* *refresh*

    JWT token to extend the session. Must be sent in the **refresh** command.

* *ecosystem*

    Ecosystem ID.

* *key_id*
    Account ID.

* *address*

    Account address in the ``XXXX-XXXX-.....-XXXX`` format.

* *notify_key*

    Key for notifications.

* *isnode*

    Whether this user is the owner of this node.

    Values: true, false.

* *isowner*

    Whether this user is the owner of this ecosystem.

    Values: true, false.

* *vde*

    Does this ecosystem have a virtual dedicated ecosystem.

    Values: true, false.


Response example
""""""""""""""""

.. code-block:: default
    
    200 (OK)
    Content-Type: application/json
    {
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........AU3yPRp64SLO4aJqhN-kMoU5HNYT8fNGODp0Y"
        "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........iOiI1Nzk3ODE3NjYwNDM2ODA5MzQ2Iiw"        
        "ecosystem":"1",
        "key_id":"12345",
        "address": "1234-....-3424"
    }      

Errors
""""""

*E_SERVER, E_UNKNOWNUID, E_SIGNATURE, E_STATELOGIN, E_EMPTYPUBLIC* 

    
refresh
-------

**POST**/ Issues new tokens and extends the user session. 

In case of successful completion you need to send the token, which was received in response, in the *Authorization* header of all queries.


Request
"""""""

.. code-block:: default

    POST
    /api/v2/refresh
    

* *[expire]*

    Lifetime of the JWT token in seconds (36000 by default).

* *token*

    Refresh token from the previous **login** or **refresh** calls.


Response
""""""""

* *token*

    JWT token.

* *refresh*

    JWT token for session extension. Must be sent to the **refresh** command.


Response example
""""""""""""""""

.. code-block:: default
    
    200 (OK)
    Content-Type: application/json
    {
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........AU3yPRp64SLO4aJqhN-kMoU5HNYT8fNGODplQXbVu0Y"
        "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........iOiI1Nzk3ODE3NjYwNDM2ODA5MzQ2Iiw"        
    }     
    

Errors
""""""

*E_SERVER, E_TOKEN, E_REFRESHTOKEN* 



Service Commands
================

    
version
-------

**GET**/ Returns the current server version.

 
Request
"""""""

.. code-block:: default

    GET
    /api/v2/version
    


Response example
""""""""""""""""

.. code-block:: default
    
    200 (OK)
    Content-Type: application/json
    "0.1.6"



Data Request Functions
======================


balance
-------

**GET**/ Requests the balance of an account in the current ecosystem. 


Request
"""""""

.. code-block:: default 
    
    GET
    /api/v2/balance/{key_id}
    

* *key_id*

    account id can be specified in any format - ``int64, uint64, XXXX-...-XXXX``. This wallet will be searched for in the ecosystem, which the user is currently logged in.   
    

Response
""""""""

* *amount*

    Account balance in minimum units.

* *money*
    
    Account balance in units.
    

Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "amount": "123450000000000000000",
        "money": "123.45"
    }      
    

Errors
""""""

*E_SERVER, E_INVALIDWALLET*



Working with ecosystems
=======================

ecosystemname
-------------

**GET**/ returns the name of an ecosystem by its identifer.

.. code-block:: default 

    GET
    /api/v2/ecosystemname?id=..
    

* *id*

    Ecosystem ID.


Response example
""""""""""""""""

.. code-block:: default 

    200 (OK)
    Content-Type: application/json
    {
        "ecosystem_name": "platform_ecosystem"
    }


Errors
""""""

*E_PARAMNOTFOUND*


ecosystems
----------

**GET**/ Returns a number of ecosystems.

.. code-block:: default  
    
    GET
    /api/v2/ecosystems/


Response
""""""""

* *number*

    The number of installed ecosystems.


Response example
""""""""""""""""

.. code-block:: default  
    
    200 (OK)
    Content-Type: application/json
    {
        "number": 100,
    }      



appparams
---------

**GET**/ Returns a list of application parameters in the current or specified ecosystem.
 

Request
"""""""
 
* *[appid]*

    Application identifier.

* *[ecosystem]*

    Ecosystem ID; if not specified, the current ecosystem's parameters will be returned.

* *[names]*

    List of received parameters.

    A list of parameter names separated by commas can be specified, for example: ``/api/v2/appparams/1?names=name,mypar``.


Response
""""""""
 
* *list*

    An array where each element contains the following parameters:
    
    * *name*–parameter name
    * *value*–parameter value
    * *conditions*–permissions to change a parameter
 

Response example
""""""""""""""""

.. code-block:: default
    
    200 (OK)
    Content-Type: application/json
    {
        "list": [{ 
            "name": "name",
            "value": "MyState",
            "conditions": "true",
        }, 
        { 
            "name": "mypar",
            "value": "My value",
            "conditions": "true",
        }, 
        ]
    }      


Errors
""""""

*E_ECOSYSTEM*


appparam/{appid}/{name}
-----------------------

 **GET**/ Returns information about the **{name}** parameter of the **{appid}** application in the current or specified ecosystem. 
 
* *appid*

    Application ID.

* *name*

    Name of the requested parameter.

* *[ecosystem]*

    Ecosystem ID (optional parameter). 

    By default, the current ecosystem will be returned.
 

Request
"""""""

.. code-block:: default
    
    GET
    /api/v2/{appid}/{appid}/{name}[?ecosystem=1]
    

Response
""""""""
     
* *id*

    Parameter identifier.

* *name*

    Parameter name.

* *value*

    Parameter value.

* *conditions*

    Conditions to change a parameter.


Response example
""""""""""""""""

.. code-block:: default
    
    200 (OK)
    Content-Type: application/json
    {
        "id": "10",
        "name": "par",
        "value": "My value",
        "conditions": "true"
    }      


Errors
""""""

*E_ECOSYSTEM, E_PARAMNOTFOUND*


ecosystemparams
---------------

**GET**/ Returns a list of ecosystem parameters. 

Request
"""""""

.. code-block:: default
    
    GET
    /api/v2/ecosystemparams/[?ecosystem=...&names=...]
    

* *[ecosystem]*

    Ecosystem identifier. If not specified, current ecosystem's parameters will be returned.

* *[names]*

    List of parameters to receive, separated by commas.

    Example: ``/api/v2/ecosystemparams/?names=name,currency,logo*``.

* *[vde]*

    Specify ``true``, if you want to recieve VDE params. Otherwise, do not specify this parameter.


Response
""""""""

* *list*

    An array where each element stores the following parameters:

    * *name*–parameter name
    * *value*–parameter value
    * *conditions*–conditions to change the parameter


Response example
""""""""""""""""

.. code-block:: default
    
    200 (OK)
    Content-Type: application/json
    {
        "list": [{ 
            "name": "name",
            "value": "MyState",
            "conditions": "true",
        }, 
        { 
            "name": "currency",
            "value": "MY",
            "conditions": "true",
        }, 
        ]
    }      


Errors
""""""

*E_ECOSYSTEM,E_VDE*


ecosystemparam/{name}
---------------------

**GET**/ Returns information about the **{name}** parameter in the current or specified ecosystem. 

Request
"""""""

.. code-block:: default
    
    GET
    /api/v2/ecosystemparam/{name}[?ecosystem=1]
    
* *name*–name of the requested parameter,
* *[ecosystem]*–ecosystem ID can be specified. The current ecosystem value will be returned by default,
* *[vde]*–specify ``true``, if you need to recieve VDE params, in the other case you don't need to specify this parameter.

Response
""""""""
    
* *name*–parameter name
* *value*–parameter value
* *conditions*–condition for parameter change
    

Response example
""""""""""""""""

.. code-block:: default
    
    200 (OK)
    Content-Type: application/json
    {
        "name": "currency",
        "value": "MYCUR",
        "conditions": "true"
    }      
    
Errors
""""""

*E_ECOSYSTEM,E_VDE*


tables/[?limit=...&offset=...]
------------------------------

**GET**/ Returns a list of tables in the current ecosystem. You can add set an offset and specify a number of requested tables. 

Request
"""""""

* *[limit]*–number of entries (25 by default),
* *[offset]*–entries start offset (0 by default),
* *[vde]*–specify *true*, if you need to recieve the list of the tables in VDE, in the other case you don't need to specify this parameter.

.. code-block:: default
    
    GET
    /api/v2/tables


Response
""""""""

* *count*–total number of entries in the table
* *list*–an array where each element stores the following parameters:

  * *name*–table name (returned without prefix)
  * *count*–number of entries in the table


Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "count": "100"
        "list": [{ 
            "name": "accounts",
            "count": "10",
        }, 
        { 
            "name": "citizens",
            "count": "5",
       }, 
        ]
    }    

Errors
""""""

*E_VDE* 

    
table/{name}
------------

**GET**/ Returns information about the requested table in the current ecosystem.

The next fields are returned: 

* *name*–table name
* *insert*–rights to insert the elements
* *new_column*–rights to insert the column 
* *update*–rights to change the rights
* *columns*–array of the columns with fields ``name, type, perm`` (name, type, rights for change).

Request
"""""""

.. code-block:: default 
    
    GET
    /api/v2/table/mytable
     
* *name*–table name (without ecosystem ID prefix),
* *[vde]*–specify *true*, if you need to recieve VDE params. In the other case you don't need to specify this parameter,

Response
""""""""

* *name*–table name (without ecosystem ID prefix)
* *insert*–right for adding an entry
* *new_column*–right for adding a column
* *update*–right for changing entries
* *conditions*–right for changing table configuration
* *columns*–an array of information about columns:

  * *name*–column name
  * *type*–column type. Possible values include: ``varchar,bytea,number,money,text,double,character``
  * *perm*–right for changing an entry in a column.
    
Response example
"""""""""""""""" 

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "name": "mytable",
        "insert": "ContractConditions(`MainCondition`)",
        "new_column": "ContractConditions(`MainCondition`)",
        "update": "ContractConditions(`MainCondition`)",
        "conditions": "ContractConditions(`MainCondition`)",
        "columns": [{"name": "mynum", "type": "number", "perm":"ContractConditions(`MainCondition`)" }, 
            {"name": "mytext", "type": "text", "perm":"ContractConditions(`MainCondition`)" }
        ]
    }      
    
Errors
""""""

*E_TABLENOTFOUND,E_VDE*  


list/{name}[?limit=...&offset=...&columns=]
-------------------------------------------

**GET**/ Returns a list of entries of the specified table in the current ecosystem. An offset and the number of requested table entries can be specified. 

Request
"""""""

* *name*–table name
* *[limit]*–number of entries (25 by default)
* *[offset]*–entries start offset (0 by default)
* *[columns]*–list of requested columns, separated by commas, if not specified, all columns will be returned. The id column will be returned in all cases
* *[vde]*–specify *true*, if you need to recieve records from the table in VDE. In the other case you don't need to specify this parameter.

.. code-block:: default 
    
    GET
    /api/v2/list/mytable?columns=name
    
Response
""""""""

* *count* - total number of entries in the table,
* *list* - an array where each element stores the following parameters:

  * *id* - entry ID,
  * sequence of requested columns. 

Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "count": "10"
        "list": [{ 
            "id": "1",
            "name": "John",
        }, 
        { 
            "id": "2",
            "name": "Mark",
       }, 
        ]
    }   

    
row/{tablename}/{id}[?columns=]
-------------------------------

**GET**/ Returns a table entry with specified id in the current ecosystem. Columns to be returned can be specified. 

Request
"""""""

* *tablename*–table name
* *id*–entry ID
* *[columns]*–a list of requested columns, separated by commas. If not specified, all columns will be returned. The id column will be returned in all cases.
* *[vde]*–specify *true*, if you need to recieve the record from the table in VDE, in the other case you don't need to specify this parameter.

.. code-block:: default 
    
    GET
    /api/v2/row/mytable/10?columns=name
    
Response
""""""""

* *value* - an array of received column values:

  * *id* - entry ID,
  * order of requested columns. 

Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "values": {
        "id": "10",
        "name": "John",
        }
    }   
    
systemparams
------------

**GET**/ Returns a list of system parameters.

Request
"""""""
 
.. code-block:: default 
    
    GET
    /api/v2/systemparams/[?names=...]

* *[names]* - list of requested parameters, a list of parameters to receive can be specified separated by commas. For instance, ``/api/v2/systemparams/?names=max_columns,max_indexes``.
 
Reply
"""""
 
* *list*–array, each element of which contains the following parameters:

* *name*–parameter name
* *value*–parameter value
* *conditions*–conditions for parameter change.


Response example
""""""""""""""""
 
.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "list": [{ 
            "name": "max_columns",
            "value": "100",
            "conditions": "ContractAccess("@0UpdSysParam")",
        }, 
        { 
            "name": "max_indexes",
            "value": "1",
            "conditions": "ContractAccess("@0UpdSysParam")",
        }, 
        ]
    }      


history/{name}/{id}
-------------------

 **GET**/ Returns the changelog of an entry in the specified table in the current ecosystem. 

Request
"""""""
 
 * *name*–table name,
 * *id*–entry identifier.
 
Reply
"""""

 * *list* an array, the elements of which contain modified parameters of the requested entry 
 
Reply Example
"""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "list": [
            {
                "name": "default_page",
                "value": "P(class, Default Ecosystem Page)"
            },
            {
                "menu": "default_menu"
            }
        ]
    }

interface/{page|menu|block}/{name}
----------------------------------

GET/ Searches the current ecosystem and returns a record from the selected table (page, menu or block) with the specified name.
 
.. code-block:: default 
    
    GET
    /api/v2/interface/page/default_page 

 
Request
"""""""
 
* *name*–name of the record in the specified table,
* *[vde]*–should be set to true, if the record is requested from a table on VDE; otherwise, this parameter should not be specified.
 
Response
""""""""
 
* *id* – identifier of the record,
* *name* – name of the record,
* *other* columns of the table.

Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "id": "1",
        "name": "default_page",
    "value": "P(Page content)",
    "default_menu": "default_menu",
    "validate_count": 1
    }   

Errors
""""""

*E_QUERY*, *E_NOTFOUND* 

Functions for working with contracts
====================================

contracts[?limit=...&offset=...]
--------------------------------

**GET**/ Returns a list of contracts in the current ecosystem. An offset and a number of requested contracts can be specified. 

Request
"""""""

* *[limit]*–number of entries (25 by default)
* *[offset]*–entries start offset (0 by default)
* *[vde]*–specify *true*, if you need to recieve the list of contracts from VDE, in the other case you don't need to specify this parameter.

.. code-block:: default 
    
    GET
    /api/v2/contracts

Response
""""""""

* *count*–total number of entries in the table
* *list*–an array where each element stores the following parameters:

  * *id*–entry ID
  * *name*–contract name
  * *value*–initial text of the contract
  * *active*–equals "1" if the contract is bound to the account or "0" otherwise
  * *key_id*–account bound to the contract
  * *address*–address of the account bound to the contract in the ``XXXX-...-XXXX`` format
  * *conditions*–conditions for change
  * *token_id*–identifier of the ecosystem, which currency will be used to pay for the contract

Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "count": "10"
        "list": [{ 
            "id": "1",
            "name": "MainCondition",
            "token_id":"1", 
            "key_id":"2061870654370469385", 
            "active":"0",
            "value":"contract MainCondition {
  conditions {
      if(StateVal(`founder_account`)!=$citizen)
      {
          warning `Sorry, you dont have access to this action.`
        }
      }
    }",
    "address":"0206-1870-6543-7046-9385",
    "conditions":"ContractConditions(`MainCondition`)"        
     }, 
    ...
      ]
    }   


contract/{name}
---------------

**GET**/ Returns information about the {name} smart contract. By default, the smart contract will be searched for in the current ecosystem.

Request
"""""""

* *name*–contract name,
* *[vde]*–set this parameter to true in case you request information about a contract from VDE, otherwise don't specify this parameter

.. code-block:: default 
    
    GET
    /api/v2/contract/mycontract
    
Response
""""""""

* *name*–name of the smart contract with ecosystem ID. Example: ``@{idecosystem}name``
* *active*–true if the contract is bound to the account and false otherwise,
* *key_id*–contract owner's ID,
* *address*–address of the account bound to the contract in the ``XXXX-...-XXXX`` format.
* *tableid*–entry ID in the contracts table, where the source code of the contract is stored.
* *fields*–an array that contains information about every parameter in the **data** section of the contract and contains the following fields:

  * *name*–field name,
  * *htmltype*–html type,
  * *type*–parameter type,
  * *tags*–parameter tags.
    
Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "fields" : [
            {"name":"amount", "htmltype":"textinput", "type":"int64", "tags": "optional"},
            {"name":"name", "htmltype":"textinput", "type":"string" "tags": ""}
        ],
        "name": "@1mycontract",
        "tableid" : 10,
        "active": true
    }      
    
contract/{request_id}
---------------------

**POST**/ Returns a smart contract based on the request identifier **{request_id}**. Before doing this, you should use the ``prepare/{name}`` (POST) command and sign the returned *forsign* field. The request with the **{request_id}** identifier is stored on server for 1 minute. If the contract is not executed within this minute, the request will be removed. In case of successful execution, the transaction hash is returned, which can be then used to receive the block number. Otherwise, an error message is returned.
 
Request
"""""""
 
* *request_id*–identifier of the request that was received from prepare,
* *[token_ecosystem]*–for contracts that are not bound to a wallet, you can specify which ecosystem’s currency will be used to pay for the contract; in this case the wallet and the public key of the token_ecosystem and the current ecosystem should be the same,
* *[max_sum]*–when calling contracts that are not bound to a wallet, you can specify the maximum amount that can be spent on execution of this contract,
* *[payover]*–for contracts that are not bound to a wallet you can specify additional payment for urgency – how much should be added to fuel_rate when calculating the payment,
* *[data]*–parameters sent to the contract,
* *signature*–hex signature of the forsign value, received from prepare,
* *time*–time, returned by prepare,
* *pubkey*–hex of the contract signer's public key; it should be noted, that if the public key is already stored in the keys table of the current ecosystem, you don't need to send it,
* *[vde]*–should be set to true, if the smart contract is requested from VDE; otherwise, you don't need to specify this parameter.
 
.. code-block:: default 
 
    POST
    /api/v2/contract/5c273816-134e-4a50-89b2-8d2b3d5ba562
    signature - hex signature
    time - time returned by prepare


Response
""""""""

* *hash* - hex hash of the sent transaction.


Response example
""""""""""""""""

.. code-block:: default 

    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634.....",
    }

Errors
""""""

*E_CONTRACT, E_EMPTYPUBLIC, E_EMPTYSIGN, E_NOTFOUNDREQUEST*


contractMultiple/{request_id}
-----------------------------

**POST**/ Executres contracts for **{request_id}** request. The :ref:`prepareMultiple/{name} <preparemultiple>` (POST) request must be called before, and all strings in the *forsign* field must be signed. Server keeps the request with **{request_id}** identifier for 1 minute. If the contracts are not executed during this time, the request is deleted. After a successful execution of this request, hashes of transactions are returned. Using these hashes, information about contracts execution can be obtained. In case of a successful execution, the block identifier can be obtained. In case of an error, the error message can be obtained.

Request
"""""""

* *request_id*–identifier of a multi-request that was obtained from the prepareMultiple request.
* *data*–json with the following fields:
 
    * *[token_ecosystem]*–for contracts that are not bound to a wallet, you can specify the which ecosystem’s currency will be used to pay for the contract; in this case the wallet and the public key of the token_ecosystem and the current ecosystem should be the same.
    * *[max_sum]*–when calling a contract that is not bound to a wallet, you can specify the maximum amount that can be spent on execution of this contract.
    * *[payover]*–for contracts that are not bound to a wallet you can specify additional payment for urgency – how much should be added to fuel_rate when calculating the payment.
    * parameters that are passed to the contract
    * *signatures*–array of hex signatures of *forsign* strings that was received from prepareMultiple.
    * *time*–time value returned by prepare.
    * *[pubkey]*–hex public key of a user that signed the contract. If this public key is already stored in the ``keys`` table of this ecosystem, this parameter can be omitted.

    .. code-block:: default 

        {"time": 1234552 , "signatures": ["fgdazgdagdag", "agaaadg"]}

    .. code-block:: default 
 
        POST
        /api/v2/contract/5c273816-134e-4a50-89b2-8d2b3d5ba562
        signatures - array of hex signatures
        time - time returned by prepare

Response
""""""""

* *hashes*–hex hashes of sent transactions.

Response example
""""""""""""""""

.. code-block:: default 

    200 (OK)
    Content-Type: application/json
    {
        "hashes" : ["67afbc435634.....","67adab435634....."]
    }

Errors
""""""
 
*E_CONTRACT, E_EMPTYPUBLIC, E_EMPTYSIGN, E_NOTFOUNDREQUEST*


prepare/name
------------

**POST**/ Sends a request to receive a string to sign the specified contract. The **{name}** should state the name of transaction for which the string for signature should be returned. The forsign parameter returns a string, which should be signed. Also, returned are the request_id and time parameters, which should be transferred along with the signature.
 
Request
"""""""
 
* *name*–contract name; if another ecosystem's contract is executed, its full name should be specified (@1MainContract).
* *[token_ecosystem]*–for contracts that are not bound to a wallet, you can specify the which ecosystem’s currency will be used to pay for the contract; in this case the wallet and the public key of the token_ecosystem and the current ecosystem should be the same,
* *[max_sum]*–when calling a contract that is not bound to a wallet, you can specify the maximum amount that can be spent on execution of this contract,
* *[payover]*–for contracts that are not bound to a wallet you can specify additional payment for urgency – how much should be added to fuel_rate when calculating the payment,
* *[vde]*–should be set to true, if the smart contract is requested from a VDE; otherwise, you don't need to specify this parameter.
* *[data]*–parameters sent to the contract.

.. code-block:: default
    
    POST
    /api/v2/prepare/mycontract


Response
""""""""

* *request_id*–the identifier of the request to be transmitted.
* *forsign*–string to be signed,
* *time*–time information, which needs to be sent together with the contract.


Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "request_id": "5c273816-134e-4a50-89b2-8d2b3d5ba562",
        "time": 423523768,
        "forsign": "......", 
    }

Errors
""""""

*E_CONTRACT*    

.. _preparemultiple:

prepareMultiple
---------------

**POST**/ Sends a request to receive strings to sign for the specified contract. The strings to be signed are returned in the *forsign* parameter. The *request_id* and *time* parameters are also returned. These parameters must be passed along with the signature.

Request
"""""""

* *data* - json with the following fields:

    * *[token_ecosystem]*–for contracts that are not bound to a wallet, you can specify the which ecosystem’s currency will be used to pay for the contract; in this case the wallet and the public key of the token_ecosystem and the current ecosystem should be the same,
    * *[max_sum]*–when calling a contract that is not bound to a wallet, you can specify the maximum amount that can be spent on execution of this contract,
    * *[payover]*–for contracts that are not bound to a wallet you can specify additional payment for urgency – how much should be added to fuel_rate when calculating the payment,
    * *[vde]*–should be set to true, if the smart contract is requested from a VDE; otherwise, you don't need to specify this parameter.
    * *contracts*–array of objects where each object is a contract call with parameters:
        * *contract*–contract name; if another ecosystem's contract is executed, its full name should be specified (``@1MainContract``).
        * *params*– a dictionary of contract parameters. All values must be strings.

.. code-block:: default 
    
    POST
    /api/v2/prepareMultiple

.. code-block:: default 

    {
      "token_ecosystem": "",
      "max_sum":"",
      "payover": "",
      "signed_by": "",
      "contracts": [
        {"contract": "ContractOne", "params": {"Param1": "Value1", "Param2": "Value2"}},
    {"contract": "ContractTwo", "params": {"Param11": "Value11", "Param21": "Value21"}}
      ]
    }

Response
""""""""

* *request_id*–the identifier of the request to be transmitted.
* *forsign*–strings to be signed, each string corresponds to a key from the dictionary that was passed in *params*.
* *time*–time information, which needs to be sent together with the contract.

Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "request_id": "5c273816-134e-4a50-89b2-8d2b3d5ba562",
        "time": 423523768,
        "forsign": ["......", "......"]
    }


Errors
""""""

*E_CONTRACT*

  
txstatus/{hash}
---------------

**GET**/ Returns a block number or an error of the sent transaction with given hash. If the returned values of *blockid* and *errmsg* are empty, then the transaction hasn't yet been included into a block.

Request
"""""""

* *hash*–hash of the checked transaction.

.. code-block:: default 
    
    GET
    /api/v2/txstatus/2353467abcd7436ef47438
     
Response
""""""""

* *blockid*–number of the block in case the transaction has been processed successfully
* *result*–result of the transaction operation, returned through the **$result** variable
* *errmsg*–error message in case the transaction was refused.


Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "blockid": "4235237",
        "result": ""
    }      


Errors
""""""

*E_HASHWRONG, E_HASHNOTFOUND*


txstatusMultiple/
-----------------

**GET**/ Returns a block number or an error of the sent transaction with given hashes. If the returned values of *blockid* and *errmsg* are empty, then the transaction hasn't yet been included into a block.

Request
"""""""

* *data*–json with a list of transaction hashes.

.. code-block:: default 

     {"hashes":["contract1hash", "contract2hash", "contract3hash"]}

.. code-block:: default 
    
    GET
    /api/v2/txstatusMultiple/
    
Response
""""""""

* *results*–a dictionary that contains transaction hashes as keys and check results as values.

        *hash*–transaction hash

                * *blockid*–number of the block in case the transaction has been processed successfully
                * *result*–result of the transaction operation, returned through the **$result** variable
                * *errmsg*–error message in case the transaction was refused.
    
Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {"results":
      { 
        "hash1": {
             "blockid": "3123",
             "result": "",
         },
         "hash2": {
              "blockid": "3124",
              "result": "",
         }
       }
     }

Errors
""""""

*E_HASHWRONG, E_HASHNOTFOUND*


content/{menu|page}/{name}
--------------------------

**POST**/ Returns a JSON representation of the code of the specified page or menu named **{name}**, which is the result of processing by the template engine. The request can have additional parameters, which can be used in the template engine. If the page or menu can't be found, the 404 error is returned.
    
Request
"""""""

* *menu|page*–*page* or *menu* to recieve the page or menu

* *name*–the name or menu of the page

*[lang]*–either lcid or a two-letter language code can be specified to address the corresponding language resources. For example, *en,ru,fr,en-US,en-GB*. If, for example, the *en-US* resource will not be found, the *en* resources will be used instead of the missing *en-US* ones,

* *[app_id]*–application ID. Passed together with lang, because the functions that work with the language in the template engine don’t automatically recognize the AppID. Should be passed as a number,

* *[vde]*–specify *true*, if you recieve data from the page or menu in VDE. Otherwise, you do not need to specify this parameter.

.. code-block:: default 
    
    POST
    /api/v2/content/page/default

    
Response
""""""""

* *menu*–the menu name for the page when calling *content/page/...*
* *menutree*–JSON menu tree for the page when calling *content/page/...*
* *title*–head for the menu *content/menu/...*
* *tree*–JSON tree of objects


Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "tree": {"type":"......", 
              "children": [
                   {...},
                   {...}
              ]
        },
    }      


Errors
""""""

*E_NOTFOUND*


content/source/{name}
---------------------

**POST**/ Returns a JSON-representation of the **{name}** page code without executing any functions or receiving any data. The returned tree corresponds to the page template and can be used in the visual designer. If the page or the menu are not found, a 404 error is returned.
 
Request
"""""""
 
* *name*–name of the requested page,
* *[vde]*–*true* should be set to true, if the page or menu is requested from VDE; otherwise, this parameter should not be specified.


Response
""""""""

.. code-block:: default 
    
    POST
    /api/v2/content/source/default


* *tree*–JSON object tree.
 

Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "tree": {"type":"......", 
              "children": [
                   {...},
                   {...}
              ]
        },
    }      
 

Errors
""""""

*E_NOTFOUND, E_SERVER*



content/hash/{name}
-------------------

**POST**/ Returns a SHA256 hash of the **{name}** page. If the page or menu is not found, a 404 error is returned.

This method does not reqire authorization. Because of this, to receive a correct hash when making a request to other nodes, *ecosystem*, *keyID*, *roleID*, and *isMobile* parameters must be also passed. To receive pages from other systems, a ``@ecosystem_id`` prefix must be added to the page name. For example: ``@2mypage``.

Request
"""""""

* *name*–page name
* *ecosystem*–ecosystem identifier
* *keyID*–user identifier
* *roleID*–user role identifier
* *isMobile*–mobile platform execution flag

.. code-block:: default 
    
    POST
    /api/v2/content/hash/default

Response
""""""""

* *hex*–resulting hash in the hex string format

Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "hash": "01fa34b589...."
    }      

Errors
""""""

*E_NOTFOUND, E_SERVER, E_HEAVYPAGE*


content
-------

**POST**/ Returns a JSON-representation of the page source code from the **template** parameter. If the additional parameter **source** is specified as true or 1, the JSON-representation will be returned without execution of functions and without receiving data. The returned tree corresponds to the sent template and can be used in the visual designer.

 
Request
"""""""
 
* *template*–page template source code to be processed
* *[source]*–if set to true or 1, the tree will be returned without execution of functions and without receiving data.
 
.. code-block:: default
    
    POST
    /api/v2/content


Response
""""""""
 
* *tree*–JSON object tree.

Response example
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "tree": {"type":"......", 
              "children": [
                   {...},
                   {...}
              ]
        },
    }      

Errors
""""""

*E_NOTFOUND, E_SERVER*


node/{name}
-----------

**POST** Calls the **{name}** smart contract on behalf of a node. Used for calling smart contracts from VDE contracts though the **HTTPRequest** function. Since in this case the contract can't be signed with an account key, it will be signed with the node's private key. All other parameters are similar to those when sending a contract. The called contract should be bound to an account, because the node's private key account does not have enough funds to execute the contract. If the contract is called from a VDE contract, then the authorization token **$auth_token** should be passed to **HTTPRequest**.

.. code-block:: js

    var pars, heads map
    heads["Authorization"] = "Bearer " + $auth_token
    pars["vde"] = "false"
    ret = HTTPRequest("http://localhost:7079/api/v2/node/mycontract", "POST", heads, pars)


Request
"""""""

.. code-block:: default 
 
    POST
    /api/v2/node/mycontract


Response
""""""""

* *hash*–hex hash of the sent transaction


Reply example
"""""""""""""

.. code-block:: default 

    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634.....",
    }


maxblockid
----------

**GET**/ Returns the highest block ID on the current node. 

Request
"""""""

.. code-block:: default 
 
    GET
    /api/v2/maxblockid


Reply
"""""

* *max_block_id*–highest block id on the current node


Reply Example
"""""""""""""

.. code-block:: default 

    200 (OK)
    Content-Type: application/json
    {
        "max_block_id" : 341,
    }

Errors
""""""

*E_NOTFOUND*


block/{id}
----------

**GET**/ Returns information on the block with the specified ID.

Request
"""""""

* *id*–id of the requested block.

.. code-block:: default 
    
    POST
    /api/v2/block/32

Reply
"""""

* *hash*–hash of the block
* *ecosystem_id*–ecosystem id
* *key_id*–key which signed the block
* *time*–block generation timestamp
* *tx_count*–number of transactions in the block
* *rollbacks_hash*–hash of rollbacks, created by transactions in the block

Reply example
"""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: application/json
    {
        "hash": "\x1214451d1144a51",
        "ecosystem_id": 1,
        "key_id": -13646477,
        "time": 134415251,
        "tx_count": 3,
        "rollbacks_hash": "\xa1234b1234"
    }      


Errors
""""""

*E_NOTFOUND*


avatar/{ecosystem}/{member}
---------------------------

**GET**/ Returns user's avatar (available without login).

 
Request
"""""""
 
* *ecosystem*–user's ecosystem ID
* *member*–user ID 
 
.. code-block:: default 
    
    GET
    /api/v2/avatar/1/7136200061669836581


Response
""""""""
 
Header Content-Type with the image type. Image data is returned in the in the response body. 
 
Response example 
""""""""""""""""

.. code-block:: default 
    
    200 (OK)
    Content-Type: image/png  

Errors
""""""

*E_NOTFOUND* *E_SERVER*


config/centrifugo
-----------------

**GET**/ Returns centrifugo's host and port (available without login)
 
Request
"""""""

.. code-block:: default 
    
    GET
    /api/v2/config/centrifugo

Response
""""""""

String in the ``http://address:port`` format that is returned in the response body. Example: ``http://127.0.0.1:8000``
 
Errors
""""""

*E_SERVER*
 



