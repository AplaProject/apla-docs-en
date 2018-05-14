################################################################################
REST API v2
################################################################################
All functions, available from the Molis software client, including authentication, receipt of data about ecosystems, error handling, operations with database tables, interface pages, and execution of contracts (network transactions) are available though REST API of the platform. Thus, by using REST API developers can access any function of the platform without using the Molis software client.

Command calls are performed by addressing ``/api/v2/command/[param]``, where **command** is a command name, and **param** is an additional parameter (for example, the name of the resource to change or receive). Request parameters should be sent with ``Content-Type: x-www-form-urlencoded``. The server response will be sent in JSON format.

********************************************************************************
Error Handling
********************************************************************************

In case of successful query execution, returned is status 200. In case of an error, apart from an error status, returned is a JSON object with the following fields:

* **error** - error identifier,
* **msg** - error text,
* **params** - array of additional parameters of the error, which can be put into the error message.

Response example 

.. code:: 

    400 (Bad Request)
    Content-Type: application/json
    {
        "err": "E_INVALIDWALLET",
        "msg": "Wallet 1111-2222-3333 is not valid",
        "params": ["1111-2222-3333"]
    }

Error List

* **E_CONTRACT** - There is not %s contract,
* **E_DBNIL** - DB is nil,
* **E_ECOSYSTEM** - Ecosystem %d doesn't exist,
* **E_EMPTYPUBLIC** - Public key is undefined,
* **E_EMPTYSIGN** - Signature is undefined,
* **E_HASHWRONG** - Hash is incorrect,
* **E_HASHNOTFOUND** - Hash has not been found,
* **E_INSTALLED** - Platform is already installed,
* **E_INVALIDWALLET** - Wallet %s is not valid,
* **E_NOTFOUND** - Content page or menu has not been found,
* **E_NOTINSTALLED** -  Platform is not installed. In this case, you need to run the *install* by the command,
* **E_QUERY** - DB query is wrong,
* **E_RECOVERED** - API recovered returtns if there is panic error,
* **E_REFRESHTOKEN** - Refresh token is not valid,
* **E_SERVER** - Server error. Returns if there is an error in the library functions golang. The *msg* field contains the error text,
* **E_SIGNATURE** - Signature is incorrect,
* **E_STATELOGIN** - %s is not a membership of ecosystem %s,
* **E_TABLENOTFOUND** - Table %s has not been found,
* **E_TOKEN** - Token is not valid,
* **E_TOKENEXPIRED** - Token is expired by %s,
* **E_UNAUTHORIZED** - Unauthorized,
* **E_UNDEFINEVAL** - Value %s is undefined,
* **E_UNKNOWNUID** - Unknown uid,
* **E_VDE** - Virtual Dedicated Ecosystem %s doesn't exist,
* **E_VDECREATED** - Virtual Dedicated Ecosystem is already created.


The **E_RECOVERED** error means that you encountered a bug that needs to be found and fixed. The **E_NOTINSTALLED** error should be returned by any command except for install, in case the system is not yet installed. The **E_SERVER** error may appear in response to any command. If it appears as a result of incorrect input parameters, it can be changed to relevant errors. In the other case, this error reports of invalid operation or incorrect system configuration, which requires a more detailed investigation. The **E_UNAUTHORIZED** error can be returned for any command except for *install, getuid, login* in cases where login was not performed or the session has expired.

********************************************************************************
Authentication
********************************************************************************

The **JWT token** http://www.jwt.org is used for authentication. After receiving a JWT token you need to put it in the header of every query: ``Authorization: Bearer TOKEN_HERE``. 

getuid
==============================
**GET**/ Returns a unique value, which needs to be signed with your private key and sent back to server using the **login** command. Currently, a temporary JWT token is created, which needs to be passed to **Authorization** when calling **login**.

.. code:: 
    
    GET
    /api/v2/getuid
    
    Response

* *uid* - signature line.
* *token* - temporary token to pass in login. Currently, the lifetime of a temporary token is 5 seconds.

In cases where authorization is not required, returned is the following:

* *expire* - number of seconds before the time runs out, 
* *ecosystem* - ecosystem ID,
* *key_id* - wallet ID,
* *address* - wallet address in the ``XXXX-XXXX-.....-XXXX`` format.
    
