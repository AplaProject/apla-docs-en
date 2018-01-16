################################################################################
Smart-contracts
################################################################################
.. contents::
  :local:
  :depth: 2
  
********************************************************************************
Structure of the contract
********************************************************************************
Data section
==============================
The contract input data, as well as the parameters of the form for the reception of the data are described in the **data** section. 
The data are listed line by line: first, the variable name is specified (only variables, but not arrays are transferred), then the type and the parameters for building of the interface form are indicated optionally through a gap in double quotation marks:

*	*hidden* - hidden element of the form,
*	*optional* - form element without obligatory filling in,
*	*date* - field of the date and time selection,
*	*polymap* - map with the selection of coordinates and areas,
*	*map* - map with the ability to mark the place,
*	*image* - images upload,
*	*text* - entry of the text of HTML-code in the textarea field,
*crypt:Field* - create and encrypt a private key for the destination specified in the *Field* field. If only * crypt * is specified, then the private key will be created for the user who signs the contract,
*	*address* - field for input of the wallet address,
*	*signature:contractname* - a line to display the contractname contract, which requires the signatures (it is discussed in detail in a special description section).

.. code:: js

  contract my {
    data {
        Name string 
        RequestId address
        Photo bytes "image optional"
        Amount money
        Private bytes "crypt:RequestId"
    }
    ...
  }
    
Conditions section
==============================
Validation of the data obtained is performed in the section. The following commands are used to warn of the presence of errors: ``error``, ``warning``, ``info``. In fact, all they generate an error that stops the contract operation, but display different messages in the interface: *critical error*, *warning*, and *informative error*. For instance, 

.. code:: js

  if fuel == 0 {
        error "fuel cannot be zero!"
  }
  if money < limit {
        warning Sprintf("You don't have enough money: %v < %v", money, limit)
  }
  if idexist > 0 {
        info "You have been already registered"
  }
  
Action section
==============================

Variables in the contract
==============================

********************************************************************************
Enclosed contracts
********************************************************************************

********************************************************************************
Contracts with signature
********************************************************************************

Since the language of contracts writing allows performing enclosed contracts, it is possible to fulfill such an enclosed contract without the knowledge of the user who has run the external contract that may lead to the user's signature of transactions unauthorized by it, let's say the transfer of money from its account.

Let's suppose there is a TokenTransfer Contract *TokenTransfer*:

.. code:: js

    contract TokenTransfer {
        data {
          Recipient int
          Amount    money
        }
        ...
    }

If in a contract launched by the user the string ``TokenTransfer("Recipient,Amount", 12345, 100)`` is inscribed, 100 coins will be transferred to the wallet 12345. In such a case the user who signs an external contract will remain not in the know of the transaction. This situation may be excluded if the TokenTransfer contract requires the additional user's signature upon its calling in of contracts. To do this:

1. Adding a field with the name **Signature** with the ``optional`` and ``hidden`` parameters in the *data* section of the *TokenTransfer* contract, which allow not to require the additional signature in the direct calling of the contract, since there will be the signature in the **Signature** field so far.

.. code:: js

    contract TokenTransfer {
        data {
          Recipient int
          Amount    money
          Signature string "optional hidden"
        }
        ...
    }

2. Adding in the *Signatures* table (on the page **Signatures** of Apla client) the entry containing:

•	*TokenTransfer* contract name,
•	field names whose values will be displayed to the user, and their text description,
•	text to be displayed upon confirmation.
  
In the current example it will be enough specifying two fields **Receipient** and **Amount**:

* **Title**: Are you agree to send money this recipient?
* **Parameter**: Receipient Text: Wallet ID
* **Parameter**: Amount Text: Amount (qEGS)

Now, if inserting the *TokenTransfer(“Recipient, Amount”, 12345, 100)* contract calling in, the system error ``“Signature is not defined”`` will be displayed. If the contract is called in as follow: ``TokenTransfer("Recipient, Amount, Signature", 12345, 100, "xxx...xxxxx")``, the system error will occur upon signature verification. Upon the contract calling in, the following information is verified: *time of the initial transaction, user ID, the value of the fields specified in the signatures table*, and it is impossible to forge the signature.

