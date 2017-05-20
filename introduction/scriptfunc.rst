################################################################################
Embedded contract language functions
################################################################################

The functions of a contract language perform operations with data obtained in the data section of the contract. They can read values from and write values to the database, they can convert value types, and establish relationship between contracts.

Functions do not return errors – all errors are checked automatically. When an error is generated in any of the functions, the contract stops working and displays a description of the error in a special window.



********************************************************************************
Retrieving values from the database
********************************************************************************

DBAmount(tblname string, column string, id int) money
==============================
The function returns the value of the **amount** column with type *money* with a search for a record by value of a specified column of the table. (The functions **DBInt()** and **DBIntExt()** that return int values cannot be used to obtain money *int*).

* *tblname* - name of the table in the database;
* *column* - name of the column where the record will be searched for;
* *id* - value for record search, *column=id* selection.

.. code:: js

    mymoney = DBAmount("dlt_wallets"), "wallet_id", $wallet)
    
StateVal(name string) string
==============================
The function returns the value of a specified parameter from the state settings (*state_parameters*).

* *name* - name of the received parameter.

.. code:: js

    Println( StateVal("gov_account"))


DBInt(tblname string, name string, id int) int
==============================
The function returns a numeric value from the database table by a specified **id** of the record.

* *tblname*  – name of the table in the database;
* *name* - name of the column whose value will be returned.
* *id*- identifier of the record **id** field from where the value will be taken.

.. code:: js

    var val int
    val = DBInt(Table("mytable"), "counter", 1)

DBIntExt(tblname string, name string, val (int|string), column string) int
==============================
The function returns a numeric value from the database table, with a search of the record by a specified field and value.

* *tblname*  – name of the table in the database
* *name* - name of the column whose value will be returned.
* *val* - value by which the record will be searched.
* *column* - name of the column where the record will be searched for; the table must have an index for this column.

.. code:: js

    var val int
    val = DBIntExt(Table("mytable"), "balance", $wallet, "wallet_id")

DBIntWhere(tblname string, name string, where string, params ...) int
==============================
The function returns a numeric value from the column of the database table with a search for the record by the conditions specified in **where**.

* *tblname*  – name of the table in the database
* *name* - name of the column whose value will be returned.
* *where* - query conditions for retrieving records; names of the fields are located to the left of the comparison signs; the characters **?** or **$** are used are used for substitution of parameters.
* *params* - parameters that are substituted in query conditions in a given sequence.

.. code:: js

    var val int
    val = DBIntWhere(Table("mytable"), "counter",  "idgroup = ? and statue=?", mygroup, 1 )

DBString(tblname string, name string, id int) string
==============================
The function returns a string value from the column of the database table by the record **id**.

* *tblname*  – name of the table in the database
* *name* - name of the column whose value will be returned.
* *id* - identifier of the record **id** field from where the value will be taken.

.. code:: js

    var val string
    val = DBString(Table("mytable"), "name", $citizen)

DBStringExt(tblname string, name string, val (int|string), column string) string
==============================
The function returns a string value from the database table, with a search of the record by a specified field and value.

* *tblname*  – name of the table in the database
* *name* - name of the column whose value will be returned.
* *val* - value by which the record will be searched;
* *column* - name of the column where the record will be searched for; the table must have an index for this column.

.. code:: js

    var val string
    val = DBStringExt(Table("mytable"), "address", $Company, "company" )
    
DBFreeRequest(tblname string, val (int|string), column string)
==============================
The function checks for the presence of a specified record and has a zero execution cost. It is intended for preliminary verification of contract parameters for the purpose of protecting against “spam”. This function can only be called once in the contract. If a record with this column value is found, then the contract will continue its work. Otherwise, this function will generate an error.

* *tblname*  – name of the table in the database
* *val* - value by which the record will be searched;
* *column* - name of the column where the record will be searched for; the table must have an index for this column.

DBStringWhere(tblname string, name string, where string, params ...) string
==============================
The function returns a string value from the column of the database table, with a search of the record according to conditions specified in *where*.

* *tblname*  – name of the table in the database
* *name* - name of the column whose value will be returned
* *where* - query conditions for retrieving records; the names of the fields are located to the left of the comparison signs; the characters **?** or **$** are used are used for substitution of parameters.
* *params* - parameters that are substituted in query conditions in a given sequence.