Response Example

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "uid": "28726874268427424",
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........AU3yPRp64SLO4aJqhN-kMoU5HNYTDplQXbVu0Y"
    }
    
    Errors: *E_SERVER*   

login
==============================
**POST**/ User authentication. The **getuid** command should be called in the first place in order to receive a unique value and sign it. A temporary JWT token, which was received along with getuid, should be passed in the header. In case of success, the token that was received in the response should be included in all queries in the *Authorization* header.

Query

.. code:: 

    POST
    /api/v2/login
    
* *[ecosystem]* - ecosystem ID. If not specified, the command will work with the first ecosystem,
* *[expire]* - lifetime of the JWT token in seconds (36000 by default),
* *[pubkey]* - public hex key, if the blockchain already stores a key, then the wallet number should be passed with the *key_id* parameter,
* *[key_id]* - account id or ``XXXX-...-XXXX`` format. Use this in cases where the public key is already stored in the blockchain. Can't be used together with *pubkey*,
* *signature* - a uid signature received though getuid hex.

Response

* *token* - JWT token,
* *refresh* - JWT token to extend the session. Should be sent in the **refresh** command,
* *ecosystem* - ecosystem ID,
* *key_id* - account ID,
* *address* - account address in the ``XXXX-XXXX-.....-XXXX`` format,
* *notify_key* - key for notifications,
* *isnode* - true or false - is this user the owner of this node,
* *isowner* - true or false - is this user the owner of this ecosystem,
* *vde* - true or false - does this ecosystem have a virtual dedicated ecosystem.

Response Example 

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........AU3yPRp64SLO4aJqhN-kMoU5HNYT8fNGODp0Y"
        "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........iOiI1Nzk3ODE3NjYwNDM2ODA5MzQ2Iiw"        
        "ecosystem":"1",
        "key_id":"12345",
        "address": "1234-....-3424"
    }      
    
 Errors: *E_SERVER, E_UNKNOWNUID, E_SIGNATURE, E_STATELOGIN, E_EMPTYPUBLIC* 
    
refresh
==============================
**POST**/ Issues new tokens and extends the user session. In case of successful completion you need to send the token, which was received in response, in the *Authorization* header of all queries.

Query

.. code:: 

    POST
    /api/v2/refresh
    
* *[expire]* - lifetime of the JWT token in seconds (36000 by default).
* *token* - refresh token from the previous **login** or **refresh** calls.

Response

* *token* - JWT token.
* *refresh* - JWT token for session extension. Should be sent to the **refresh** command.

Response Example

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........AU3yPRp64SLO4aJqhN-kMoU5HNYT8fNGODplQXbVu0Y"
        "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6I........iOiI1Nzk3ODE3NjYwNDM2ODA5MzQ2Iiw"        
    }     
    
Errors: *E_SERVER, E_TOKEN, E_REFRESHTOKEN* 

signtest
==============================
**POST**/ This command signs the string with the specified private key. It should be used only for API-testing purposes, since in actual operation private keys should not be passed to the server. The private key can be found in the directory from which the server is launched. If the signature is formed for subsequent transfer to /login, then the LOGINforsign string should be passed instead of forsign.

.. code:: 
    
    POST
    /api/v2/signtest
    
* *private* - hex private key,
* *forsign* - string for signing.

Response

* *signature* - signature in hexadecimal, 
* *pubkey* - public key for the sent hex private key.
    
Response Example

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "signature": "0011fa...",
        "pubkey": "324bd7..."
    }      

Errors: *E_SERVER* 

********************************************************************************
Service Commands
********************************************************************************

install
==============================
**POST**/ Starts the installation process. After successful installation, the system should be restarted. 

Query

.. code:: 

    POST
    /api/v2/install
    
* *type* - installation type: **PRIVATE_NET, TESTNET_NODE, TESTNET_URL**,
* *log_level* - level of logging: **ERROR, DEBUG**,
* *first_load_blockchain_url* - address to obtain the blockchain, is specified in case of *type* it is set as *TESTNET_URL*,
* *db_host* - host for PostgreSQL DB. For example, *localhost*,
* *db_port* - port for the PostgreSQL DB. For example, *5432*,
* *db_name* - name of the PostgreSQL DB. For example, *mydb*,
* *db_user* - username to connect to the PostgreSQL DB, for example, *postgres*,
* *db_pass* - password to connect to the PostgreSQL DB, for example, *postgres*,
* *generate_first_block* - can be 0 or 1 when *type* is *Private-net*,
* *first_block_dir* - directory that contains the *1block* file with the fist block, should be specified when, *generate_first_block* is 0 and *type* is *PRIVATE_NET*.