In order for the user to see the money transfer confirmation upon the *TokenTransfer* contract calling in, it is necessary to add a field with an arbitrary name and the type ``string``, and with the optional parameter ``signature:contractname``. Upon calling in of the enclosed *TokenTransfer* contract, you just need to forward this parameter. It should also be borne in mind that the parameters for the secured contract calling in must also be described in the ``data`` section of the external contract (they may be hidden, but they will still be displayed upon confirmation). For instance,

.. code:: js

    contract MyTest {
      data {
          Recipient int "hidden"
          Amount  money
          Signature string "signature:TokenTransfer"
      }
      func action {
          TokenTransfer("Recipient,Amount,Signature",$Recipient,$Amount,$Signature)
      }
    }

When sending a *MyTest* contract, the additional confirmation of the money transfer to the indicated account will be requested from user. If other values, such as ``TokenTransfer(“Recipient,Amount,Signature”,$Recipient, $Amount+10, $Signature)``, are listed in the enclosed contract, the invalid signature error will occur.

Simvolio Contracts Language
Contracts in Apla are written using an original (developed by the Apla team) Turing-complete script language called Simvolio, with compilation into bytecode. The language includes a set of functions, operators and constructions that can be used for implementation of data processing algorithms and operations with the database. The Simvolio language provides for:
•	Declaration of variables with different data types, as well as simple and associative arrays: var, array, map
•	Use of the if conditional statement and the while loop structure
•	Retrieval of values from the database and recording data to database DBFind, DBInsert, DBUpdate
•	Work with contracts CallContract, ContractAccess, etc.
•	Conversion of variables HexToBytes, Int, Str, etc.
•	Operations with strings Size, Replace, Substr

********************************************************************************
Contract Editor
********************************************************************************
Contracts can be created and edited in a special editor which is a part of the Molis software client. Each new contract has a typical structure created in it by default with three sections: ``data, conditions, action``. The contracts editor helps to:

