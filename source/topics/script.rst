Smart contracts
###############

A smart contract (hereinafter, "contract") is a basic element of applications, which performs a single action (typically, makes a record in a database table), initiated from the user interface by a user or by another contract. All operations with data in applications are formed as a system of contracts, interacting with each other through database tables or by call functions in a contract body.

Contracts are written using an original (developed by the team of platform developers) Turing-complete script language called Simvolio, with compilation into bytecode. The language includes a set of functions, operators and constructions that can be used for the implementation of data processing algorithms and operations with the database.

Contracts can be edited, but only if editing was not forbidden by way of putting ``false`` in the contract editing rights. Operations with data in the blockchain are performed by the most up-to-date (current) version of the contract. The complete history of changes made to contracts is stored in the blockchain and available from the software client.


Structure of the contract
=========================

Contracts are declared with the contract keyword, followed by the new contract's name. The contract's body must be enclosed in curly brackets. Every contract consists of three sections:

    #. **data** - declares the input data (names and types of variables).
    #. **conditions** - verifies the correctness of input data.
    #. **action** - includes the actions performed by the contract.

.. code:: js

  contract MyContract {
      data {
          FromId int
          ToId   int
          Amount money
      }
      func conditions {
          ...
      }
      func action {
      }
  }


Data section
------------

The contract input data, as well as the parameters of the form for the reception of the data are described in the ``data`` section.

The data are listed line by line: first, the variable name is specified (only variables, but not arrays are transferred), then the type and the parameters for the building of the interface form are indicated optionally through a gap in double quotation marks:

    * *optional* - form element that does not require to be filled.

.. code:: js

  contract my {
    data {
        Name string
        RequestId int
        Photo file "optional"
        Amount money
        Private bytes
    }
    ...
  }


Conditions section
------------------

Validation of the data obtained is performed in the Conditions section. The following commands are used to warn of the presence of errors: ``error``, ``warning``, ``info``. In fact, all three of them generate an error that stops the contract operation, but each of them displays a different message in the interface: *critical error*, *warning*, and *informative error*. For instance,


.. code:: js

  if fuel == 0 {
        error "fuel cannot be zero!"
  }
  if money < limit {
        warning Sprintf("You don't have enough money: %v < %v", money, limit)
  }
  if idexist > 0 {
        info "You have already been registered"
  }


Action section
--------------

The action section contains the contract's main program code that retrieves additional data and records the resulting values to database tables. For example,

.. code:: js

	action {
		DBUpdate("keys", $key_id,"-amount", $amount)
		DBUpdate("keys", $recipient,"+amount,pub", $amount, $Pub)
	}


Price function
--------------

A contract can contain the **price** function. This function determines the extra fuel cost for executing the contract, in fuel units. 

This function can return a value of *int* and *money* types. The returned value will be added to the cost of the contract execution and multiplied by **fuel_rate** coefficient.

.. code:: js
  
  contract MyContract {
    action {
               DBUpdate("keys", $key_id, {"-amount": $amount})
               DBUpdate("keys", $recipient, {"+amount": $amount,pub: $Pub})
    }
    func price int {
         return 10000
    }
  }


.. _simvolio-predefined-variables:

Variables in the contract
=========================

Contract input data, declared in the data section, is passed to other sections though variables with the ``$`` sign followed by data names. The ``$`` sign can be used to declare additional variables; such variables will be considered global for this contract and all nested contracts.

A contract can access predefined variables that contain data about the transaction, from which this contract was called.

    * ``$time`` – transaction time, int.

    * ``$ecosystem_id`` – ecosystem ID, int.

    * ``$block`` – number of the block, in which this transaction is included, int.

    * ``$key_id`` – ID of the account that signed the transaction if the contract is outside of ecosystem with ``ecosystem_id == 0``.

    * ``$type`` identifier of an external contract from where the current contract was called.

        .. todo::

            Check that this is still used.

    * ``$block_key_id`` – address of the node that formed the block, in which this transaction is included.

    * ``$block_time`` – time, when the block with the transaction containing the current contract was formed.

    * ``$original_contract`` – name of the contract, which was initially called for transaction processing. If this variable is an empty string, it means that the contract was called in the process of verification of a condition. To check whether this contract was called by another contract or directly from a transaction, the values of **$original_contract** and **$this_contract** are to be compared. If they are equal, it means that the contract was called from the transaction.

    * ``$this_contract`` – name of the currently executed contract.

    * ``$guest_key`` – guest account ID.

    * ``$stack`` – stack of contract calls. It is of the *array* type and contains strings with names of the called contract. Array element 0 is the currently executed contract. The last element is the name of the original contract that was called when transaction was processed.


    * ``$auth_token`` is the authorization token, which can be used in OBS contracts, for example, when calling contracts though API with the ``HTTPRequest`` function.

    .. todo::

        Check that this one is still used.

    .. code:: js

        var pars, heads map
        heads["Authorization"] = "Bearer " + $auth_token
        pars["obs"] = "false"
        ret = HTTPRequest("http://localhost:7079/api/v2/node/mycontract", "POST", heads, pars)


Predefined variables are accessible not only in contracts, but also in Permissions fields, (where conditions for access to application elements are defined), where they are used in construction of logical expressions. When used in Permissions fields, variables related to block formation (``$time``, ``$block``, etc.) always equal zero.

Predefined variable ``$result`` is used to return a value from a nested contract.

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


Nested Contracts
================

A nested contract can be called from the *conditions* and *action* sections of the enclosing contract. A nested contract can be called directly with parameters specified in parenthesis after its name (``NameContract(Params)``), or using the *CallContract* function, for which the contract name is passed using a string variable.


File Upload
===========

To upload files from ``multipart/form-data`` forms, the contract fields with type ``file`` must be used. Example:

.. code:: js

    contract Upload {
        data {
            File file
        }
        ...
    }

The `UploadBinary` system contract is intended to upload and store files.
To request a download link for a file from the template designer, there is a special template designer function – `Binary`.


Using JSON in PostgreSQL queries
================================

**JSON** type can be specified as column type. In this case, use the following syntax: **columnname->fieldname** to address record fields. The obtained value will be recorded in the column with name **columnname.fieldname**. Syntax **columnname->fieldname** can be used in parameters *Columns,One,Where* when using **DBFind**.

.. code:: js

    var ret map
    var val str
    var list array
    ret = DBFind("mytable").Columns("myname,doc,doc->ind").WhereId($Id).Row()
    val = ret["doc.ind"]
    val = DBFind("mytable").Columns("myname,doc->type").WhereId($Id).One("doc->type")
    list = DBFind("mytable").Columns("myname,doc,doc->ind").Where("doc->ind = ?", "101")
    val = DBFind("mytable").WhereId($Id).One("doc->check")


Date/time operations in PostgreSQL queries
==========================================

Functions do not allow direct possibilities to select, update, etc.. but they allow you to use the capabilities and functions of PostgreSQL when you get values and a description of the where conditions  in the samples. This includes, among other things, the functions for working with dates and time. For example, you need to compare the column *date_column* and the current time. If  *date_column* has the  type timestamp, then the expression will be the following ``date_column> now ()``. And if *date_column* stores time in Unix format as a number, then the expression will be ``to_timestamp (date_column)> now ()``.

.. code:: js

    to_timestamp(date_column) > now()
    date_initial < now() - 30 * interval '1 day'

Consider the situation when we have a value in Unix format and we need to write it in a field of type *timestamp *. In this case, when listing fields, before the name of this column you need to specify **timestamp**.

.. code:: js

   DBInsert("mytable", "name,timestamp mytime", "John Smith", 146724678424 )

If you have a string value of time and you need to write it in a field with the type *timestamp*, in this case, **timestamp** must be specified before the value itself.

.. code:: js

   DBInsert("mytable", "name,mytime", "John Smith", "timestamp 2017-05-20 00:00:00" )
   var date string
   date = "2017-05-20 00:00:00"
   DBInsert("mytable", "name,mytime", "John Smith", "timestamp " + date )
   DBInsert("mytable", "name,mytime", "John Smith", "timestamp " + $txtime )


Following Simvolio functions work with date and time in SQL format:

    - :ref:`simvolio-BlockTime`
    - :ref:`simvolio-DateTime`
    - :ref:`simvolio-UnixDateTime`


Contract Editor
===============

.. todo::

    This is offtopic, belongs in Molis description.