Response

* *success* - true in case of successful completion.

Response Example

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "success": true
    }      
    
    Errors: *E_SERVER, E_INSTALLED, E_DBNIL* 
    
version
==============================
**GET**/ Returns the current server version.
 
Request

.. code:: 

    GET
    /api/v2/version
    
Response Example

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    "0.1.6"

********************************************************************************
Data Request Functions
********************************************************************************

balance
==============================
**GET**/ Requests the balance of an account in the current ecosystem. 

Query

.. code:: 
    
    GET
    /api/v2/balance/{key_id}
    
* *key_id* - account id can be specified in any format - ``int64, uint64, XXXX-...-XXXX``. This wallet will be searched for in the ecosystem, which the user is currently logged in.   
    
Response

* *amount* - account balance in minimum units,
* *money* - account balance in units.
    
Response Example

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "amount": "123450000000000000000",
        "money": "123.45"
    }      
    
Errors: E_SERVER, E_INVALIDWALLET
    
********************************************************************************
Work with Ecosystems
********************************************************************************

ecosystems
==============================
**GET**/ Returns a number of ecosystems.

.. code:: 
    
    GET
    /api/v2/ecosystems/

Response

* *number* - the number of ecosystems installed.
    
Response Example

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "number": 100,
    }      

vde/create
==============================
**POST**/ Creates Virtal Dedicated Ecosystem (VDE) for the current ecosystem.

.. code:: 
    
    POST
    /api/v2/vde/create

Rsponse

* *result* - returns *true*, if VDE have been created.
    
Response Example

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "result": true,
    }     
    
Errors: *E_VDECREATED*

appparams
==============================
 **GET**/ Returns a list of application parameters in the current or specified ecosystem.
 
 Request
 
 * *[appid]* - application identifier,
 * *[ecosystem]* - ecosystem ID; if not specified, the current ecosystem's parameters will be returned,
 * *[names]* - list of received parameters; a list of parameter names separated by commas can be specified, for example: ``/api/v2/appparams/1?names=name,mypar``.

Response
 
 * *list* - an array, where each element contains the following parameters:
 * *name* - parameter name,
 * *value* - parameter value,
 * *conditions* - permissions to change a parameter.
 
Response Example

.. code:: 
    
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

Errors: *E_ECOSYSTEM*

appparam/{appid}/{name}
==============================
 **GET**/ Returns information about the **{name}** parameter of the **{appid}** application in the current or specified ecosystem. 
 
* *appid* - application ID,
* *name* - name of the requested parameter,
* *[ecosystem]* - ecosystem ID (optional parameter). By default, the current ecosystem will be considered.
 
 Request
 
.. code:: 
    
    GET
    /api/v2/{appid}/{appid}/{name}[?ecosystem=1]
    
 Response   
     
 * *id* - parameter identifier,
 * *name* - parameter name,
 * *value* - parameter value,
 * *conditions* - conditions to change a parameter.
     
Response Example

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "id": "10",
        "name": "par",
        "value": "My value",
        "conditions": "true"
    }      
 
Errors: *E_ECOSYSTEM,E_PARAMNOTFOUND*


ecosystemparams
==============================
**GET**/ Returns a list of ecosystem parameters. 

Query

.. code:: 
    
    GET
    /api/v2/ecosystemparams/[?ecosystem=...&names=...]
    
* *[ecosystem]* - ecosystem identifier. If not specified, current ecosystem's parameters will be returned.
* *[names]* - list of parameters to receive, separated by commas, example: ``/api/v2/ecosystemparams/?names=name,currency,logo*``,
* *[vde]* - specify ``true``, if you need to recieve VDE params. In the other case you don't need to specify this parameter.


Response

* *list* - an array where each element stores the following parameters:

  * *name* - parameter name,
  * *value* - parameter value,
  * *conditions* - conditions to change the parameter.

Response Example