- Write the contract code (highlighting key words of the Simvolio language,
-	Format the contract source code,
-	Bind the contract to an account, from which the payment for its execution will be charged, 
-	Define permissions to edit the contract (typically, by specifying the contract name with the permissions stipulated in a special function ContractConditions or by way of direct indication of access conditions in the Change conditions field),
-	View the history of changes made to the contract with the option to restore previous versions.

********************************************************************************
Simvolio Contracts Language
********************************************************************************
Contracts in Apla are written using an original (developed by the Apla team) Turing-complete script language called Simvolio, with compilation into bytecode. The language includes a set of functions, operators and constructions that can be used for implementation of data processing algorithms and operations with the database. The Simvolio language provides for:
-	Declaration of variables with different data types, as well as simple and associative arrays: var, array, map,
-	Use of the if conditional statement and the while loop structure,
-	Retrieval of values from the database and recording data to database DBFind, DBInsert, DBUpdate,
-	Work with contracts CallContract, ContractAccess, etc.,
-	Conversion of variables HexToBytes, Int, Str, etc.,
-	Operations with strings Size, Replace, Substr.

Basic elements and constructions of the language
==============================
Value Types and Variables
------------------------------

Massives
------------------------------

Functions
------------------------------

Predefined values
------------------------------
The following variables are available when executing a contract. 

* ``$key_id`` - a numerical identifier (int64) of the account that signed the transaction,
* ``$ecosystem_id`` - identifier of the ecosystem where the transaction was created, 
* ``$type`` identifier of an external contract from where the current contract was called, 
* ``$time`` - time specified in the transaction in Unix format, 
* ``$block`` - block number in which this transaction is sealed, 
* ``$block_time`` - time specified in the block, 
* ``$block_key_id`` - numeric identifier (int64) of the node that signed the block. 

.. code:: js

	var pars, heads map
	heads["Authorization"] = "Bearer " + $auth_token
	pars["vde"] = "false"
	ret = HTTPRequest("http://localhost:7079/api/v2/node/mycontract", "POST", heads, pars)

It should be kept in mind that these variables are available not only in the functions of the contract but also in other functions and expressions, for example, in conditions that are specified for contracts, pages and other objects. In this case, *$time*, *$block* variables related to the block and others are equal to 0.

The value that needs to be returned from the contract should be assigned to a predefined variable ``$result``.

Retrieving values from the database
==============================

DBFind(table string) [.Columns(columns string)] [.Where(where string, params ...)] [.WhereId(id int)] [.Order(order string)] [.Limit(limit int)] [.Offset(offset int)] [.Ecosystem(ecosystemid int)] array
==========================
The Function receives data from a database table in accordance with the request specified. Returned is an *array* comprised of *map* associative arrays.

* *table* - table name.
* *сolumns* - list of returned columns. If not specified, all columns will be returned, 
* *Where* - search condition. For instance, ``.Where("name = 'John'")`` or ``.Where("name = ?", "John")``,
* *id* - search by identifier. For example, *.WhereId(1)*,
* *order* - a field, which will be used for sorting. By default, values are sorted by *id*,
* *limit* - number of returned values (default = 25, maximum = 250),
* *offset* - returned values offset,
* *ecosystemid* - ecosystem ID. By default, values are taken from the table in the current ecosystem.

.. code:: js

   var i int
   ret = DBFind("contracts").Columns("id,value").Where("id> ? and id < ?", 3, 8).Order("id")
   while i < Len(ret) {
       var vals map
       vals = ret[0]
       Println(vals["value"])
       i = i + 1
   }
   
   var ret string
   ret = DBFind("contracts").Columns("id,value").WhereId(10).One("value")
   if ret != nil { 
   	Println(ret) 
   }

DBRow(table string) [.Columns(columns string)] [.Where(where string, params ...)] [.WhereId(id int)] [.Order(order string)] [.Ecosystem(ecosystemid int)] map
------------------------------
The function returns an associative array *map* with data obtained from a database table in accordance with the specified query.

 * *table* - table name,
 * *columns* - a list of columns to be returned. If not specified, all columns will be returned, 
 * *Where* - search parameters; for example, ``.Where("name = 'John'")`` or ``.Where("name = ?", "John")``,
 * *id* - identifier of the string to be returned.  For instance, ``.WhereId(1)``,
 * *order* - a field to use for sorting; by default, information is sorted by *id* field,
 * *ecosystemid* - ecosystem identifier; by default it is the current ecosystem id.
 	
.. code:: js

   var ret map
   ret = DBRow("contracts").Columns("id,value").Where("id = ?", 1)
   Println(map)
    
EcosysParam(name string) string
------------------------------
The function returns the value of a specified parameter from the ecosystem settings (*parameters*).

* *name* - name of the received parameter.

.. code:: js

    Println( EcosysParam("gov_account"))

LangRes(label string, lang string) string
------------------------------
This function returns a language resource with name label for language lang, specified as a two-character code, for instance, *en, fr, ru*; if there is no language resource for a selected language, the result will be returned in English.

* *label* - language resource name,
* *lang* - two-character language code.

.. code:: js

    warning LangRes("confirm", $Lang)
    error LangRes("problems", "de")
                     	
Changing values in tables
==============================

DBInsert(table string, params string, val ...) int
------------------------------
The function adds a record to a specified table and returns the **id** of the inserted record.

* *tblname*  – name of the table in the database,
* *params* - list of comma-separated names of columns, where the values listed in **val** will be written,
* *val* - list of comma-separated values for the columns listed in **params**; values can be a string or a number.

.. code:: js

    DBInsert("mytable", "name,amount", "John Dow", 100)

DBUpdate(tblname string, id int, params string, val...)
------------------------------
The function changes the column values in the table in the record with a specified **id**.

* *tblname*  – name of the table in the database,
* *id* - identifier **id** of the changeable record,
* *params* - list of comma-separated names of the columns to be changed,
* *val* - list of values for a specified columns listed in **params**; can either be a string or a number.

.. code:: js

    DBUpdate("mytable", myid, "name,amount", "John Dow", 100)

DBUpdateExt(tblname string, column string, value (int|string), params string, val ...)
------------------------------
The function updates columns in a record whose column has a specified value. The table should have an index for a specified column.

* *tblname*  – name of the table in the database,
* *column*  - name of the column by which the record will be searched for,
* *value* - value for searching a record in a column,
* *params* - list of comma-separated names of columns, where the values specified in **val** will be written,
* *val* - list of values for recording in the columns listed in **params**; can either be a string or a number.

.. code:: js

    DBUpdateExt("mytable", "address", addr, "name,amount", "John Dow", 100)
    
Array operations
==============================

Operations with contracts and conditions
==============================

CallContract(name string, params map)
------------------------------
The function calls a contract by its name. All the parameters specified in the section data of the contract should be listed in the transmitted array. The function returns the value that was assigned to **$ result**  variable in the contract.

* *name*  - name of the contract being called,
* *params* - an associative array with input data for the contract.

.. code:: js

    var par map
    par["Name"] = "My Name"
    CallContract("MyContract", par)

ContractAccess(name string, [name string]) bool
------------------------------
The function checks whether the name of the executed contract matches with one of the names listed in the parameters. Typically used to control access of contracts to tables. The function is specified in the *Permissions* fields when editing table columns or in the *Insert* and *New Column* fields in the *Table permission* section.

* *name* – contract name.

.. code:: js

    ContractAccess("MyContract")  
    ContractAccess("MyContract","SimpleContract") 
    
ContractConditions(name string, [name string]) bool
------------------------------
The function calls the **conditions** section from contracts with specified names. For such contracts, the *data* block must be empty. If the conditions *conditions* is executed without errors, then *true* is returned. If an error is generated during execution, the parent contract will also end with this error. This function is usually used to control access of contracts to tables and can be called in the *Permissions* fields when editing system table.

* *name* – contract name.

.. code:: js

    ContractConditions("MainCondition")  

EvalCondition(tablename string, name string, condfield string) 
------------------------------
Function takes from the *tablename* table the value of the *condfield* field from the record with the *’name’* field, which is equal to the *name* parameter and checks if the condition from the field *condfield* is made. 

* *tablename* - name of the table.
* *name* - value for searching by the field 'name'
* *condfield* - the name of the field where the condition to be checked is stored.

.. code:: js

    EvalCondition(`menu`, $Name, `condition`)  

ValidateCondition(condition string, state int) 
------------------------------
The function tries to compile the condition specified in the *condition* parameter. If a mistake occurs during the compilation process, the mistake will be generated and the calling contract will complete is’s job. This function is designed to check the correctness of the conditions when they change.

* *condition* - verifiable condition.
* *state* - identifier of the state. Specifie 0 if checking for global conditions

.. code:: js

    ValidateCondition(`ContractAccess("@0MyContract")`, 0)
    
Operations with account addresses
==============================

AddressToId(address string) int
------------------------------
Function returns the the identification number of the citizen by the string value of the address of his wallet. If the wrong adress is specified, then 0 returns. 

* *address* - the wallet adress in the format XXXX-...-XXXX or in the form of number.

.. code:: js

    wallet = AddressToId($Recipient)
    
IdToAddress(id int) string
------------------------------
Returns the address of a wallet based on its ID number. If a wrong ID is specified, returned is 'invalid'.

* *id* - ID, numerical.

.. code:: js

    $address = IdToAddress($id)
    

PubToID(hexkey string) int
------------------------------
The function returns the wallet address by the public key in hexadecimal encoding.

* *hexkey* - public key in hexadecimal form.

.. code:: js

    var wallet int
    wallet = PubToID("fa5e78.....34abd6")


Operations with values of variables
==============================

Float(val int|string) float
------------------------------
The function converts an integer *int* or *string* to a floating-point number.

* *val* - an integer or string.

.. code:: js

    val = Float("567.989") + Float(232)

HexToBytes(hexdata string) bytes
------------------------------
The function converts a string with hexadecimal encoding to a *bytes* value (sequence of bytes).

* *hexdata* – a string containing a hexadecimal notation.

.. code:: js

    var val bytes
    val = HexToBytes("34fe4501a4d80094")

Int(val string) int
------------------------------
The function converts a string value to an integer.

* *val*  – a string containing a number.

.. code:: js

    mystr = "-37763499007332"
    val = Int(mystr)

Sha256(val string) string
------------------------------
The function returns **SHA256** hash of a specified string.

* *val* - incoming line for which the **Sha256** hash should be calculated.

.. code:: js

    var sha string
    sha = Sha256("Test message")

Str(val int|float) string
------------------------------
The function converts a numeric *int* or *float* value to a string.

* *val* - an integer or a floating-point number.

.. code:: js

    myfloat = 5.678
    val = Str(myfloat)

UpdateLang(name string, trans string)
------------------------------
Function updates the language source in the memory. Is used in the transactions that change language sources.

* *name* - name of the language source,
* *trans* - source with translations.

.. code:: js

    UpdateLang($Name, $Trans)


Operations with string values
==============================

HasPrefix(s string, prefix string) bool
------------------------------
Function returns true, if the string bigins from the specified substring *prefix*.

* *s* - checked string,
* *prefix* - checked prefix for this string.

.. code:: js

    if HasPrefix($Name, `my`) {
    ...
    }

Contains(s string, substr string) bool
------------------------------
Returnes true if the string *s* containts the substring *substr*.

* *s* - checked string,
* *substr* - which is searched in the specified line.

.. code:: js

    if Contains($Name, `my`) {
    ...
    }    

Replace(s string, old string, new string) string
------------------------------
Function replaces in the *s* string all cccurrences of the *old* string to *new* string and returnes the result.  

* *s* - source string,
* *old* - changed string,
* *new* - new string.

.. code:: js

    s = Replace($Name, `me`, `you`)
    
Size(val string) int
------------------------------
The function returns the size of the specified string.

* *val* - the string for which we have to calculate the size.

.. code:: js

    var len int
    len = Size($Name) 
 
Sprintf(pattern string, val ...) string
------------------------------
The function forms a string based on specified template and parameters, you can use *%d (number), %s (string), %f (float), %v* (for any types).

* *pattern*  - a template for forming a string.

.. code:: js

    out = Sprintf("%s=%d", mypar, 6448)

Substr(s string, offset int, length int) string
------------------------------
Function returns the substring from the specified string starting from the offset *offset* (calculating from the 0) and with length *length*. In case of not correct offsets or length the empty column is returned. If the sum of offset and *length* is more than string size, then the substring will be returned from the offset to the end of the string.

* *val* - string,
* *offset* - offset of substring,
* *length* - size of substring.

.. code:: js

    var s string
    s = Substr($Name, 1, 10)

Operations with system parameters
==============================

SysParamString(name string) string
------------------------------
The function returns the value of the specified system parameter.

* *name* - parameter name;

.. code:: js

    url = SysParamString(`blockchain_url`)

SysParamInt(name string) int
------------------------------
The function returns the value of the specified system parameter in the form of a number.

* *name* - parameter name.

.. code:: js

    maxcol = SysParam(`max_columns`)

DBUpdateSysParam(name, value, conditions string)
------------------------------
The function updates the value and the condition of the system parameter. If you do not need to change the value or condition, then specify an empty string in the corresponding parameter.

* *name* - parameter name,
* *value* - new value of the parameter,
* *conditions* - new condition for changing the parameter.

.. code:: js

    DBUpdateSysParam(`fuel_rate`, `400000000000`, ``)
    

Date/time operations in PostgreSQL queries
==============================

Functions do not allow direct possibilities to select, update, etc.. but they allow you to use the capabilities and functions of PostgreSQL when you get values and a description of the where conditions  in the samples. This includes, among other things, the functions for working with dates and time. For example, you need to compare the column *date_column* and the current time. If  *date_column* has the  type timestamp, then the expression will be the following ``date_column> now ()``.And if *date_column* stores time in Unix format as a number, then the expression will be  ``to_timestamp (date_column)> now ()``.

.. code:: js

    to_timestamp(date_column) > now()
    date_initial < now() - 30 * interval '1 day'
    
Consider the situation when we have a value in Unix format and we need to write it in a field of type *timestamp *. In this case, when listing fields, before the name of this column you need to specify **timestamp**.

.. code:: js

   DBInsert("mytable", "name,timestamp mytime", "John Dow", 146724678424 )

If you have a string value of time and you need to write it in a field with the type *timestamp*, in this case, **timestamp** must be specified before the value itself.

.. code:: js

   DBInsert("mytable", "name,mytime", "John Dow", "timestamp 2017-05-20 00:00:00" )
   var date string
   date = "2017-05-20 00:00:00"
   DBInsert("mytable", "name,mytime", "John Dow", "timestamp " + date )
   DBInsert("mytable", "name,mytime", "John Dow", "timestamp " + $txtime )


Functions for VDE
==============================

************************************************
System contracts
************************************************

