################################################################################
Smart-contracts
################################################################################
.. contents::
  :local:
  :depth: 2

A smart contract (hereinafter, "contract") is a basic element of applications, which performs a single action (typically, makes a record in a database table), initiated from the user interface by a user or by another contract. All operations with data in applications are formed as a system of contracts, interacting with each other through database tables or by call functions in a contract body.

Contracts are written using an original (developed by the team of platform developers) Turing-complete script language called Simvolio, with compilation into bytecode. The language includes a set of functions, operators and constructions that can be used for implementation of data processing algorithms and operations with the database. 

Contracts can be edited, but only if editing was not forbidden by way of putting false in the contract editing rights. Operations with data in the blockchain are performed by the most up-to-date (current) version of the contract. The complete history of changes made to contracts is stored in the blockchain and available from the software client.

********************************************************************************
Structure of the contract
********************************************************************************
Contracts are declared with the contract keyword, followed by the new contract's name. The contract's body should be enclosed in curly brackets. Every contract consists of three sections:

1. **data** - declares the input data (names and types of variables),
2. **conditions** - verifies the correctness of input data,
3. **action** - includes the actions performed by the contract.

Contract structure:

.. code:: js

  contract MyContract {
      data {
          FromId address
          ToId   address
          Amount money
      }
      func conditions {
          ...
      }
      func action {
      }
  }
  

Data section
==============================
The contract input data, as well as the parameters of the form for the reception of the data are described in the **data** section. 
The data are listed line by line: first, the variable name is specified (only variables, but not arrays are transferred), then the type and the parameters for building of the interface form are indicated optionally through a gap in double quotation marks:

* *hidden* - hidden element of the form,
* *optional* - form element without obligatory filling in,
* *date* - field of the date and time selection,
* *polymap* - map with the selection of coordinates and areas,
* *map* - map with the ability to mark the place,
* *image* - images upload,
* *text* - entry of the text of HTML-code in the textarea field,
* *crypt:Field* - create and encrypt a private key for the destination specified in the *Field* field. If only ``crypt`` is specified, then the private key will be created for the user who signs the contract,
* *address* - field for input of the account address,
* *signature:contractname* - a line to display the contractname contract, which requires the signatures (it is discussed in detail in a special description section).

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
The action section contains the contract's main program code that retrieves additional data and records the resulting values to database tables. For example,

.. code:: js

	action {
		DBUpdate("keys", $key_id,"-amount", $amount)
		DBUpdate("keys", $recipient,"+amount,pub", $amount, $Pub)
	}


Variables in the contract
==============================
Contract input data, declared in the data section, is passed to other sections though variables with the ``$`` sign followed by data names. The ``$`` sign can be used to declare additional variables; such variables will be considered global for this contract and all nested contracts.

A contract can access predefined variables that contain data about the transaction, from which this contract was called.