.. code:: js

    var val string
    val = DBStringWhere(Table("mytable"), "address",  "idgroup = ? and company=?",
           mygroup, "My company" )

DBGetList(tblname string, column string, offset int, limit int, order string, where string, params ...) array
==============================
The function returns a string value from the column of the database table, with a search of the record according to the conditions specified in **where**.

* *tblname*  – name of the table in the database
* *column*  - name of the column from where the values will be taken;
* *offset* - offset to start selection of records;
* *limit*  - number of records received; if limit is not needed, then the parameter value is **-1**;
* *order* - sorting by columns; can be an empty string;
* *where* - query conditions for retrieving records; the names of the fields are located to the left of the comparison signs; the characters **?** or **$** are used are used for substitution of parameters;
* *params* - parameters that are substituted in query conditions in a given sequence.

.. code:: js

    var ret array
    ret = DBGetList(Table("mytable"), "name", 0, -1, "", "idval > ? and idval <= ? and company=?", 
                     10, 200, "My company")
                     
                     
DBGetTable(tblname string, columns string, offset int, limit int, order string, where string, params ...) array
==============================
The function returns associative map arrays, containing a list of values of the listed columns of table records obtained by the conditions specified in **where**. All values in the associative array are of the type **string**, so they should be subsequently brought to the appropriate type.

* *tblname*  – name of the table in the database
* *columns* - names of received columns separated by comma;
* *offset* - offset to start selection of records;
* *limit*  - number of records received; if limit is not needed, then the parameter value is **-1**;
* *order* - sorting by columns; can be an empty string;
* *where* - query conditions for retrieving records; the names of the fields are located to the left of the comparison signs; the characters **?** or **$** are used are used for substitution of parameters;
* *params* - parameters that are substituted in query conditions in a given sequence.

.. code:: js

    var ret array
    ret = DBGetTable(Table("mytable"), "name,idval,company", 0, -1, "", "idval > ? and idval <= ? and company=?",
                     10, 200, "My company")
    var i int
    while i<Len(ret) {
        var row map
    
        row = ret[i]
        myfunc(Sprintf("%s %s", row["name"], row["company"]), Int(row["idval"]) )
        i++
    }
	
********************************************************************************
Changing values in tables
********************************************************************************

DBInsert(tblname string, params string, val ...) int
==============================
The function adds a record to a specified table and returns the **id** of the inserted record.

* *tblname*  – name of the table in the database.
* *params* - list of comma-separated names of columns, where the values listed in **val** will be written.
* *val* - list of comma-separated values for the columns listed in **params**; values can be a string or a number.

.. code:: js

    DBInsert(Table("mytable"), "name,amount", "John Dow", 100)


DBUpdate(tblname string, id int, params string, val...)
==============================
The function changes the column values in the table in the record with a specified **id**.

* *tblname*  – name of the table in the database.
* *id* - identifier **id** of the changeable record.
* *params* - list of comma-separated names of the columns to be changed.
* *val* - list of values for a specified columns listed in **params**; can either be a string or a number.

.. code:: js

    DBUpdate(Table("mytable"), myid, "name,amount", "John Dow", 100)

DBUpdateExt(tblname string, column string, value (int|string), params string, val ...)
==============================
The function updates columns in a record whose column has a specified value. The table should have an index for a specified column.

* *tblname*  – name of the table in the database.
* *column*  - name of the column by which the record will be searched for.
* *value* - value for searching a record in a column.
* *params* - list of comma-separated names of columns, where the values specified in **val** will be written.
* *val* - list of values for recording in the columns listed in **params**; can either be a string or a number.

.. code:: js

    DBUpdateExt(Table("mytable"), "address", addr, "name,amount", "John Dow", 100)

********************************************************************************
Calling of contracts
********************************************************************************

CallContract(name string, params map)
==============================
The function calls a contract by its name. All the parameters specified in the section data of the contract should be listed in the transmitted array.

* *name*  - name of the contract being called.
* *params* - an associative array with input data for the contract.

.. code:: js

    var par map
    par["Name"] = "My Name"
    CallContract("MyContract", par)

ContractAccess(name string, [name string]) bool
==============================
The function checks whether the name of the executed contract matches with one of the names listed in the parameters. Typically used to control access of contracts to tables. The function is specified in the *Permissions* fields when editing table columns or in the *Insert* and *New Column* fields in the *Table permission*. section.

* *name* – contract name.

.. code:: js

    ContractAccess("MyContract")  
    ContractAccess("MyContract","SimpleContract") 
    