.. code:: 
    
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
    
Errors: *E_ECOSYSTEM,E_VDE*

ecosystemparam/{name}
==============================
**GET**/ Returns information about the **{name}** parameter in the current or specified ecosystem. 

Query

.. code:: 
    
    GET
    /api/v2/ecosystemparam/{name}[?ecosystem=1]
    
* *name* - name of the requested parameter,
* *[ecosystem]* - ecosystem ID can be specified. The current ecosystem value will be returned by default,
* *[vde]* - specify ``true``, if you need to recieve VDE params, in the other case you don't need to specify this parameter.

Response
    
* *name* - parameter name,
* *value* - parameter value,
* *conditions* - condition for parameter change. 
    
Response Example

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "name": "currency",
        "value": "MYCUR",
        "conditions": "true"
    }      
    
Errors: **E_ECOSYSTEM,E_VDE*

tables/[?limit=...&offset=...]
==============================
**GET**/ Returns a list of tables in the current ecosystem. You can add set an offset and specify a number of requested tables. 

Query

* *[limit]* - number of entries (25 by default),
* *[offset]* - entries start offset (0 by default),
* *[vde]* - specify *true*, if you need to recieve the list of the tables in VDE, in the other case you don't need to specify this parameter.

.. code:: 
    
    GET
    /api/v2/tables
    
Response

* *count* - total number of entries in the table,
* *list* - an array where each element stores the following parameters:

  * *name* - table name (returned without prefix),
  * *count* - number of entries in the table.

Response Example

.. code:: 
    
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
    
    Ошибки: *E_VDE* 
    
table/{name}
==============================
**GET**/ Returns information about the requested table in the current ecosystem.

The next fields return: 

* *name* - table name, 
* *insert* - rights to insert the elements, 
* *new_column* - rights to insert the column, 
* *update* - rights to change the rights, 
* *columns* - array of the columns with fields ``name, type, perm`` - name, type, rights for change.

Query

.. code:: 
    
    GET
    /api/v2/table/mytable
     
* *name* - table name (without ecosystem ID prefix),
* *[vde]* - specify *true*, if you need to recieve VDE params. In the other case you don't need to specify this parameter,

Response

* *name* - table name (without ecosystem ID prefix),
* *insert* - right for adding an entry,
* *new_column* - right for adding a column,
* *update* - right for changing entries,
* *conditions* - right for changing table configuration,
* *columns* - an array of information about columns:

  * *name* - column name,
  * *type* - column type. Possible values include: ``varchar,bytea,number,money,text,double,character``,
  * *perm* - right for changing an entry in a column.
    
Response Example 

.. code:: 
    
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
    
Errors: *E_TABLENOTFOUND,E_VDE*  

list/{name}[?limit=...&offset=...&columns=]
==============================
**GET**/ Returns a list of entries of the specified table in the current ecosystem. An offset and the number of requested table entries can be specified. 

Query

* *name* - table name,
* *[limit]* - number of entries (25 by default),
* *[offset]* - entries start offset (0 by default),
* *[columns]* - list of requested columns, separated by commas, if not specified, all columns will be returned. The id column will be returned in all cases,
* *[vde]* - specify *true*, if you need to recieve records from the table in VDE. In the other case you don't need to specify this parameter.

.. code:: 
    
    GET
    /api/v2/list/mytable?columns=name
    
Response

* *count* - total number of entries in the table,
* *list* - an array where each element stores the following parameters:

  * *id* - entry ID,
  * order of requested columns. 

Response Example

.. code:: 
    
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
==============================
**GET**/ Returns a table entry with specified id in the current ecosystem. Columns to be returned can be specified. 

Query

* *tablename* - table name,
* *id* - entry ID,
* *[columns]* - a list of requested columns, separated by commas. If not specified, all columns will be returned. The id column will be returned in all cases,
* *[vde]* - specify *true*, if you need to recieve the record from the table in VDE, in the other case you don't need to specify this parameter.

.. code:: 
    
    GET
    /api/v2/row/mytable/10?columns=name
    
Response

* *value* - an array of received column values:

  * *id* - entry ID,
  * order of requested columns. 

Response Example

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "values": {
        "id": "10",
        "name": "John",
        }
    }   
    
systemparams
==============================
**GET**/ Returns a list of system parameters.