* ``$time` – transaction time, int,
* ``$ecosystem_id`` – ecosystem ID, int,
* ``$block`` – number of the block, in which this transaction is included, int,
* ``$key_id`` – ID of the account that signed the transaction; the value will be zero for VDE contracts,
* ``$wallet_block`` – address of the node that formed the block, in which this transaction is included,
* ``$block_time`` – time, when the block with the transaction containing the current contract was formed.

Predefined variables are accessible not only in contracts, but also in Permissions fields, (where conditions for access to application elements are defined), where they are used in construction of logical expressions. When used in Permissions fields, variables related to block formation (``$time``, ``$block``, etc.) always equal zero.

Predefined variable $result is used to return a value from a nested contract.

.. code:: js

  contract my {
    data {
        Name string 
        Amount money
    }
    func conditions {
        if $Amount <= 0 {
           error "Amount cannot be 0"
        }
        $ownerId = 1232
    }
    func action {
        DBUpdate("mytable", $ownerId, "name,amount", $Name, $Amount - 10 )
        DBUpdate("mytable2", $citizen, "amount", 10 )
    }
  }
  
********************************************************************************
Nested Contracts 
********************************************************************************
A nested contract can be called from the conditions and action sections of the enclosing contract. A nested contract can be called directly with parameters specified in parenthesis after its name (NameContract(Params)), or using the CallContract function, for which the contract name is passed using a string variable.

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

If in a contract launched by the user the string ``TokenTransfer("Recipient,Amount", 12345, 100)`` is inscribed, 100 coins will be transferred to the account 12345. In such a case the user who signs an external contract will remain not in the know of the transaction. This situation may be excluded if the TokenTransfer contract requires the additional user's signature upon its calling in of contracts. To do this:

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

2. Adding in the *Signatures* table (on the page *Signatures* of platform client) the entry containing:

•	*TokenTransfer* contract name,
•	field names whose values will be displayed to the user, and their text description,
•	text to be displayed upon confirmation.
  
In the current example it will be enough specifying two fields **Receipient** and **Amount**:

* **Title**: Are you agree to send money this recipient?
* **Parameter**: Receipient Text: Account ID
* **Parameter**: Amount Text: Amount (qEGS)

Now, if inserting the ``TokenTransfer(“Recipient, Amount”, 12345, 100)`` contract calling in, the system error ``“Signature is not defined”`` will be displayed. If the contract is called in as follow: ``TokenTransfer("Recipient, Amount, Signature", 12345, 100, "xxx...xxxxx")``, the system error will occur upon signature verification. Upon the contract calling in, the following information is verified: *time of the initial transaction, user ID, the value of the fields specified in the signatures table*, and it is impossible to forge the signature.

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

********************************************************************************
Contract Editor
********************************************************************************
Contracts can be created and edited in a special editor which is a part of the Molis software client. Each new contract has a typical structure created in it by default with three sections: ``data, conditions, action``. The contracts editor helps to:

- Write the contract code (highlighting key words of the Simvolio language,
- Format the contract source code,
- Bind the contract to an account, from which the payment for its execution will be charged, 
- Define permissions to edit the contract (typically, by specifying the contract name with the permissions stipulated in a special function ContractConditions or by way of direct indication of access conditions in the Change conditions field),
- View the history of changes made to the contract with the option to restore previous versions.

********************************************************************************
Simvolio Contracts Language
********************************************************************************
Contracts in the platform are written using an original (developed by the platform team) Turing-complete script language called Simvolio, with compilation into bytecode. The language includes a set of functions, operators and constructions that can be used for implementation of data processing algorithms and operations with the database. The Simvolio language provides for:

- Declaration of variables with different data types, as well as simple and associative arrays: var, array, map,
- Use of the ``if`` conditional statement and the ``while`` loop structure,
- Retrieval of values from the database and recording data to database ``DBFind, DBInsert, DBUpdate``,
- Work with contracts,
- Conversion of variables,
- Operations with strings.

Basic elements and constructions of the language
==============================
Data Types and Variables
------------------------------
Data type should be defined for every variable. In obvious cases, data types are converted automatically. The following data types can be used:

* ``bool`` - Boolean, can be true or false,
* ``bytes`` - a sequence of bytes,
* ``int`` - a 64-bit integer,
* ``address`` - a 64-bit unsigned integer,
* ``array`` - an array of values of arbitrary types,
* ``map`` - an associative array of values of arbitrary data types with string keys,
* ``money`` - an integer of the big integer type; values are stored in the database without decimal points, which are added when displaying values in the user interface in accordance with the currency configuration settings,
* ``float`` - a 64-bit number with a floating point,
* ``string`` - a string; should be defined in double quotes or back quotes: "This is a string" or `This is a string`.

All identifiers, including the names of variables, functions, contracts, etc. are case sensitive (MyFunc and myFunc are different names). 

Variables are declared with the **var** keyword, followed by names and types of variables. Variables declared inside curly brackets should be used within the same pair of curly brackets. When declared, variables have default values: for *bool* type it is *false*, for all numeric types – zero values, for strings – empty strings. Examples of variables declaration: 

.. code:: js

  func myfunc( val int) int {
      var mystr1 mystr2 string, mypar int
      var checked bool
      ...
      if checked {
           var temp int
           ...
      }
  }

Arrays
------------------------------
The language supports two array types: 

* ``array`` - a simple array with numeric index starting from zero, 
* ``map`` - an associative array with string keys.

When assigning and и retrieving array elements, index should be put in square brackets.

.. code:: js

    var myarr array
    var mymap map
    var s string
    
    myarr[0] = 100
    myarr[1] = "This is a line"
    mymap["value"] = 777
    mymap["param"] = "Parameter"

    s = Sprintf("%v, %v, %v", myarr[0] + mymap["value"], myarr[1], mymap["param"])
    // s = 877, This is a line, Parameter 

If and While Statements
------------------------------
The contract language supports the standard **if** conditional statement and the **while** loop, which can be used in functions and contracts. These statements can be nested in each other. 

A keyword should be followed by a conditional statement. If the conditional statement returns a number, then it is considered as *false* when its value = zero. For example, *val == 0* is equivalent to *!val*, and *val != 0* is the same as just *val*. The **if** statement can have an **else** block, which executes in case the **if** conditional statement is false. The following comparison operators can be used in conditional statements: ``<, >, >=, <=, ==, !=``, as well as ``||`` (OR) and ``&&`` (AND).

.. code:: js

    if val > 10 || id != $citizen {
      ...
    } else {
      ...
    }

The **while** statement is intended for implementation of loops. A **while** block will be executed while its condition is true. The **break** operator is used to end a loop inside a block. To start a loop from the beginning, the **continue** operator should be used.

.. code:: js

  while true {
      if i > 100 {
         break
      }
      ...
      if i == 50 {
         continue
      }
      ...
  }

Apart from conditional statements, the language supports standard arithmetic operations: ``+,-,*,/``
Variables of **string** and **bytes** types can be used as a condition. In this case, the condition will be true when the length of the string (bytes) is greater than zero, and false for an empty string.

Functions
------------------------------
Functions of the contracts language perform operations with data received in the data section of a contract: reading and writing database values, converting value types, and establishing connections between contracts.

Functions are declared with the **func** keyword, followed by the function name and a list of parameters passed to it (with their types), all enclosed in curly brackets and separated by commas. After the closing curly bracket the data type of the value returned by the function should be stated. The function body should be enclosed in curly brackets. If a function does not have parameters, then the curly brackets are not necessary. To return a value from a function, the ``return`` keyword is used.

.. code:: js

  func myfunc(left int, right int) int {
      return left*right + left - right
  }
  func test int {
      return myfunc(10, 30) + myfunc(20, 50)
  }
  func ooops {
      error "Ooops..."
  }
  
Functions don't return errors, because all error checks are carried out automatically. When an error is generated in any function, the contract stops its operation and displays a window with the error description.

An undefined number of parameters can be passed to a function. To do this, put **...** instead of the type of the last parameter. In this case, the data type of the last parameter will be *array*, and it will contain all, starting from this parameter, variables that were passed with the call. Variables of any type can be passed, but you should take care of possible conflicts related to data type mismatch.

.. code:: js

  func sum(out string, values ...) {
      var i, res int
      
      while i < Len(values) {
         res = res + values[i]
         i = i + 1
      }
      Println(out, res)
  }

  func main() {
     sum("Sum:", 10, 20, 30, 40)
  }
  
Let's consider a situation, where a function has many parameters, but we need only some of them when calling it. In this case, optional parameters can be declared in the following way: ``func myfunc(name string).Param1(param string).Param2(param2 int) {...}``. You can specify only the parameters you need with the call in arbitrary order: ``myfunc("name").Param2(100)``. In the function body you can address these variables as usual. If an extended parameter is not specified with the call, it will have the default value, for example, an empty string for a string and zero for a number. It should be noted, that you can specify several extended parameters and use ``...``: ``func DBFind(table string).Where(request string, params ...)`` and call ``DBFind("mytable").Where("id > ? and type = ?", myid, 2)``

.. code:: js
 
    func DBFind(table string).Columns(columns string).Where(format string, tail ...)
             .Limit(limit int).Offset(offset int) string  {
       ...
    }
     
    func names() string {
       ...
       return DBFind("table").Columns("name").Where("id=?", 100).Limit(1)
    }

Predefined values
------------------------------
The following variables are available when executing a contract. 

* ``$key_id`` - a numerical identifier (int64) of the account that signed the transaction,
* ``$ecosystem_id`` - identifier of the ecosystem where the transaction was created, 
* ``$type`` identifier of an external contract from where the current contract was called, 
* ``$time`` - time specified in the transaction in Unix format, 
* ``$block`` - block number in which this transaction is sealed, 
* ``$block_time`` - time specified in the block, 
* ``$block_key_id`` - numeric identifier (int64) of the node that signed the block,
* ``$auth_token`` is the authorization token, which can be used in VDE contracts, for example, when calling contracts though API with the ``HTTPRequest`` function.

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
------------------------------
The Function receives data from a database table in accordance with the request specified. Returned is an *array* comprised of *map* associative arrays.

* *table* - table name,
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
The function returns the value of a specified parameter from the ecosystem settings (*parameters* table).

* *name* - name of the received parameter,
* *num* - sequence number of the parameter.

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
The function adds a record to a specified *table* and returns the **id** of the inserted record.

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
Join(in array, sep string) string
------------------------------
This function merges the elements of the *in* array into a string with the specified *sep* separator.

* *in* - is the name of the *array* type array, the elements of which you want to merge,
* *sep* - is a separator string.

.. code:: js

    var val string, myarr array
    myarr[0] = "first"
    myarr[1] = 10
    val = Join(myarr, ",")

Split(in string, sep string) array
------------------------------
This function splits the *in* string into elements using *sep* as a separator, and puts them into an array.

* *in* is the initial string,
* *sep* is the separator string.

.. code:: js

    var myarr array
    myarr = Split("first,second,third", ",")

Len(val array) int
------------------------------
This function returns the number of elements in the specified array.

* *val* - an array of the *array* type.

.. code:: js

    if Len(mylist) == 0 {
      ...
    }

Row(list array) map
------------------------------
This function returns the first *map* associative array from the *list* array. If the *list* is empty, then the result will be an empty *map*. This function is mostly used with the DBFind function. The *list* parameter should not be specified in this case. 

* *list* - a map array, returned by the **DBFind** function.

.. code:: js

   var ret map
   ret = DBFind("contracts").Columns("id,value").WhereId(10).Row()
   Println(ret)

One(list array, column string) string
------------------------------
The function returns the value of the *column* key from the first associative array in the *list* array. If the *list* list is empty, then nil is returned. This function is mostly used with the DBFind function. The *list* parameter should not be specified in this case. 

* *list* - a map array, returned by the **DBFind** function,
* *column* - name of the returned key.

.. code:: js

   var ret string
   ret = DBFind("contracts").Columns("id,value").WhereId(10).One("value")
   if ret != nil {
      Println(ret)
   }

Operations with contracts and conditions
==============================
CallContract(name string, params map)
------------------------------
The function calls a contract by its name. All the parameters specified in the section *data* of the contract should be listed in the transmitted array. The function returns the value that was assigned to **$result**  variable in the contract.

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

* *tablename* - name of the table,
* *name* - value for searching by the field 'name',
* *condfield* - the name of the field where the condition to be checked is stored.

.. code:: js

    EvalCondition(`menu`, $Name, `condition`)  

ValidateCondition(condition string, state int) 
------------------------------
The function tries to compile the condition specified in the *condition* parameter. If a mistake occurs during the compilation process, the mistake will be generated and the calling contract will complete is’s job. This function is designed to check the correctness of the conditions when they change.

* *condition* - verifiable condition,
* *state* - identifier of the state. Specifie 0 if checking for global conditions.

.. code:: js

    ValidateCondition(`ContractAccess("@1MyContract")`, 1)  
    

Operations with account addresses
==============================
AddressToId(address string) int
------------------------------
Function returns the the identification number of the citizen by the string value of the address of his account. If the wrong adress is specified, then 0 returns. 

* *address* - the account adress in the format XXXX-...-XXXX or in the form of number.

.. code:: js

    wallet = AddressToId($Recipient)
    
IdToAddress(id int) string
------------------------------
Returns the address of a account based on its ID number. If a wrong ID is specified, returned is 'invalid'.

* *id* - ID, numerical.

.. code:: js

    $address = IdToAddress($id)
    

PubToID(hexkey string) int
------------------------------
The function returns the account address by the public key in hexadecimal encoding.

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
       
Random(min int, max int) int
------------------------------
This function returns a random number in the range between min and max (min <= result < max). Both min and max should be positive numbers.

* *min* is the minimum value for the random number,
* *max* - the random number will be smaller than this number.

.. code:: js

    i = Random(10,5000)
   
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
The function forms a string based on specified template and parameters, you can use ``%d`` (number), ``%s`` (string), ``%f`` (float), ``%v`` (for any types).

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

* *name* - parameter name.

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
Functions do not allow direct possibilities to select, update, etc.. but they allow you to use the capabilities and functions of PostgreSQL when you get values and a description of the where conditions  in the samples. This includes, among other things, the functions for working with dates and time. For example, you need to compare the column *date_column* and the current time. If  *date_column* has the  type timestamp, then the expression will be the following ``date_column> now ()``. And if *date_column* stores time in Unix format as a number, then the expression will be ``to_timestamp (date_column)> now ()``.

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
The following functions can be used only in Virtual Dedicated Ecosystems (VDE) contracts.

HTTPRequest(url string, method string, heads map, pars map) string
------------------------------
This function sends an HTTP request to a specified address.

* *url* - address, to which the request will be sent,
* *method* - request method – GET or POST,
* *heads* - a data array for header formation,
* *pars* - parameters.

.. code:: js

	var ret string 
	var pars, heads, json map
	heads["Authorization"] = "Bearer " + $auth_token
	pars["vde"] = "true"
	ret = HTTPRequest("http://localhost:7079/api/v2/content/page/default_page", "POST", heads, pars)
	json = JSONToMap(ret)

HTTPPostJSON(url string, heads map, pars string) string
------------------------------
This function is similar to the *HTTPRequest* function, but it sends a *POST* request and parameters are passed in one string.

* *url* - address, to which the request will be sent,
* *heads* - a data array for header formation,
* *pars* - parameters as a json string.

.. code:: js

	var ret string 
	var heads, json map
	heads["Authorization"] = "Bearer " + $auth_token
	ret = HTTPPostJSON("http://localhost:7079/api/v2/content/page/default_page", heads, `{"vde":"true"}`)
	json = JSONToMap(ret)

************************************************
System Contracts
************************************************
System contracts are created by default during product installation. All of these contracts are created in the first ecosystem, that's why you need to specify their full name to call them from other ecosystems, for instance, ``@1NewContract``.

List of System Contracts
==============================
NewEcosystem
------------------------------
This contract creates a new ecosystem. To get an identifier of the newly created ecosystem, take the *result* field, which will return in txstatus. Parameters:
   
* *Name string "optional"* - name for the ecosystem. This parameter can be set and/or chanted later.

MoneyTransfer
------------------------------
This contract transfers money from the current account in the current ecosystem to a specified account. Parameters:

* *Recipient string* - recipient's account in any format – a number or ``XXXX-....-XXXX``,
* *Amount    string* - transaction amount in qAPL,
* *Comment   string "optional"* - comments.

NewContract
------------------------------
This contract creates a new contract in the current ecosystem. Parameters:

* *Value string* - text of the contract or contracts,
* *Conditions string* - contract change conditions,
* *Wallet string "optional"* - identifier of user's id where contract should be tied,
* *TokenEcosystem int "optional"* - identifier of the ecosystem, which currency will be used for transactions when the contract is activated.

EditContract
------------------------------
Editing the contract in the current ecosystem.

Parameters
      
* *Id int* - ID of the contract to be edited,
* *Value string* - text of the contract or contracts,
* *Conditions string* - rights for contract change.

ActivateContract
------------------------------
Binding of a contract to the account in the current ecosystem. Contracts can be tied only from the account, which was specified when the contract was created. After the contract is tied, this account will pay for execution of this contract.

Parameters
      
* *Id int* - ID of the contract to activate.

DeactivateContract
------------------------------
Unbinds a contract from an account in the current ecosystem. Only the account which the contract is currently bound to can unbind it. After the contract is unbound, its execution will be paid by a user that executes it.
 
 Parameters
 
* *Id int* - identifier of the tied contract.

NewParameter
------------------------------
This contract adds a new parameter to the current ecosystem. 

Parameters

* *Name string* - parameter name,
* *Value string* - parameter value,
* *Conditions string - rights for parameter change.

EditParameter
------------------------------
This contract changes an existing parameter in the current ecosystem.

Parameters

* *Name string* - name of the parameter to be changed,
* *Value string* - new value,
* *Conditions string* - new condition for parameter change.

NewMenu
------------------------------
This contract adds a new menu in the current ecosystem.

Parameters

* *Name string* - menu name,
* *Value string* - menu text,
* *Title string "optional"* - menu header,
* *Conditions string* - rights for menu change.

EditMenu
------------------------------
This contract changes an existing menu in the current ecosystem.

Parameters

* *Id int* - ID of the menu to be changed,
* *Value string* - new text of menu,
* *Title string "optional"* - menu header,
* *Conditions string* - new rights for page change.

AppendMenu
------------------------------
This contract adds text to an existing menu in the current ecosystem.

Parameters

* *Id int* - complemented menu identifier,
* *Value string* - text to be added.

NewPage
------------------------------
This contract adds a new page in the current ecosystem. Parameters:

* *Name string* - page name,
* *Value string* - page text,
* *Menu string* - name of the menu, attached to this page,
* *Conditions string* - rights for change.

EditPage
------------------------------
This contract changes an existing page in the current ecosystem.

Parameters

* *Id int* - ID of the page to be changed,
* *Value string* - new text of the page,
* *Menu string* - name of the new menu on the page,
* *Conditions string* - new rights for page change.

AppendPage
------------------------------
The contract adds text to an existing page in the current ecosystem.

Parameters

* *Id int* - ID of the page to be changed,
* *Value string* - text that needs to be added to the page.

NewBlock
------------------------------
This contract adds a new page block with a template to the current ecosystem. 

Parameters

* *Name string* - block name,
* *Value string* - block text,
* *Conditions string* - rights for block change.

EditBlock
------------------------------
This contract changes an existing block in the current ecosystem.

Parameters

* *Id int* - ID of the block to be changed,
* *Value string* - new text of a block,
* *Conditions string* - new rights for change.

NewTable
------------------------------
This contract adds a new table in the current ecosystem. 

Parameters

* *Name string* - table name in Latin script, 
* *Columns string* - array of columns in JSON format ``[{"name":"...", "type":"...","index": "0", "conditions":"..."},...]``, where

  * *name* - column name in Latin script,
  * *type* - type ``varchar,bytea,number,datetime,money,text,double,character``,
  * *index* - non-indexed field - "0"; create index - "1",
  * *conditions* - condition for changing data in a column; read access rights should be specified in the JSON format. For example, ``{"update":"ContractConditions(`MainCondition`)", "read":"ContractConditions(`MainCondition`)"}``


* *Permissions string* - access conditions in JSON format ``{"insert": "...", "new_column": "...", "update": "..."}``.

  * *insert* - rights to insert records,
  * *new_column* - rights to add columns,
  * *update* - rights to change rights.

EditTable
------------------------------
This contract changes access permissions to tables in the current ecosystem. 

Parameters 

* *Name string* - table name, 
* *Permissions string* - access permissions in JSON format ``{"insert": "...", "new_column": "...", "update": "..."}``.

  * *insert* - condition to insert records,
  * *new_column* - condition to add columns,
  * *update* - condition to change data.   

NewColumn
------------------------------
This contract adds a new column to a table in the current ecosystem. 

Parameters

* *TableName string* - table name in,
* *Name* - column name in Latin script,
* *Type* - type ``varchar,bytea,number,money,datetime,text,double,character``,
* *Index* - non-indexed field - "0"; create index - "1",
* *Permissions* - condition for changing data in a column; read access rights should be specified in the JSON format. For example, ``{"update":"ContractConditions(`MainCondition`)", "read":"ContractConditions(`MainCondition`)"}``

EditColumn
------------------------------
This contract changes the rights to change a table column in the current ecosystem. 

Parameters

* *TableName string* - table name in Latin script, 
* *Name* - column name in Latin script,
* *Permissions* - condition for changing data in a column; read access rights should be specified in the JSON format. For example, ``{"update":"ContractConditions(`MainCondition`)", "read":"ContractConditions(`MainCondition`)"}``.

NewLang
------------------------------
This contract adds language resources in the current ecosystem. Permissions to add resources are set in the *changing_language* parameter in the ecosystem configuration. 

Parameters

* *Name string* - name of the language resource in Latin script, 
* *Trans* - language resources as a string in JSON format with two-character language codes as keys and translated strings as values. For example: ``{"en": "English text", "ru": "Английский текст"}``.

EditLang
------------------------------
This contract updates the language resource in the current ecosystem. Permissions to make changes are set in the *changing_language* parameter in the ecosystem configuration. 

Parameters

* *Name string* - name of the language resource,
* *Trans* - language resources as a string in JSON format with two-character language codes as keys and translated strings as values. For example ``{"en": "English text", "ru": "Английский текст"}``.
 
NewSign
------------------------------
This contract adds the signature confirmation requirement for a contract in the current ecosystem.

Parameters

* *Name string* - name of the contract, where an additional signature confirmation will be required,
* *Value string* - description of parameters in a JSON string, where
    
  * *title* - message text,
  * *params* - array of parameters that are displayed to users, where **name** is the field name, and **text** is the parameter description.
    
* *Conditions string* - condition for changing the parameters.

Example of *Value*

``{"title": "Would you like to sign?", "params":[{"name": "Recipient", "text": "Wallet"},{"name": "Amount", "text": "Amount(EGS)"}]}`` 

EditSign
------------------------------
The contract updates the parameters of a contract with a signature in the current ecosystem. 

Parameters

 * *Id int* - identifier of the signature to be changed,
 * *Value string* - a string containing new parameters,
 * *Conditions string* - new condition for changing the signature parameters.

Import 
------------------------------
This contract imports data from a *. sim file into the ecosystem.

Parameters

* *Data string* - data to be imported in text format; this data is the result of export from an ecosystem to a .sim file.

NewCron
------------------------------
The contract adds a new task in cron to be launched by timer. The contract is available only in VDE systems. Parameters:

* *Cron string* - a string that defines the launch of the contract by timer in the *cron* format,
* *Contract string* - name of the contract to launch in VDE; the contract should not have parameters in its ``data`` section,
* *Limit int* - an optional field, where the number of contract launches can be specified (until contract is executed this number of times),
* *Till string* - an optional string with the time when the task should be ended (this feature is not yet implemented),
* *Conditions string* - rights to modify the task.

EditCron
------------------------------
This contract changes the configuration of a task in cron for launch by timer. The contract is available only in VDE systems. Parameters:

* *Id int* - task ID,
* *Cron string* - a string that defines the launch of the contract by timer in the *cron* format; to disable a task, this parameter should be either an empty string or absent, 
* *Contract string* - name of the contract to launch in VDE; the contract should not have parameters in its data section,
* *Limit int* - an optional field, where the number of contract launches can be specified (until contract is executed this number of times),
* *Till string* - an optional string with the time of task should be ended (this feature is not yet implemented),
* *Conditions string* - new rights to modify the task.