ContractConditions(name string, [name string]) bool
==============================
The function calls the **conditions** section from contracts with specified names. For such contracts, the *data* block must be empty. If the conditions *conditions* is executed without errors, then *true* is returned. If an error is generated during execution, the parent contract will also end with this error. This function is usually used to control access of contracts to tables and can be called in the *Permissions* fields when editing system table.

* *name* – contract name.

.. code:: js

    ContractConditions("MainCondition")  

********************************************************************************
Operations with values of variables
********************************************************************************

AddressToId(val int) int
==============================


Float(val int|string) float
==============================
The function converts an integer *int* or *string* to a floating-point number.

* *val* - an integer or string.

.. code:: js

    val = Float("567.989") + Float(232)

HexToBytes(hexdata string) bytes
==============================
The function converts a string with hexadecimal encoding to a *bytes* value (sequence of bytes).

* *hexdata* – a string containing a hexadecimal notation.

.. code:: js

    var val bytes
    val = HexToBytes("34fe4501a4d80094")

Int(val string) int
==============================
The function converts a string value to an integer.

* *val*  – a string containing a number.

.. code:: js

    mystr = "-37763499007332"
    val = Int(mystr)

Len(val array) int
==============================
The function returns the number of elements in a specified array.

* *val* – *array*.

.. code:: js

    if Len(mylist) == 0 {
      ...
    }

PubToID(hexkey string) int
==============================
The function returns the wallet address by the public key in hexadecimal encoding.

* *hexkey* - public key in hexadecimal form.

.. code:: js

    var wallet int
    wallet = PubToID("fa5e78.....34abd6")
    
 AddressToId(address string) int
 ==============================
 The function returns the citizen's identification number by the string value of the address of his wallet.
 
 Sha256(val string) string
==============================
The function returns **SHA256** hash of a specified string.

* *val* - incoming line for which the **Sha256** hash should be calculated.

.. code:: js

    var sha string
    sha = Sha256("Test message")

Sprintf(pattern string, val ...) string
==============================
The function forms a string based on specified template and parameters, you can use *%d (number), %s (string), %f (float), %v* (for any types).

* *pattern*  - a template for forming a string.

.. code:: js

    out = Sprintf("%s=%d", mypar, 6448)



Str(val int|float) string
==============================
The function converts a numeric *int* or *float* value to a string.

* *val* - an integer or a floating-point number.

.. code:: js

    myfloat = 5.678
    val = Str(myfloat)

Table(tblname) string
==============================
The function returns the full name of a table with the numeric prefix of the state number in which the contract is called and with an underscore symbol between the prefix and the name. Allows you to make contracts become independent of the state.

* *tblname* - part of the name of the table in the database after the underscore symbol.

.. code:: js

    Println( Table("citizens")) // may be 1_citizens or 2_citizens etc.

********************************************************************************
Updating platform elements
********************************************************************************

UpdateContract(name string, value string, conditions string)
==============================
The function updates a specified contract (a contract cannot be modified via the **DBUpdate** function).

* *name* - contract name;
* *value* - contract text;
* *conditions* - access rights to modify a contract.

.. code:: js

    UpdateContract("MyContract", source, "СonditionsContract($citizen)")

UpdateMenu(name string, value string, conditions string)
==============================
The function updates a specified menu (the menu cannot be changed via the DBUpdate function).

* *name* - name of the menu to be updated.
* *value* - menu text.
* *conditions* - access rights to change the menu.

.. code:: js

    UpdateMenu("main_menu", mymenu, "СonditionsContract($citizen)")

UpdatePage(name string, value string, menu string, conditions string)
==============================
The function updates a specified page (the page cannot be changed via the **DBUpdate** function).

* *name*  - name of the page being updated;
* *value* - text of the page;
* *menu* - name of the menu bound to the page;
* *conditions* - access rights to change the page.

.. code:: js

    UpdatePage("default_dashboard",mypage, "main_menu", "СonditionsContract($citizen)")

UpdateParam(name string, value string, conditions string)
==============================
The function updates the *state_parameters* in the state_parameters table (parameters cannot be changed through the DBUpdate **DBUpdate**).

* *name* - parameter name;
* *value* - parameter value;
* *conditions* - access rights to change the parameter.

.. code:: js

    UpdateParam("state_flag", $flag, "ContractConditions(`MainCondition`)")