Query
 
.. code:: 
    
    GET
    /api/v2/systemparams/[?names=...]

* *[names]* - list of requested parameters, a list of parameters to receive can be specified separated by commas. For instance, ``/api/v2/systemparams/?names=max_columns,max_indexes``.
 
Reply 
 
* *list* - array, each element of which contains the following parameters:

* *name* - parameter name,
* *value* - parameter value,
* *conditions* - conditions for parameter change.

 Response example
 
 .. code:: 
    
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
==============================
 **GET**/ Returns the changelog of an entry in the specified table in the current ecosystem. 

Request
 
 * *name* - table name,
 * *id* - entry identifier.
 
Reply 
 * *list* an array, the elements of which contain modified parameters of the requested entry 
 
Reply Example
  
.. code:: 
    
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
==================================
GET/ Searches the current ecosystem and returns a record from the selected table (page, menu or block) with the specified name.
 
.. code:: 
    
    GET
    /api/v2/interface/page/default_page 
 
Request
 
* *name* – name of the record in the specified table,
* *[vde]* – should be set to true, if the record is requested from a table on VDE; otherwise, this parameter should not be specified.
 
Response
 
* *id* – identifier of the record,
* *name* – name of the record,
* *other* columns of the table.

Response Examples

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "id": "1",
        "name": "default_page",
	"value": "P(Page content)",
	"default_menu": "default_menu",
	"validate_count": 1
    }   

Errors: *E_QUERY*, *E_NOTFOUND* 

********************************************************************************
Functions for Work with Contracts
********************************************************************************

contracts[?limit=...&offset=...]
==============================
**GET**/ Returns a list of contracts in the current ecosystem. An offset and a number of requested contracts can be specified. 

Query

* *[limit]* - number of entries (25 by default),
* *[offset]* - entries start offset (0 by default),
* *[vde]* - specify *true*, if you need to recieve the list of contracts from VDE, in the other case you don't need to specify this parameter.

.. code:: 
    
    GET
    /api/v2/contracts

Response

* *count* - total number of entries in the table,
* *list* - an array where each element stores the following parameters:

  * *id* - entry ID,
  * *name* - contract name,
  * *value* - initial text of the contract,
  * *active* - equals "1" if the contract is bound to the account or "0" otherwise,
  * *key_id* - account bound to the contract, 
  * *address* - address of the account bound to the contract in the ``XXXX-...-XXXX`` format, 
  * *conditions* - conditions for change.
  * *token_id* - identifier of the ecosystem, which currency will be used to pay for the contract.

Response Example

.. code:: 
    
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
==============================
**GET**/ Returns information about the {name} smart contract. By default, the smart contract will be searched for in the current ecosystem.

Response

* *name* – contract name,
* *[vde]* – set this parameter to true in case you request information about a contract from VDE, otherwise don't specify this parameter

.. code:: 
    
    GET
    /api/v2/contract/mycontract
    
Response

* *name* - name of the smart contract with ecosystem ID. Example: ``@{idecosystem}name``,
* *active* - true if the contract is bound to the account and false otherwise,
* *key_id* - contract owner's ID,
* *address* - address of the account bound to the contract in the ``XXXX-...-XXXX`` format.
* *tableid* - entry ID in the contracts table, where the source code of the contract is stored.
* *fields* -  an array that contains information about every parameter in the **data** section of the contract and contains the following fields:

  * *name* - field name,
  * *htmltype* - html type,
  * *type* - parameter type,
  * *tags* - parameter tags.
    
Response Example

.. code:: 
    
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
==============================
**POST**/ Returns a smart contract based on the request identifier **{request_id}**. Before doing this, you should use the ``prepare/{name}`` (POST) command and sign the returned field *forsign*. The request with the **{request_id}** identifier is stored on server for 1 minute. If the contract is not executed within this minute, the request will be removed. In case of successful execution, the transaction hash is returned, which can be then used to receive the block number. Otherwise, an error message is returned.
 
Request
 