Contracts can be created and edited in a special editor which is a part of the Molis software client. Each new contract has a typical structure created in it by default with three sections: ``data, conditions, action``. The contracts editor helps to:

    - Write the contract code (highlighting key words of the Simvolio language.

    - Format the contract source code.

    - Bind the contract to an account, from which the payment for its execution will be charged.

    - Define permissions to edit the contract (typically, by specifying the contract name with the permissions stipulated in a special function ContractConditions or by way of direct indication of access conditions in the Change conditions field).

    - View the history of changes made to the contract with the option to restore previous versions.


Simvolio Contracts Language
###########################

|platform| contracts are written in a Turing-complete script language called Simvolio, with compilation into bytecode. The language includes a set of functions, operators and constructions that can be used for implementation of data processing algorithms and operations with the database.


Basic elements and constructions of the language
================================================


Data Types and Variables
------------------------

Data type must be defined for every variable. In obvious cases, data types are converted automatically. The following data types can be used:

    * ``bool`` - Boolean, can be true or false.

    * ``bytes`` - a sequence of bytes.

    * ``int`` - a 64-bit integer.

    * ``array`` - an array of values of arbitrary types.

    * ``map`` - an associative array of values of arbitrary data types with string keys.

    * ``money`` - an integer of the big integer type; values are stored in the database without decimal points, which are added when displaying values in the user interface in accordance with the currency configuration settings.

    * ``float`` - a 64-bit number with a floating point.

    * ``string`` - a string; must be defined in double quotes or back quotes: "This is a string" or \`This is a string\`.

    * ``file`` - associative array with a set of keys and values:

        * ``Name`` - file name (``string`` type).

        * ``MimeType`` - mime-type of the file (``string`` type).

        * ``Body`` - file contents (``bytes`` type).


All identifiers, including the names of variables, functions, contracts, etc. are case sensitive (MyFunc and myFunc are different names).

Variables are declared with the **var** keyword, followed by names and types of variables. Variables declared inside curly brackets must be used within the same pair of curly brackets. When declared, variables have default values: for *bool* type it is *false*, for all numeric types – zero values, for strings – empty strings. Examples of variables declaration:

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
------

The language supports two array types:

* ``array`` - a simple array with numeric index starting from zero.
* ``map`` - an associative array with string keys.

When assigning and и retrieving array elements, index must be put in square brackets. Multiple indexes are not supported in arrays. For example, you cannot address an array element as *myarr[i][j]*.

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


You can also define arrays of ``array`` type by specifying elements in ``[]``. For ``map`` type arrays, use ``{}``.

.. code:: js

     var my map
     my={"key1": "value1", key2: i, "key3": $Name}
     var mya array
     mya=["value1", {key2: i}, $Name]

You can use such initializaton in the expressions. For example, this can be used in the function parameters.

.. code:: js

     DBFind...Where({id: 1})

For associative arrays, you must specify a key. Keys are specified as a string in double quotes (``""``). If key name contrains only letters, numbers and underscore, then double quotes can be omitted.

.. code:: js

    {key1: "value1", key2: "value2"}

An array can contain strings, numbers, variable names of any type, and variable names with ``$`` sign. Nested arrays are supported. A different map or array can be specified as a value.

Expressions cannot be used as array elements. Use a variable to store the expression result and specify this variable instead.

.. code:: js

     [1+2, myfunc(), name["param"]] // don't do this
     [1, 3.4, mystr, "string", $ext, myarr, mymap, {"ids": [1,2, i], company: {"Name": "MyCompany"}} ] // this is ok

     var val string
     val = my["param"]
     MyFunc({key: val, sub: {name: "My name", "color": "Red"}})



If and While Statements
-----------------------

The contract language supports the standard **if** conditional statement and the **while** loop, which can be used in functions and contracts. These statements can be nested in each other.

A keyword must be followed by a conditional statement. If the conditional statement returns a number, then it is considered as *false* when its value = zero.

For example, *val == 0* is equivalent to *!val*, and *val != 0* is the same as just *val*. The **if** statement can have an **else** block, which executes in case the **if** conditional statement is false. The following comparison operators can be used in conditional statements: ``<, >, >=, <=, ==, !=``, as well as ``||`` (OR) and ``&&`` (AND).

.. code:: js

    if val > 10 || id != $citizen {
        ...
    } else {
        ...
    }

The **while** statement is intended for implementation of loops. A **while** block will be executed while its condition is true. The **break** operator is used to end a loop inside a block. To start a loop from the beginning, the **continue** operator must be used.

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

Apart from conditional statements, the language supports standard arithmetic operations: ``+``, ``-``, ``*``, ``/``.

Variables of **string** and **bytes** types can be used as a condition. In this case, the condition will be true when the length of the string (bytes) is greater than zero, and false for an empty string.


Functions
---------

Functions of the contracts language perform operations with data received in the data section of a contract: reading and writing database values, converting value types, and establishing connections between contracts.

Functions are declared with the **func** keyword, followed by the function name and a list of parameters passed to it (with their types), all enclosed in curly brackets and separated by commas. After the closing curly bracket the data type of the value returned by the function must be stated. The function body must be enclosed in curly brackets. If a function does not have parameters, then the curly brackets are not necessary. To return a value from a function, the ``return`` keyword is used.

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

An undefined number of parameters can be passed to a function. To do this, put ``...`` instead of the type of the last parameter. In this case, the data type of the last parameter will be *array*, and it will contain all, starting from this parameter, variables that were passed with the call. Variables of any type can be passed, but you should take care of possible conflicts related to data type mismatch.

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

Let's consider a situation, where a function has many parameters, but we need only some of them when calling it. In this case, optional parameters can be declared in the following way: ``func myfunc(name string).Param1(param string).Param2(param2 int) {...}``. You can specify only the parameters you need with the call in arbitrary order: ``myfunc("name").Param2(100)``. In the function body you can address these variables as usual. If an extended parameter is not specified with the call, it will have the default value, for example, an empty string for a string and zero for a number. You can specify several extended parameters and use ``...``: ``func DBFind(table string).Where(request string, params ...)`` and call ``DBFind("mytable").Where("id > ? and type = ?", myid, 2)``

.. code:: js

    func DBFind(table string).Columns(columns string).Where(format string, tail ...)
             .Limit(limit int).Offset(offset int) string  {
       ...
    }

    func names() string {
       ...
       return DBFind("table").Columns("name").Where("id=?", 100).Limit(1)
    }


Simvolio functions by purpose
=============================


Retrieving values from the database:

.. hlist::
    :columns: 3

    - :ref:`simvolio-AppParam`
    - :ref:`simvolio-DBFind`
    - :ref:`simvolio-DBRow`
    - :ref:`simvolio-DBSelectMetrics`
    - :ref:`simvolio-EcosysParam`
    - :ref:`simvolio-GetHistory`
    - :ref:`simvolio-GetHistoryRow`
    - :ref:`simvolio-GetColumnType`
    - :ref:`simvolio-GetDataFromXLSX`
    - :ref:`simvolio-GetRowsCountXLSX`
    - :ref:`simvolio-GetBlock`
    - :ref:`simvolio-LangRes`


Changing values in tables:

.. hlist::
    :columns: 3

    - :ref:`simvolio-DBInsert`
    - :ref:`simvolio-DBUpdate`
    - :ref:`simvolio-DBUpdateExt`
    - :ref:`simvolio-DelColumn`
    - :ref:`simvolio-DelTable`


Array operations:

.. hlist::
    :columns: 3

    - :ref:`simvolio-Append`
    - :ref:`simvolio-Join`
    - :ref:`simvolio-Split`
    - :ref:`simvolio-Len`
    - :ref:`simvolio-Row`
    - :ref:`simvolio-One`
    - :ref:`simvolio-GetMapKeys`
    - :ref:`simvolio-SortedKeys`

Operations with contracts and conditions:

.. hlist::
    :columns: 3

    - :ref:`simvolio-CallContract`
    - :ref:`simvolio-ContractAccess`
    - :ref:`simvolio-ContractConditions`
    - :ref:`simvolio-EvalCondition`
    - :ref:`simvolio-GetContractById`
    - :ref:`simvolio-RoleAccess`
    - :ref:`simvolio-GetContractByName`
    - :ref:`simvolio-TransactionInfo`
    - :ref:`simvolio-Throw`
    - :ref:`simvolio-ValidateCondition`


Operations with account addresses:

.. hlist::
    :columns: 3

    - :ref:`simvolio-AddressToId`
    - :ref:`simvolio-IdToAddress`
    - :ref:`simvolio-PubToID`


Operations with values of variables:

.. hlist::
    :columns: 3

    - :ref:`simvolio-DecodeBase64`
    - :ref:`simvolio-EncodeBase64`
    - :ref:`simvolio-Float`
    - :ref:`simvolio-HexToBytes`
    - :ref:`simvolio-FormatMoney`
    - :ref:`simvolio-Random`
    - :ref:`simvolio-Int`
    - :ref:`simvolio-Hash`
    - :ref:`simvolio-Sha256`
    - :ref:`simvolio-Str`
    - :ref:`simvolio-UpdateLang`

Mathematical operations:

.. hlist::
  :columns: 3

  - :ref:`simvolio-Floor`
  - :ref:`simvolio-Log`
  - :ref:`simvolio-Log10`
  - :ref:`simvolio-Pow`
  - :ref:`simvolio-Round`
  - :ref:`simvolio-Sqrt`


Operations with JSON:

.. hlist::
    :columns: 3

    - :ref:`simvolio-JSONEncode`
    - :ref:`simvolio-JSONEncodeIndent`
    - :ref:`simvolio-JSONDecode`

Operations with strings:

.. hlist::
    :columns: 3

    - :ref:`simvolio-HasPrefix`
    - :ref:`simvolio-Contains`
    - :ref:`simvolio-Replace`
    - :ref:`simvolio-Size`
    - :ref:`simvolio-Sprintf`
    - :ref:`simvolio-Substr`
    - :ref:`simvolio-ToLower`
    - :ref:`simvolio-ToUpper`
    - :ref:`simvolio-TrimSpace`

Operations with bytes:

.. hlist::
    :columns: 3

    - :ref:`simvolio-StringToBytes`
    - :ref:`simvolio-BytesToString`

Operations with date and time in SQL format:

.. hlist::
    :columns: 3

    - :ref:`simvolio-BlockTime`
    - :ref:`simvolio-DateTime`
    - :ref:`simvolio-UnixDateTime`

Operations with system parameters:

.. hlist::
    :columns: 3

    - :ref:`simvolio-SysParamString`
    - :ref:`simvolio-SysParamInt`
    - :ref:`simvolio-DBUpdateSysParam`
    - :ref:`simvolio-UpdateNotifications`
    - :ref:`simvolio-UpdateRolesNotifications`

Functions for OBS mode:

.. hlist::
    :columns: 3

    - :ref:`simvolio-HTTPRequest`
    - :ref:`simvolio-HTTPPostJSON`


Functions for OBS Master mode:

.. hlist::
    :columns: 3

    - :ref:`simvolio-CreateOBS`
    - :ref:`simvolio-ListOBS`
    - :ref:`simvolio-RunOBS`
    - :ref:`simvolio-StopOBS`
    - :ref:`simvolio-RemoveOBS`


Simvolio functions reference
============================


.. _simvolio-AppParam:

AppParam
--------

Returns the value of a specified application parameter (from the application parameters table *app_params*).


Syntax
""""""

.. code-block:: text
    
    AppParam(app int, name string, ecosystemid int) string


.. describe:: app

    Application identifer.

.. describe:: name

    Application parameter name.

.. describe:: ecosystemid

    Ecosystem identifier.


Example
"""""""

.. code:: js

    AppParam(1, "app_account", 1)


.. _simvolio-DBFind:

DBFind
------

Receives data from a database table in accordance with the specified request. 

Returs an *array* comprised of *map* associative arrays that contain data from the database record. To get the first *map* element (first record from the request), use the ``.Row()`` tail function. To get a value from the first *map* element (a value from the specific column), use the ``.One(column string)``.

Syntax
""""""

.. code-block:: text

    DBFind(table string)
        [.Columns(columns array|string)]
        [.Where(where map)]
        [.WhereId(id int)]
        [.Order(order string)]
        [.Limit(limit int)]
        [.Offset(offset int)]
        [.Ecosystem(ecosystemid int)] array

.. describe:: table

    Table name.

.. describe:: сolumns

    List of returned columns. If not specified, all columns will be returned.

    This value can be specified as an array or as a string with comma separators.

.. describe:: where

    Search condition.

    Example: ``.Where({name: "John"})`` or ``.Where({"id": {"$gte": 4}})``.

    This parameter must contain an associative array with search conditions. This array may contain nested elements.

    Following syntactic constructions are possible:

    - ``{"field1": "value1", "field2" : "value2"}``

        This is equivalent to ``field1 = "value1" AND field2 = "value2"``.

    - ``{"field1": {"$eq":"value"}}``

        This is equivalent to ``field = "value"``.

    - ``{"field1": {"$neq": "value"}}``

        This is equivalent to ``field != "value"``.


    - ``{"field1: {"$in": [1,2,3]}``

        This is equivalent to ``field IN (1,2,3)``.

    - ``{"field1": {"$nin" : [1,2,3]}``

        This is equivalent to ``field NOT IN (1,2,3)``.
    
    - ``{"field": {"$lt": 12}}``

        This is equivalent to ``field < 12``.
    
    - ``{"field": {"$lte": 12}}``

        This is equivalent to ``field <= 12``.
    
    - ``{"field": {"$gt": 12}}``

        This is equivalent to ``field > 12``.
    
    - ``{"field": {"$gte": 12}}``

        This is equivalent to ``field >= 12``.
    
    - ``{"$and": [<expr1>, <expr2>, <expr3>]}``

        This is equivalent to ``expr1 AND expr2 AND expr3``.
    
    - ``{"$or": [<expr1>, <expr2>, <expr3>]}``

        This is equivalent to ``expr1 OR expr2 OR expr3``.
    
    - ``{field: {"$like": "value"}}``

        This is equivalent to ``field like '%value%'`` (substring search).
    
    - ``{field: {"$begin": "value"}}``

        This is equivalent to ``field like 'value%'`` (begins with ``value``).
    
    - ``{field: {"$end": "value"}}``

        This is equivalent to ``field like '%value'`` (ends with ``value``).
    
    - ``{field: "$isnull"}``

        This is equivalent to ``field is null``.

    Make sure that you don't overwrite associative array keys. For example, if you want to make the ``id>2 and id<5`` query, you cannot specify ``{id:{"$gt": 2}, id:{"$lt": 5}}``, because the first ``id`` element will be overwritten by the second one. Instead, you can use the following constructions:

    .. code:: js

        {id: [{"$gt": 2}, {"$lt": 5}]}

    .. code:: js

        {"$and": [{id:{"$gt": 2}}, {id:{"$lt": 5}}]}


.. describe:: id

    Search by identifier. For example, ``.WhereId(1)``.

.. describe:: order

    A field that will be used for sorting. By default, values are sorted by *id*.

    If sorting is done using only one field, then it can be specified as a string. Otherwise, pass an array of strings and objects:

        Descending order: ``{"field": "-1"}``. Eqivalent of ``field desc``. 

        Ascending order: ``{"field": "1"}``. Eqivalent of ``field asc``. 

.. describe:: limit

    Maximum number of returned values (default = 25, maximum = 250).

.. describe:: offset

    Offset for returned values.

.. describe:: ecosystemid

    Ecosystem ID.

    By default, values are taken from the table in the current ecosystem.


Example
"""""""

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


.. _simvolio-DBRow:

DBRow
-----

Returns an associative array *map* with data obtained from a database table in accordance with the specified query.


Syntax
""""""

.. code-block:: text

    DBRow(table string)
        [.Columns(columns array|string)]
        [.Where(where map)]
        [.WhereId(id int)]
        [.Order(order array|string)]
        [.Ecosystem(ecosystemid int)] map

.. describe:: table

    Table name.

.. describe:: columns

    List of returned columns. If not specified, all columns will be returned.

    This value can be specified as an array or as a string with comma separators.

.. describe:: where

    Search condition.

    Example: ``.Where({name: "John"})`` or ``.Where({"id": {"$gte": 4}})``.

    For more information, see :ref:`simvolio-DBFind`.

.. describe:: id

    Search by identifier. For example, ``.WhereId(1)``.

.. describe:: order

    A field that will be used for sorting. By default, values are sorted by *id*.

    For more information, see :ref:`simvolio-DBFind`.


.. describe:: ecosystemid

    Ecosystem identifier. By default, current ecosystem identifer.


Example
"""""""

.. code:: js

   var ret map
   ret = DBRow("contracts").Columns(["id","value"]).Where({id: 1})
   Println(ret)


.. _simvolio-DBSelectMetrics:

DBSelectMetrics
---------------

Returns aggregated data for a metric.

Metrics are updated each time 100 blocks are generated. Aggregated data is stored in 1-day periods.


Syntax
""""""

.. code-block:: text

    DBSelectMetrics(metric string, timeInterval string, aggregateFunc string) array


.. describe:: metric

    Metric name.

    .. describe:: ecosystem_pages

        Number of pages in ecosystems.

        Returned values: *key* is an ecosystem ID, *value* is the number of pages in this ecosystem.

    .. describe:: ecosystem_members

        Number of members in ecosystems.

        Returned values: *key* is an ecosystem ID, *value* is the number of members in this ecosystem.

    .. describe:: ecosystem_tx

        Number of transactions in ecosystems.

        Returned values: *key* is an ecosystem ID, *value* is the number of transactions in this ecosystem.

.. describe:: timeInterval

    Time interval for aggregated metric data. Examples: ``1 day``, ``30 days``.

.. describe:: aggregateFunc

    Aggregation function. Examples: ``max``, ``min``, ``avg``.


Example
"""""""

In the example below, ``row`` contains a map with ``key`` and ``value`` keys. The ``key`` key contains ecosystem IDs, the ``value`` key contains metric value. 

.. code:: js

   var rows array
   rows = DBSelectMetrics("ecosystem_tx", "30 days", "avg")

   var i int
   while(i < Len(rows)) {
      var row map
      row = rows[i]
      i = i + 1
   }


.. _simvolio-EcosysParam:

EcosysParam
-----------

Returns the value of a specified parameter from the ecosystem settings (*parameters* table).


Syntax
""""""

.. code-block:: text

    EcosysParam(name string) string


.. describe:: name

    Name of the received parameter.

.. describe:: num

    Position of the parameter.


Example
"""""""

.. code:: js

    Println( EcosysParam("gov_account"))


.. _simvolio-GetHistory:

GetHistory
----------

Returns the history of changes of a record from the specified table.


Syntax
""""""

.. code-block:: text

    GetHistory(table string, id int) array

.. describe:: table

    Table name.

.. describe:: id

    Identifier of a record.


Return value
""""""""""""

Return an array of associative arrays of *map* type. These arrays contain the history of changes of a record in the specified table.

Each associative array contains fields of a record before the next change was made.

The resulting list is sorted in the order from recent changes to earlier ones.

The *id* field in the resulting table points to the id in the *rollback_tx* table. The *block_id* field contains the block number. The *block_time* field contains the block timestamp.


Example
"""""""

.. code:: js

    var list array
    var item map
    list = GetHistory("blocks", 1)
    if Len(list) > 0 {
       item = list[0]
    }


.. _simvolio-GetHistoryRow:

GetHistoryRow
-------------

Returns a single snapshot from the history of changes of a record in a specified table.


Syntax
""""""

.. code-block:: text

    GetHistoryRow(table string, id int, rollbackId int) map

.. describe:: table

    Table name.

.. describe:: id

    Record identifier.

.. describe:: RollbackId

    Identifier of the id record in the *rollback_tx* table.


Return value
""""""""""""

Returns a single history record with the *rollbackId* identifier from the *rollback_tx* table.


.. todo::

    Add example.


.. _simvolio-GetColumnType:

GetColumnType
-------------

Returns the type of a column in a specified table.

Syntax
""""""

.. code-block:: text

    GetColumnType(table, column string) string

.. describe:: table

    Table name.

.. describe:: column

    Column name.


Returned value
""""""""""""""

Following column types can be returned: *text, varchar, number, money, double, bytes, json, datetime, double*.


Example
"""""""

.. code:: js

    var coltype string
    coltype = GetColumnType("members", "member_name")

.. _simvolio-GetDataFromXLSX:

GetDataFromXLSX
---------------

Returns data from XLSX spreadsheets.


Syntax
""""""

.. code-block:: text

    GetDataFromXLSX(binId int, line int, count int, sheet int) string

.. describe:: binId

    Identifier of an XLSX spreadsheet from the *binary* table.

.. describe:: line

    Spreadsheet line, starting from 0.

.. describe:: count

    Number of lines to return.

.. describe:: sheet

    List number in XLSX file, starting from 1.


Example
"""""""

.. code:: js

    var a array
    a = GetDataFromXLSX(binid, 12, 10, 1)


Returned value
""""""""""""""

Returned value is an array that contains arrays with spreadsheet cell data.


.. _simvolio-GetRowsCountXLSX:

GetRowsCountXLSX
----------------

Returns the number of rows in a specified XLSX file.


Syntax
""""""

.. code-block:: text

    GetRowsCountXLSX(binId int, sheet int) int

.. describe:: binId

    Identifier of an XLSX spreadsheet from the *binary* table.

.. describe:: sheet

    List number in XLSX file, starting from 1.


Example
"""""""

.. code:: js

    var count int
    count = GetRowsCountXLSX(binid, 1)


.. _simvolio-LangRes:

LangRes
-------

Returns a language resource with name label for language *lang*, specified as a two-character code, for instance, *en, fr, ru*; if there is no language resource for a selected language, the result will be returned in English.


Syntax
""""""

.. code-block:: text

    LangRes(label string, lang string) string


.. describe:: label

    Language resource name.

.. describe:: lang

    Two-character language code.


Example
"""""""

.. code:: js

    warning LangRes("confirm", $Lang)
    error LangRes("problems", "de")


.. _simvolio-GetBlock:

GetBlock
--------

Returns information about a block.


Syntax
""""""

.. code-block:: text

    GetBlock(blockID int64) map

.. describe:: blockID

    Block identifier.


Returned value
""""""""""""""

Returned value is a *map* associative array that contains the following data:

- *id*

    Block identifier.

- *time*

    Block generation time, in unixtime format.

- *key_id*

    Node key of a node that generated the block.


Example
"""""""

.. code:: js

   var b map
   b = GetBlock(1)
   Println(b)


.. _simvolio-DBInsert:

DBInsert
--------

Adds a record to a specified *table* and returns the **id** of the inserted record.


Syntax
""""""

.. code-block:: text

    DBInsert(table string, params map) int


.. describe:: tblname

    Name of the table in the database.

.. describe:: params

    Associative array where keys are field names and values are values to insert.


Example
"""""""

    DBInsert("mytable", {name: "John Smith", amount: 100})


.. _simvolio-DBUpdate:

DBUpdate
--------

Changes column values in the table for the record with a specified **id**. If a record with this identifier does not exist, the operation will result with an error.


Syntax
""""""

.. code-block:: text

    DBUpdate(tblname string, id int, params map)


.. describe:: tblname

    Name of the table in the database.

.. describe:: id

    Identifier **id** of the changeable record.

.. describe:: params

    Associative array where keys are field names and values are values to update.


Example
"""""""

.. code:: js

    DBUpdate("mytable", myid, {name: "John Smith", amount: 100})


.. _simvolio-DBUpdateExt:

DBUpdateExt
-----------

The function updates columns in a record whose column has a specified value. The table must have an index for a specified column.


Syntax
""""""

.. code-block:: text

    DBUpdateExt(tblname string, column string, value (int|string), params map)


.. describe:: tblname

    Name of the table in the database.

.. describe:: column

    Name of the column by which the record will be searched for.

.. describe:: value

    Value for searching a record in a column.

.. describe:: params

    Associative array where keys are field names and values are values to update.


Example
"""""""

.. code:: js

    DBUpdateExt("mytable", "address", addr, {name: "John Smith", amount: 100})


.. _simvolio-DelColumn:

DelColumn
---------

Deletes a column in the specified table. The table must have no records in it.


Syntax
""""""

.. code-block:: text

    DelColumn(tblname string, column string)


.. describe:: tblname

    Name of the table in the database.

.. describe:: column

    Name of the column that must be deleted.

.. code:: js

    DelColumn("mytable", "mycolumn")


.. _simvolio-DelTable:

DelTable
--------

Deletes the specified table. The table must have no records in it.


Syntax
""""""

.. code-block:: text

    DelTable(tblname string)

.. describe:: tblname

    Name of the table in the database.


Example
"""""""

.. code:: js

    DelTable("mytable")


.. _simvolio-Append:

Append
------

Inserts *val* of any *type* to an *src* array.


Syntax
""""""

.. code-block:: text

    Append(src array, val anyType) array

.. describe:: src

    An array.

.. describe:: val

    Value to append.


Example
"""""""

.. code:: js

  var list array
  list = Append(list, "new_val")


.. _simvolio-Join:

Join
----

Merges the elements of the *in* array into a string with the specified *sep* separator.


Syntax
""""""

.. code-block:: text

    Join(in array, sep string) string


.. describe:: in

    Name of the *array* type array, the elements of which you want to merge.

.. describe:: sep

    Separator string.


Example
"""""""

.. code:: js

    var val string, myarr array
    myarr[0] = "first"
    myarr[1] = 10
    val = Join(myarr, ",")


.. _simvolio-Split:

Split
-----

Splits the *in* string into elements using *sep* as a separator, and puts them into an array.


Syntax
""""""

.. code-block:: text

    Split(in string, sep string) array


.. describe:: in

     is the initial string.

.. describe:: sep

     is the separator string.


Example
"""""""

.. code:: js

    var myarr array
    myarr = Split("first,second,third", ",")


.. _simvolio-Len:

Len
---

Returns the number of elements in the specified array.


Syntax
""""""

.. code-block:: text

    Len(val array) int


.. describe:: val

    Array of the *array* type.


Example
"""""""

.. code:: js

    if Len(mylist) == 0 {
      ...
    }


.. _simvolio-Row:

Row
---

Returns the first *map* associative array from the *list* array. If the *list* is empty, then the result will be an empty *map*. This function is mostly used with the DBFind function. The *list* parameter must not be specified in this case.


Syntax
""""""

.. code-block:: text

    Row(list array) map


.. describe:: list

     - a map array, returned by the **DBFind** function.


Example
"""""""

.. code:: js

   var ret map
   ret = DBFind("contracts").Columns("id,value").WhereId(10).Row()
   Println(ret)


.. _simvolio-One:

One
---

The function returns the value of the *column* key from the first associative array in the *list* array. If the *list* list is empty, then nil is returned. This function is mostly used with the DBFind function. The *list* parameter must not be specified in this case.


Syntax
""""""

.. code-block:: text

    One(list array, column string) string


.. describe:: list

     - a map array, returned by the **DBFind** function.

.. describe:: column

     - name of the returned key.


Example
"""""""

.. code:: js

   var ret string
   ret = DBFind("contracts").Columns("id,value").WhereId(10).One("value")
   if ret != nil {
      Println(ret)
   }

.. _simvolio-GetMapKeys:

GetMapKeys
----------

Returns an array of keys from the *val* associative array.


Syntax
""""""

.. code-block:: text

    GetMapKeys(val map) array


.. describe:: val

    An array.


Example
"""""""

.. code:: js

   var val map
   var arr array
   val["k1"] = "v1"
   val["k2"] = "v2"


.. _simvolio-SortedKeys:

SortedKeys
----------

Returns a sorted array of keys from the *val* associative array.


Syntax
""""""

.. code-block:: text

    SortedKeys(val map) array


.. describe:: val

    An array.


Example
"""""""

.. code:: js

   var val map
   var arr array
   val["k1"] = "v1"
   val["k2"] = "v2"
   arr = SortedKeys(val)


.. _simvolio-CallContract:

CallContract
------------


Calls a contract by its name. All the parameters specified in the section *data* of the contract must be listed in the transmitted array. The function returns the value that was assigned to **$result**  variable in the contract.


Syntax
""""""

.. code-block:: text

    CallContract(name string, params map)


.. describe:: name

    Name of the contract being called.

.. describe:: params

    Associative array with input data for the contract.


Example
"""""""

.. code:: js

    var par map
    par["Name"] = "My Name"
    CallContract("MyContract", par)


.. _simvolio-ContractAccess:

ContractAccess
--------------

Checks whether the name of the executed contract matches with one of the names listed in the parameters. Typically used to control access of contracts to tables. The function is specified in the *Permissions* fields when editing table columns or in the *Insert* and *New Column* fields in the *Table permission* section.


Syntax
""""""

.. code-block:: text

    ContractAccess(name string, [name string]) bool


.. describe:: name

    Contract name.


Example
"""""""

.. code:: js

    ContractAccess("MyContract")
    ContractAccess("MyContract","SimpleContract")




.. _simvolio-ContractConditions:

ContractConditions
------------------

Calls the **conditions** section from contracts with specified names.

For such contracts, the *data* block must be empty. If the conditions *conditions* is executed without errors, then *true* is returned. If an error is generated during execution, the parent contract will also end with this error. This function is usually used to control access of contracts to tables and can be called in the *Permissions* fields when editing system table.


Syntax
""""""

.. code-block:: text

    ContractConditions(name string, [name string]) bool

.. describe:: name

    Contract name.


Example
"""""""

.. code:: js

    ContractConditions("MainCondition")



.. _simvolio-EvalCondition:

EvalCondition
-------------

Takes from the *tablename* table the value of the *condfield* field from the record with the *'name'* field, which is equal to the *name* parameter and checks if the condition from the field *condfield* is made.


Syntax
""""""

.. code-block:: text

    EvalCondition(tablename string, name string, condfield string)


.. describe:: tablename

    Name of the table.

.. describe:: name

    Value for searching by the field 'name'.

.. describe:: condfield

    Name of the field where the condition to be checked is stored.


Example
"""""""

.. code:: js

    EvalCondition(`menu`, $Name, `condition`)



.. _simvolio-GetContractById:

GetContractById
---------------

The function returns the contract name by its identifier. If the contract can't be found, an empty string will be returned.


Syntax
""""""

.. code-block:: text

    GetContractById(id int) string


.. describe:: id

    Contract identifier in the *contracts* table.


Example
"""""""

.. code:: js

    var name string
    name = GetContractById($IdContract)



.. _simvolio-GetContractByName:

GetContractByName
-----------------

Кeturns a contract identifier in the *contracts* by its name. If the contract does not exist, a zero value will be returned.


Syntax
""""""

.. code-block:: text

    GetContractByName(name string) int

.. describe:: name

    Сontract identifier in the *contracts* table.


Example
"""""""


.. code:: js

    var id int
    id = GetContractByName(`NewBlock`)


.. _simvolio-RoleAccess:

RoleAccess
----------

Checks if the contract caller's role identifier matches one of the identifiers specified in parameters.

Use this function to control contract access to tables and other data.

Syntax
""""""

.. code-block:: text

    RoleAccess(id int, [id int]) bool


.. describe:: id

    Role identifier.


Example
"""""""

.. code:: js

    RoleAccess(1)
    RoleAccess(1, 3)


.. _simvolio-TransactionInfo:

TransactionInfo
---------------

Searches a transaction by the specified hash and returns information about the executed contract and its parameters.


Syntax
""""""

.. code-block:: text

    TransactionInfo(hash: string)


.. describe:: hash

    Transaction hash in a hex string format.


Return value
""""""""""""

The function returns a string in the json format:

.. code-block:: json

    {"contract":"ContractName", "params":{"key": "val"}, "block": "N"}


.. describe:: contract

    Contract name.

.. describe:: params

    Parameters passed to the contract.

.. describe:: block

    Block ID where this transaction was processed.


Example
"""""""

.. code:: js

    var out map
    out = JSONDecode(TransactionInfo(hash))


.. _simvolio-Throw:

Throw
-----

Generates an error of type *exception*, but adds an *id* field to it.


Syntax
""""""

.. code-block:: text

    Throw(ErrorId string, ErrDescription string)

.. describe:: ErrorId

    Error identifier.

.. describe:: ErrDescription

    Error description.


Results
"""""""

The result of such transaction has this format:

.. code-block:: json

    {"type":"exception","error":"Error description","id":"Error ID"}


Example
"""""""

.. code:: js

    Throw("Problem", "There is a problem")


.. _simvolio-ValidateCondition:

ValidateCondition
-----------------

Attempts to compile the condition specified in the *condition* parameter. If a mistake occurs during the compilation process, the mistake will be generated and the calling contract will terminate. This function is designed to check the correctness of the conditions when they change.


Syntax
""""""

.. code-block:: text

    ValidateCondition(condition string, state int)


.. describe:: condition

    Condition to validate.

.. describe:: state

    State identifier. Specify 0 if checking for global conditions.


Example
"""""""

.. code:: js

    ValidateCondition(`ContractAccess("@1MyContract")`, 1)


.. _simvolio-AddressToId:

AddressToId
-----------

Returns the identification number of the citizen by the string value of the address of his account. If a wrong adress is specified, then 0 returns.


Syntax
""""""

.. code-block:: text

    AddressToId(address string) int


.. describe:: address

    Account address in the ``XXXX-...-XXXX`` format or in the number form.


Example
"""""""

.. code:: js

    wallet = AddressToId($Recipient)



.. _simvolio-IdToAddress:

IdToAddress
-----------

Returns the address of a account based on its ID number. If a wrong ID is specified, returned is 'invalid'.


Syntax
""""""

.. code-block:: text

    IdToAddress(id int) string


.. describe:: id

    ID, numerical.


Example
"""""""

.. code:: js

    $address = IdToAddress($id)



.. _simvolio-PubToID:

PubToID
-------

Returns the account address by the public key in hexadecimal encoding.


Syntax
""""""

.. code-block:: text

    PubToID(hexkey string) int


.. describe:: hexkey

    Public key in hexadecimal form.


Example
"""""""

.. code:: js

    var wallet int
    wallet = PubToID("fa5e78.....34abd6")


.. _simvolio-DecodeBase64:

DecodeBase64
------------


Decodes a string in base64 encoding.


Syntax
""""""

.. code-block:: text

    DecodeBase64(input string) string


.. describe:: input

    Input string in base64 encoding.


Example
"""""""

.. code:: js

    val = DecodeBase64(mybase64)


.. _simvolio-EncodeBase64:

EncodeBase64
------------

Encodes a string in base64 encoding and returns a string in the encoded format.


Syntax
""""""

.. code-block:: text

    EncodeBase64(input string) string

.. describe:: input

    Input string.


Example
"""""""

.. code:: js

    var base64str string
    base64str = EncodeBase64("my text")


.. _simvolio-Float:

Float
-----

Converts an integer *int* or *string* to a floating-point number.


Syntax
""""""

.. code-block:: text

    Float(val int|string) float

.. describe:: val

    An integer or a string.


Example
"""""""

.. code:: js

    val = Float("567.989") + Float(232)



.. _simvolio-HexToBytes:

HexToBytes
----------

Converts a string with hexadecimal encoding to a *bytes* value (sequence of bytes).


Syntax
""""""

.. code-block:: text

    HexToBytes(hexdata string) bytes


.. describe:: hexdata

    A string containing a hexadecimal notation.


Example
"""""""

.. code:: js

    var val bytes
    val = HexToBytes("34fe4501a4d80094")



.. _simvolio-FormatMoney:

FormatMoney
-----------

Returns a string value of exp/10^digit. If *digit* parameter is not specified, it is taken from the **money_digit** ecosystem parameter.


Syntax
""""""

.. code-block:: text

    FormatMoney(exp string, digit int)

.. describe:: exp

    Numeric value as a string.

.. describe:: digit

    Exponent of the base 10 in the ``exp/10^digit`` expression. This value can be positive or negative. Positive value determines the number of digits after the comma.


Example
"""""""

.. code:: js

    s = FormatMoney("123456723722323332", 0)


.. _simvolio-Random:

Random
------

Returns a random number in the range between min and max (min <= result < max). Both min and max must be positive numbers.


Syntax
""""""

.. code-block:: text

    Random(min int, max int) int

.. describe:: min

    Minimum value for the random number.

.. describe:: max

    Upper bound for random numbers. Generated random numbers will be smaller than this value.


Example
"""""""

.. code:: js

    i = Random(10,5000)


.. _simvolio-Int:

Int
----

Converts a string value to an integer.


Syntax
""""""

.. code-block:: text

    Int(val string) int

.. describe:: val

    String containing a number.


Example
"""""""

.. code:: js

    mystr = "-37763499007332"
    val = Int(mystr)


.. _simvolio-Hash:

Hash
----

Accepts a byte array or a string and returns a hash that was generated by the system cryptoprovider.


Syntax
""""""

.. code-block:: text

    Hash(val interface{}) string, error

.. describe:: val

    A string or a byte array.


Example
"""""""

.. code:: js

    var hash string
    hash = Hash("Test message")



.. _simvolio-Sha256:

Sha256
------

Feturns **SHA256** hash of a specified string.


Syntax
""""""

.. code-block:: text

    Sha256(val string) string


.. describe:: val

    Incoming line for which the **Sha256** hash must be calculated.


Example
"""""""

.. code:: js

    var sha string
    sha = Sha256("Test message")


.. _simvolio-Str:

Str
---

Converts a numeric *int* or *float* value to a string.


Syntax
""""""

.. code-block:: text

    Str(val int|float) string

.. describe:: val

    An integer or a floating-point number.


Example
"""""""

.. code:: js

    myfloat = 5.678
    val = Str(myfloat)



.. _simvolio-UpdateLang:

UpdateLang
----------

Updates the language source in the memory. Is used in the transactions that change language sources.


Syntax
""""""

.. code-block:: text

    UpdateLang(name string, trans string)

.. describe:: name

    Name of the language source.

.. describe:: trans

    Source with translations.


Example
"""""""

.. code:: js

    UpdateLang($Name, $Trans)


.. _simvolio-JSONEncode:

JSONEncode
----------

Converts a number, a string, or an array to a string in JSON format.


Syntax
""""""

.. code-block:: text

    JSONEncode(src int|float|string|map|array) string


.. describe:: src

    Data to convert.


Example
"""""""

.. code:: js

    var mydata map
    mydata["key"] = 1
    var json string
    json = JSONEncode(mydata)


.. _simvolio-JSONEncodeIndent:

JSONEncodeIndent
----------------

Converts a number, a string, or an array to a string in JSON format with the specified indentation.


Syntax
""""""

.. code-block:: text

    JSONEncodeIndent(src int|float|string|map|array, indent string) string

.. describe:: src

    Data to convert.

.. describe:: indent

    String that will be used as indentation.


Example
"""""""

.. code:: js

    var mydata map
    mydata["key"] = 1
    var json string
    json = JSONEncodeIndent(mydata, "\t")


.. _simvolio-JSONDecode:

JSONDecode
----------

Converts a string in JSON format to a number, a string, or an array.


Syntax
""""""

.. code-block:: text

    JSONDecode(src string) int|float|string|map|array


.. describe:: src

    String with data in JSON format.


Example
"""""""

.. code:: js

    var mydata map
    mydata = JSONDecode(`{"name": "John Smith", "company": "Smith's company"}`)



.. _simvolio-HasPrefix:

HasPrefix
---------

Checks if the string begins with a specified substring.


Syntax
""""""

.. code-block:: text

    HasPrefix(s string, prefix string) bool


.. describe:: s

    A string.

.. describe:: prefix

    Prefix to check.


Return value
""""""""""""

Returns ``true`` if the string begins with a specified substring.


Example
"""""""

.. code:: js

    if HasPrefix($Name, `my`) {
    ...
    }


.. _simvolio-Contains:

Contains
--------

Checks if a string contains the specified substring.


Syntax
""""""

.. code-block:: text

    Contains(s string, substr string) bool


.. describe:: s

    A string.

.. describe:: substr

    A substring.


Return value
""""""""""""

Returns ``true`` if the string *s* containts the *substr* substring.


Example
"""""""

.. code:: js

    if Contains($Name, `my`) {
    ...
    }



.. _simvolio-Replace:

Replace
-------

Replaces in the *s* string all occurrences of the *old* string with the *new* string and returns the result.


Syntax
""""""

.. code-block:: text

    Replace(s string, old string, new string) string

.. describe:: s

    A string.

.. describe:: old

    Searched string.

.. describe:: new

    Replacement string.


Example
"""""""

.. code:: js

    s = Replace($Name, `me`, `you`)



.. _simvolio-Size:

Size
----

Returns the size of the specified string.


Syntax
""""""

.. code-block:: text

    Size(val string) int

.. describe:: val

    A string.


Example
"""""""

.. code:: js

    var len int
    len = Size($Name)


.. _simvolio-Sprintf:

Sprintf
-------

The function creates a string based on the specified template and parameters.

Available wildcards:

    * ``%d`` (number)

    * ``%s`` (string)

    * ``%f`` (float)

    * ``%v`` (for any types)

Syntax
""""""

.. code-block:: text

    Sprintf(pattern string, val ...) string

.. describe:: pattern

    Template for the string.

Example
"""""""

.. code:: js

    out = Sprintf("%s=%d", mypar, 6448)

.. _simvolio-Substr:

Substr
------

Returns the substring obtained from a specified string starting from the offset *offset* (calculating from 0) and with maximum length of *length*.

In case of wrong offsets or length an empty string is returned.

If the sum of the *offset* and *length* is more than string size, then the substring will be returned starting from the offset to the end of the string.


Syntax
""""""

.. code-block:: text

    Substr(s string, offset int, length int) string


.. describe:: val

    A string.

.. describe:: offset

    Offset from the beginning of the string.

.. describe:: length

    Maximum substring length.


Example
"""""""

.. code:: js

    var s string
    s = Substr($Name, 1, 10)


.. _simvolio-ToLower:

ToLower
-------

Returns the specified string in a lower case.

Syntax
""""""

.. code-block:: text

    ToLower(val string) string

.. describe:: val

    Input string.


Example
"""""""

.. code:: js

    val = ToLower(val)


.. _simvolio-ToUpper:

ToUpper
-------

Returns the specified string in an upper case.

Syntax
""""""

.. code-block:: text

    ToUpper(val string) string

.. describe:: val

    Input string.


Example
"""""""

.. code:: js

    val = ToUpper(val)


.. _simvolio-TrimSpace:

TrimSpace
---------

Removes leading and trailing spaces, tab, and newline symbols from a string.

Syntax
""""""

.. code-block:: text

    TrimSpace(val string) string

.. describe:: val

    Input string.


Example
"""""""

.. code:: js

    val = TrimSpace(val)



.. _simvolio-Floor:

Floor
-----

Returns the largest integer smaller than or equal to the specified number. 

Syntax
""""""

.. code-block:: text

    Floor(x float|int|string) int

.. describe:: x
  
    A number.

Example
"""""""

.. code:: js

    val = Floor(5.6) // returns 5


.. _simvolio-Log:

Log
---

Returns the natural logarithm of the specified number. 


Syntax
""""""

.. code-block:: text

    Log(x float|int|string) float

.. describe:: x

    A number.


Example
"""""""

.. code:: js

    val = Log(10)


.. _simvolio-Log10:

Log10
-----

Returns the base-10 logarithm of the specified number.


Syntax
""""""

.. code-block:: text

    Log10(x float|int|string) float

.. describe:: x

    A number.


Example
"""""""

.. code:: js

    val = Log10(100)


.. _simvolio-Pow:

Pow
---

Returns the specified base to the specified power (x\ :sup:`y`).


Syntax
""""""

.. code-block:: text

    Pow(x float|int|string, y float|int|string) float

.. describe:: x

    Base number.

.. describe:: y

    Exponent.


Example
"""""""

.. code:: js

    val = Pow(2, 3)


.. _simvolio-Round:

Round
-----

Returns the value of the specified number rounded to the nearest integer.


Syntax
""""""

.. code-block:: text

    Round(x float|int|string) int

.. describe:: x

    A number.


Example
"""""""

.. code:: js

    val = Round(5.6)


.. _simvolio-Sqrt:

Sqrt
----

Returns the square root of the specified number.

.. code-block:: text

    Sqrt(x float|int|string) float

.. describe:: x

  A number.


Example
"""""""

.. code:: js

    val = Sqrt(225)


.. _simvolio-StringToBytes:

StringToBytes
-------------

Converts a string to *bytes*.


Syntax
""""""

.. code-block:: text

    StringToBytes(src string) bytes

.. describe:: src

    Input string.


Example
"""""""

.. code:: js

    var b bytes
    b = StringToBytes("my string")


.. _simvolio-BytesToString:

BytesToString
-------------

Converts *bytes* to a string.


Syntax
""""""

.. code-block:: text

    BytesToString(src bytes) string

.. describe:: src

    Input string.


Example
"""""""

.. code:: js

    var s string
    s = BytesToString($Bytes)


.. _simvolio-SysParamString:

SysParamString
--------------

Returns the value of the specified system parameter.


Syntax
""""""

.. code-block:: text

    SysParamString(name string) string

.. describe:: name

    Parameter name.


Example
"""""""

.. code:: js

    url = SysParamString(`blockchain_url`)


.. _simvolio-SysParamInt:

SysParamInt
-----------

Returns the value of the specified system parameter in the form of a number.

Syntax
""""""

.. code-block:: text

    SysParamInt(name string) int

.. describe:: name

    System parameter name.


Example
"""""""

.. code:: js

    maxcol = SysParam(`max_columns`)



.. _simvolio-DBUpdateSysParam:

DBUpdateSysParam
----------------

Updates the value and the condition of the system parameter. If you do not need to change the value or condition, then specify an empty string in the corresponding parameter.


Syntax
""""""

.. code-block:: text

    DBUpdateSysParam(name, value, conditions string)

.. describe:: name

    System parameter name.

.. describe:: value

    New value for the parameter.

.. describe:: conditions

    New condition for changing the parameter.


Example
"""""""

.. code:: js

    DBUpdateSysParam(`fuel_rate`, `400000000000`, ``)


.. _simvolio-UpdateNotifications:

UpdateNotifications
-------------------

Obtains a list of notifications for the specified keys from the database, and sends the obtained notifications to Centrifugo.


Syntax
""""""

.. code-block:: text

    UpdateNotifications(ecosystemID int, keys int ...)


.. describe:: ecosystemID

    Ecosystem identifier.

.. describe:: key

    List of keys, separated by commas. As an alternative, you can specify one array with a list of keys.


Example
"""""""

.. code:: js

    UpdateNotifications($ecosystem_id, $key_id, 23345355454, 35545454554)
    UpdateNotifications(1, [$key_id, 23345355454, 35545454554] )


.. _simvolio-UpdateRolesNotifications:

UpdateRolesNotifications
------------------------

Obtains a list of notifications for all keys with specified role identifiers from the database, and sends the obtained notifications to Centrifugo.


Syntax
""""""

.. code-block:: text

    UpdateRolesNotifications(ecosystemID int, roles int ...)

.. describe:: ecosystemID

    Ecosystem identifier.

.. describe:: roles

    List of role identifiers, separated by commas. As an alternative, you can specify one array with a list of keys.


Example
"""""""

.. code:: js

    UpdateRolesNotifications(1, 1, 2)



.. _simvolio-HTTPRequest:

HTTPRequest
-----------

Sends an HTTP request to a specified address.

.. note::

    This function can be used only in Off-Blockchain Server (OBS) contracts.


Syntax
""""""

.. code-block:: text

    HTTPRequest(url string, method string, heads map, pars map) string


.. describe:: url

    Address, to which the request will be sent.

.. describe:: method

    Request type (GET or POST)

.. describe:: heads

    An array of data for header formation.

.. describe:: pars

    Parameters.

    .. todo::

        Explain above better.


Example
"""""""

.. code:: js

    var ret string
    var pars, heads, json map
    heads["Authorization"] = "Bearer " + $auth_token
    pars["obs"] = "true"
    ret = HTTPRequest("http://localhost:7079/api/v2/content/page/default_page", "POST", heads, pars)
    json = JSONToMap(ret)


.. _simvolio-HTTPPostJSON:

HTTPPostJSON
------------

This function is similar to the *HTTPRequest* function, but it sends a *POST* request and parameters are passed in one string.

.. note::

    This function can be used only in Off-Blockchain Server (OBS) contracts.


Syntax
""""""

.. code-block:: text

    HTTPPostJSON(url string, heads map, pars string) string


.. describe:: url

    Address, to which the request will be sent.

.. describe:: heads

    Data array for header formation.

.. describe:: pars

    Parameters as a json string.


Example
"""""""

.. code:: js

    var ret string
    var heads, json map
    heads["Authorization"] = "Bearer " + $auth_token
    ret = HTTPPostJSON("http://localhost:7079/api/v2/content/page/default_page", heads, `{"obs":"true"}`)
    json = JSONToMap(ret)



.. _simvolio-BlockTime:

BlockTime
---------

Returns generation time of a block in SQL format.

Use this function instead of the ``NOW()`` function.

Syntax
""""""

.. code-block:: text

    BlockTime()

Example
"""""""

.. code:: js

    DBInsert(`mytable`, `created_at`, BlockTime())


.. _simvolio-DateTime:

DateTime
--------

Converts unixtime to a string in the `YYYY-MM-DD HH\:MI\:SS` format.


Syntax
""""""

.. code-block:: text

    DateTime(unixtime int) string


Example
"""""""

.. code:: js

    DateTime(1532325250)


.. _simvolio-UnixDateTime:

UnixDateTime
------------

Converts a string in the `YYYY-MM-DD HH\:MI\:SS` format to unixtime.


Syntax
""""""

.. code-block:: text

    UnixDateTime(datetime string) int

Example
"""""""

.. code:: js

    UnixDateTime("2018-07-20 14:23:10")



.. _simvolio-CreateOBS:

CreateOBS
---------

Creates a child OBS (Off-Blockchain Server).

.. todo::

  Check that it works as described.

Syntax
""""""

.. code-block:: text

    CreateOBS(OBSName string, DBUser string, DBPassword string, OBSAPIPort int)

.. describe:: OBSName

    Name for the created OBS.

.. describe:: DBUser

    Role name for the database.

.. describe:: DBPassword

    Password for this role.

.. describe:: OBSAPIPort

    Port for API requests.

.. todo::

    Add example.


.. _simvolio-ListOBS:

ListOBS
-------

Returns a list of child OBSs (Off-Blockchain Servers).

This function can be used only in OBS Master mode.

.. todo::

  Check that it works as described.

Syntax
""""""

.. code-block:: text

    ListOBS()

Returned value
""""""""""""""

An associative array where keys are OBS names and values are process statuses.

.. todo::

    Add example.


.. _simvolio-RunOBS:

RunOBS
------

Runs a process for an OBS.

This function can be used only in OBS Master mode.

.. todo::

  Check that it works as described.

Syntax
""""""

.. code-block:: text

    RunOBS(OBSName string)

.. describe:: OBSName

    Name for an OBS.

    Can contain only letters and numbers. Space symbols cannot be used.

.. todo::

    Add example.


.. _simvolio-StopOBS:

StopOBS
-------

Stops the process of a specified OBS.

This function can be used only in OBS Master mode.


Syntax
""""""

.. code-block:: text

    StopOBS(OBSName string)

.. describe:: OBSName

    Name for a OBS.

    Can contain only letters and numbers. Space symbols cannot be used.

.. todo::

    Add example.


.. _simvolio-RemoveOBS:

RemoveOBS
---------

Removes the process of the specified OBS. Stops and removes linked processes.

.. todo::

    Clarify about linked processes. Check that it works as described.

This function can be used only in OBS Master mode.

Syntax
""""""

.. code-block:: text

    RemoveOBS(OBSName string)

.. describe:: OBSName

    Name for a OBS.

    Can contain only letters and numbers. Space symbols cannot be used.

.. todo::

    Add example.

System Contracts
================

System contracts are created by default during product installation. All of these contracts are created in the first ecosystem, that's why you need to specify their full name to call them from other ecosystems, for instance, ``@1NewContract``.


NewEcosystem
------------

This contract creates a new ecosystem. To get an identifier of the newly created ecosystem, take the *result* field, which will return in txstatus.

Parameters:

    * *Name string "optional"* - name for the ecosystem. This parameter can be set and/or chanted later.


EditEcosystemName
-----------------

This contract changes ecosystem name in the *1_ecosystems* table. This table exists only in the first ecosystem.

Parameters:

    * *SystemID* - код экосистемы, имя которой требуется изменить
    * *NewName* - новое имя экосистемы


MoneyTransfer
-------------

This contract transfers money from the current account in the current ecosystem to a specified account.

Parameters:

    * *Recipient string* - recipient's account in any format – a number or ``XXXX-....-XXXX``.
    * *Amount    string* - transaction amount in qAPL.
    * *Comment   string "optional"* - comments.


NewContract
-----------

This contract creates a new contract in the current ecosystem.

Parameters:

    * *Value string* - text of the contract, there must be only one contract on the upper level.
    * *Conditions string* - contract change conditions.
    * *Wallet string "optional"* - identifier of user's id where contract must be tied.
    * *TokenEcosystem int "optional"* - identifier of the ecosystem, which currency will be used for transactions when the contract is activated.


EditContract
------------

Edits the contract in the current ecosystem.

Parameters:

    * *Id int* - ID of the contract to be edited.
    * *Value string "optional"* - text of the contract or contracts.
    * *Conditions string "optional"* - rights for contract change.


BindWallet
----------

Binds a contract to an account in the current ecosystem. Contracts can be bound only from the account that was specified when the contract was created. After the contract is bound to an account, this account will pay for the execution of this contract.

Parameters:

    * *Id int* - identifier of the contract to bind.


UnbindWallet
------------

Unbinds a contract from an account in the current ecosystem. Only the account that is bound to the contract can unbind it. After the contract is unbound, a user that executes the contract will pay for the execution.

Parameters:

    * *Id int* - identifier of the bound contract.


NewParameter
------------

This contract adds a new parameter to the current ecosystem.

Parameters:

    * *Name string* - parameter name.
    * *Value string* - parameter value.
    * *Conditions string* - rights for parameter change.


EditParameter
-------------

This contract changes an existing parameter in the current ecosystem.

Parameters:

    * *Name string* - name of the parameter to be changed.
    * *Value string* - new value.
    * *Conditions string* - new condition for parameter change.


NewMenu
-------

This contract adds a new menu in the current ecosystem.

Parameters:

    * *Name string* - menu name.
    * *Value string* - menu text.
    * *Title string "optional"* - menu header.
    * *Conditions string* - rights for menu change.


EditMenu
--------

This contract changes an existing menu in the current ecosystem.

Parameters:

    * *Id int* - ID of the menu to be changed.
    * *Value string "optional"* - new text of menu.
    * *Title string "optional"* - menu header.
    * *Conditions string "optional"* - new rights for page change.


AppendMenu
----------

This contract adds text to an existing menu in the current ecosystem.

Parameters:

    * *Id int* - complemented menu identifier.
    * *Value string* - text to be added.


NewPage
-------

This contract adds a new page in the current ecosystem.

Parameters:

    * *Name string* - page name.
    * *Value string* - page text.
    * *Menu string* - name of the menu, attached to this page.
    * *Conditions string* - rights for changing this page.
    * *ValidateCount int "optional"* - number of nodes that is required for page validation. If this parameter is not specified, then *min_page_validate_count* ecosystem parameter value is used. This value cannot be less than *min_page_validate_count* and greater than *max_page_validate_count*.
    * *ValidateMode int "optional"* - mode of page validity checks. A value of 0 means that a page is checked upon loading the page. A value of 1 means that a page is checked upon loading and leaving the page.


EditPage
--------

This contract changes an existing page in the current ecosystem.

Parameters:

    * *Id int* - ID of the page to be changed.
    * *Value string "optional"* - new text of the page.
    * *Menu string "optional"* - name of the new menu on the page.
    * *Conditions string "optional"* - new rights for page change.
    * *ValidateCount int "optional"* - number of nodes that is required for page validation. If this parameter is not specified, then *min_page_validate_count* ecosystem parameter value is used. This value cannot be less than *min_page_validate_count* and greater than *max_page_validate_count*.
    * *ValidateMode string "optional"* - mode of page validity checks. A value of 0 means that a page is checked upon loading the page. A value of 1 means that a page is checked upon loading and leaving the page.

AppendPage
----------

The contract adds text to an existing page in the current ecosystem.

Parameters:

    * *Id int* - ID of the page to be changed.
    * *Value string* - text that needs to be added to the page.


NewBlock
--------

This contract adds a new page block with a template to the current ecosystem.

Parameters:

    * *Name string* - block name.
    * *Value string* - block text.
    * *Conditions string* - rights for block change.


EditBlock
---------

This contract changes an existing block in the current ecosystem.

Parameters

    * *Id int* - ID of the block to be changed.
    * *Value string* - new text of a block.
    * *Conditions string* - new rights for change.


NewTable
--------

This contract adds a new table in the current ecosystem.

Parameters:

    * *Name string* - table name in Latin script.
    * *Columns string* - array of columns in JSON format ``[{"name":"...", "type":"...","index": "0", "conditions":"..."},...]``, where

        * *name* - column name in Latin script.
        * *type* - type ``varchar,bytea,number,datetime,money,text,double,character``.
        * *index* - non-indexed field - "0"; create index - "1".
        * *conditions* - condition for changing data in a column; read access rights must be specified in the JSON format. For example, ``{"update":"ContractConditions(`MainCondition`)", "read":"ContractConditions(`MainCondition`)"}``


    * *Permissions string* - access conditions in JSON format ``{"insert": "...", "new_column": "...", "update": "..."}``.

        * *insert* - rights to insert records.
        * *new_column* - rights to add columns.
        * *update* - rights to change rights.


EditTable
---------

This contract changes access permissions to tables in the current ecosystem.

Parameters:

    * *Name string* - table name.
    * *Permissions string* - access permissions in JSON format ``{"insert": "...", "new_column": "...", "update": "..."}``.

        * *insert* - condition to insert records.
        * *new_column* - condition to add columns.
        * *update* - condition to change data.


NewColumn
---------

This contract adds a new column to a table in the current ecosystem.

Parameters:

    * *TableName string* - table name in.
    * *Name* - column name in Latin script.
    * *Type* - type ``varchar,bytea,number,money,datetime,text,double,character``.
    * *Index* - non-indexed field - "0"; create index - "1".
    * *Permissions* - condition for changing data in a column; read access rights must be specified in the JSON format. For example, ``{"update":"ContractConditions(`MainCondition`)", "read":"ContractConditions(`MainCondition`)"}``


EditColumn
----------

This contract changes the rights to change a table column in the current ecosystem.

Parameters:

    * *TableName string* - table name in Latin script.
    * *Name* - column name in Latin script.
    * *Permissions* - condition for changing data in a column; read access rights must be specified in the JSON format. For example, ``{"update":"ContractConditions(`MainCondition`)", "read":"ContractConditions(`MainCondition`)"}``.

NewLang
-------

This contract adds language resources in the current ecosystem. Permissions to add resources are set in the *changing_language* parameter in the ecosystem configuration.

Parameters:

    * *Name string* - name of the language resource in Latin script.
    * *Trans* - language resources as a string in JSON format with two-character language codes as keys and translated strings as values. For example: ``{"en": "English text", "ru": "Английский текст"}``.
    * *[Lang string]* - optional parameter that specifies the language for error messages generated during the contract execution.


EditLang
--------

This contract updates the language resource in the current ecosystem. Permissions to make changes are set in the *changing_language* parameter in the ecosystem configuration.

Parameters:

    * *Id int*- language resource ID.
    * *Name string* - name of the language resource.
    * *Trans* - language resources as a string in JSON format with two-character language codes as keys and translated strings as values. For example ``{"en": "English text", "ru": "Английский текст"}``.
    * *[Lang string]* - optional parameter that specifies the language for error messages generated during the contract execution.


NewSign
-------

This contract adds the signature confirmation requirement for a contract in the current ecosystem.

Parameters:

    * *Name string* - name of the contract, where an additional signature confirmation will be required.
    * *Value string* - description of parameters in a JSON string, where

      * *title* - message text.
      * *params* - array of parameters that are displayed to users, where **name** is the field name, and **text** is the parameter description.

    * *Conditions string* - condition for changing the parameters.

Example of *Value*:

``{"title": "Would you like to sign?", "params":[{"name": "Recipient", "text": "Wallet"},{"name": "Amount", "text": "Amount(EGS)"}]}``


EditSign
--------

The contract updates the parameters of a contract with a signature in the current ecosystem.

Parameters:

     * *Id int* - identifier of the signature to be changed.
     * *Value string* - a string containing new parameters.
     * *Conditions string* - new condition for changing the signature parameters.


Import
------

This contract imports data from a \*.sim file into the ecosystem.

Parameters:

    * *Data string* - data to be imported in text format; this data is the result of export from an ecosystem to a .sim file.


NewCron
-------

The contract adds a new task in cron to be launched by timer. The contract is available only in OBS. 

Parameters:

    * *Cron string* - a string that defines the launch of the contract by timer in the *cron* format.
    * *Contract string* - name of the contract to launch in OBS; the contract must not have parameters in its ``data`` section.
    * *Limit int* - an optional field, where the number of contract launches can be specified (until contract is executed this number of times).
    * *Till string* - an optional string with the time when the task must be ended (this feature is not yet implemented).
    * *Conditions string* - rights to modify the task.


EditCron
--------

This contract changes the configuration of a task in cron for launch by timer. The contract is available only in OBS. 

Parameters:

    * *Id int* - task ID.
    * *Cron string* - a string that defines the launch of the contract by timer in the *cron* format; to disable a task, this parameter must be either an empty string or absent.
    * *Contract string* - name of the contract to launch in OBS; the contract must not have parameters in its data section.
    * *Limit int* - an optional field, where the number of contract launches can be specified (until contract is executed this number of times).
    * *Till string* - an optional string with the time of task must be ended (this feature is not yet implemented).
    * *Conditions string* - new rights to modify the task.

NewAppParam
-----------

Adds a new app parameter to the current ecosystem.

Parameters:

    * *App int* - application identifier.
    * *Name string* - parameter name.
    * *Value string* - parameter value.
    * *Conditions string* - rights to change the parameter.


EditAppParam
------------

Changes an existing app parameter in the current ecosystem.

Parameters:

    * *Id int* - parameter indentifier.
    * *Value string* - new value for the parameter.
    * *Conditions string* - new rights to change the parameter.

NewDelayedContract
------------------

Adds a new task to the delayed contract scheduler.

The delayed contracts scheduler runs contracts that are required for the currently generated block.

Parameters:

    * *Contract string* - contract name.
    * *EveryBlock int* - the contract will be executed every this amount of blocks.
    * *Conditions string* - rights for changing the task.
    * *BlockID int "optional"* - block number where the contract must be executed. If this value is not specified, then it is calculated automatically by adding *EveryBlock* to the current block number.
    * *Limit int "optional"* - maximum number of task executions. If this value is not specified, then the task will be executed for an unlimited amount of time.

EditDelayedContract
-------------------

Changes a task in the delayed contract scheduler.

Parameters:

    * *Id int* - task identifier.
    * *Contract string* - contract name.
    * *EveryBlock int* - the contract will be executed every this amount of blocks.
    * *Conditions string* - rights for changing the task.
    * *BlockID int "optional"* - block number where the contract must be executed. If this value is not specified, then it is calculated automatically by adding *EveryBlock* to the current block number.
    * *Limit int "optional"* - maximum number of task executions. If this value is not specified, then the task will be executed for an unlimited amount of time.
    * *Deleted int "optional"* - task toggle. A value of *1* disables the task. A value of *0* enables the task.


UploadBinary
------------

The contract adds/rewrites a static file in X_binaries. When calling a contract via HTTP API, ``multipart/form-data`` must be used; the ``DataMimeType`` parameter will be used with the form data.

Parameters:

    * *Name string* - static file name.
    * *Data bytes "file"* - content of the static file.
    * *DataMimeType string "optional"* - mime type of the static file.
    * *AppID int* - application identifier.
    * *MemberID int "optional"* - ecosystem member identifier. Is 0 by default.

If the DataMimeType is not passed, then ``application/octet-stream`` is used by default.
If MemberID is not passed, then the static file is considered a system file.