* *request_id* – identifier of the request that was received from prepare,
* *[token_ecosystem]* – for contracts that are not bound to a wallet, you can specify which ecosystem’s currency will be used to pay for the contract; in this case the wallet and the public key of the token_ecosystem and the current ecosystem should be the same,
* *[max_sum]* – when calling contracts that are not bound to a wallet, you can specify the maximum amount that can be spent on execution of this contract,
* *[payover]* – for contracts that are not bound to a wallet you can specify additional payment for urgency – how much should be added to fuel_rate when calculating the payment,
* *[data]* – parameters sent to the contract,
* *signature* - hex signature of the forsign value, received from prepare,
* *time* - time, returned by prepare,
* *pubkey* - hex of the contract signer's public key; it should be noted, that if the public key is already stored in the keys table of the current ecosystem, you don't need to send it,
* *[vde]* – should be set to true, if the smart contract is requested from VDE; otherwise, you don't need to specify this parameter.
 
.. code:: 
 
    POST
    /api/v2/contract/5c273816-134e-4a50-89b2-8d2b3d5ba562
    signature - hex подпись
    time - время, возвращенное prepare

Response

* *hash* - hex хэш отправленной транзакции.

Response example

.. code:: 

    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634.....",
    }

ERRORS: *E_CONTRACT, E_EMPTYPUBLIC, E_EMPTYSIGN, E_NOTFOUNDREQUEST*

    
prepare/name
==============================
**POST**/ Sends a request to receive a string to sign the specified contract. The **{name}** should state the name of transaction for which the string for signature should be returned. The forsign parameter returns a string, which should be signed. Also, returned are the request_id and time parameters, which should be transferred along with the signature.
 
Request
 
* *name* – contract name; if another ecosystem's contract is executed, its full name should be specified (@1MainContract).
* *[token_ecosystem]* – for contracts that are not bound to a wallet, you can specify the which ecosystem’s currency will be used to pay for the contract; in this case the wallet and the public key of the token_ecosystem and the current ecosystem should be the same,
* *[max_sum]* - when calling a contract that is not bound to a wallet, you can specify the maximum amount that can be spent on execution of this contract,
* *[payover]* - for contracts that are not bound to a wallet you can specify additional payment for urgency – how much should be added to fuel_rate when calculating the payment,
* *[vde]* - should be set to true, if the smart contract is requested from a VDE; otherwise, you don't need to specify this parameter.
* *[data]* – parameters sent to the contract,

.. code::
    
    POST
    /api/v2/prepare/mycontract

Response

* *request_id* - the identifier of the request to be transmitted,
* *forsign* - string to be signed,
* *time* - time information, which needs to be sent together with the contract.

Response Example

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "request_id": "5c273816-134e-4a50-89b2-8d2b3d5ba562",
        "time": 423523768,
        "forsign": "......", 
    }

Errors: *E_CONTRACT*    
    
txstatus/{hash}
==============================
**GET**/ Returns a block number or an error of the sent transaction with given hash. If the returned values of *blockid* and *errmsg* are empty, then the transaction hasn't yet been included into a block.

Query

* *hash* - hash of the checked transaction.

.. code:: 
    
    GET
    /api/v2/txstatus/2353467abcd7436ef47438
     
Response

* *blockid* - number of the block in case the transaction has been processed successfully,
* *result* - result of the transaction operation, returned through the **$result** variable,
* *errmsg* - error message in case the transaction was refused.
    
Response Example

.. code:: 
    
    200 (OK)
    Content-Type: application/json
    {
        "blockid": "4235237",
        "result": ""
    }      

content/{menu|page}/{name}
==============================
**POST**/ Returns a JSON representation of the code of the specified page or menu named **{name}**, which is the result of processing by the template engine. The query can have additional parameters, which can be used in the template engine. If the page or menu can't be found, the 404 error is returned.
    
Request

* *menu|page* - *page* or *menu* to recieve the page or menu,
* *name* - the name or menu of the page,
*[lang]* - either lcid or a two-letter language code can be specified to address the corresponding language resources. For example, *en,ru,fr,en-US,en-GB*. If, for example, the *en-US* resource will not be found, the *en* resources will be used instead of the missing *en-US* ones,
* *[app_id]* - application ID. Passed together with lang, because the functions that work with the language in the template engine don’t automatically recognize the AppID. Should be passed as a number,
* *[vde]* - specify *true*, if you recieve data from the page or menu in VDE. Otherwise, you do not need to specify this parameter.

.. code:: 
    
    POST
    /api/v2/content/page/default
    
Response

* *menu* - the menu name for the page when calling *content/page/...*,
* *menutree* - JSON menu tree for the page when calling *content/page/...*,
* *title* - head for the menu *content/menu/...*,
* *tree* - JSON tree of objects.

Response example

.. code:: 
    
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

Errors: *E_NOTFOUND*

content/source/{name}
==============================
**POST**/ Returns a JSON-representation of the **{name}** page code without executing any functions or receiving any data. The returned tree corresponds to the page template and can be used in the visual designer. If the page or the menu are not found, a 404 error is returned.
 
Request
 
* *name* – name of the requested page,
* *[vde]* - *true* should be set to true, if the page or menu is requested from VDE; otherwise, this parameter should not be specified.

Response

.. code:: 
    
    POST
    /api/v2/content/source/default

Response

* *tree* - JSON object tree.
 
Response Example

.. code:: 
    
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
 
Errors: E_NOTFOUND, E_SERVER


content
==============================
**POST**/ Returns a JSON-representation of the page source code from the **template** parameter. If the additional parameter **source** is specified as true or 1, the JSON-representation will be returned without execution of functions and without receiving data. The returned tree corresponds to the sent template and can be used in the visual designer.
 
Request
 
* *template* –page template source code to be processed,
* *[source]* – if set to true or 1, the tree will be returned without execution of functions and without receiving  data.
 
.. code:: 
    
    POST
    /api/v2/content

Response
 
* *tree* – JSON object tree.

Response Example

.. code:: 
    
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

ERRORS: *E_NOTFOUND, E_SERVER*

node/{name}
==============================
**POST** Calls the **{name}** smart contract on behalf of a node. Used for calling smart contracts from VDE contracts though the **HTTPRequest** function. Since in this case the contract can't be signed with an account key, it will be signed with the node's private key. All other parameters are similar to those when sending a contract. The called contract should be bound to an account, because the node's private key account does not have enough funds to execute the contract. If the contract is called from a VDE contract, then the authorization token **$auth_token** should be passed to **HTTPRequest**.
.. code:: js

	var pars, heads map
	heads["Authorization"] = "Bearer " + $auth_token
	pars["vde"] = "false"
	ret = HTTPRequest("http://localhost:7079/api/v2/node/mycontract", "POST", heads, pars)

Reply

.. code:: 
 
    POST
    /api/v2/node/mycontract

Ответ

* *hash* - hex hash of the sent transaction.

Reply example

.. code:: 

    200 (OK)
    Content-Type: application/json
    {
        "hash" : "67afbc435634.....",
    }

maxblockid
==============================
**GET**/ Returns the highest block ID on the current node. 

Request

.. code:: 
 
    GET
    /api/v2/maxblockid

Reply

* *max_block_id* - highest block id on the current node.

Reply Example

.. code:: 

    200 (OK)
    Content-Type: application/json
    {
        "max_block_id" : 341,
    }

Error message: *E_NOTFOUND*

block/{id}
==============================
**GET**/ Returns information on the block with the specified ID.

Request

* *id* - id of the requested block.

.. code:: 
    
    POST
    /api/v2/block/32

Reply

* *hash* - hash of the block.
* *ecosystem_id* - ecosystem id.
* *key_id* - key which signed the block.
* *time* - block generation timestamp.
* *tx_count* - number of transactions in the block.
* *rollbacks_hash* - hash of rollbacks, created by transactions in the block.

Reply Example

Error messages: *E_NOTFOUND*

avatar/{ecosystem}/{member}
==============================
**GET**/ Returns user's avatar (available without login)
 
Request
 
* *ecosystem* - user's ecosystem ID
* *member* - user ID 
 
.. code:: 
    
    GET
    /api/v2/avatar/1/7136200061669836581

Response
 
Header Content-Type with the image type 
Image in the body 
 
Response Example 

.. code:: 
    
    200 (OK)
    Content-Type: image/png  

Errors: *E_NOTFOUND* *E_SERVER*

config/centrifugo
==============================
**GET**/ Returns centrifugo's host and port (available without login)
 
Request

.. code:: 
    
    GET
    /api/v2/config/centrifugo

Response

String http://127.0.0.1:8000 in the response body 
 
Errors: *E_SERVER*
 



