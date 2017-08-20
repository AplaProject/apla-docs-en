################################################################################
Embedded contract language functions
################################################################################

The functions of a contract language perform operations with data obtained in the data section of the contract. They can read values from and write values to the database, they can convert value types, and establish relationship between contracts.

Functions do not return errors – all errors are checked automatically. When an error is generated in any of the functions, the contract stops working and displays a description of the error in a special window.


********************************************************************************
Predefined values
********************************************************************************

The following variables are available when executing a contract. 

* **$citizen** - numeric identifier (int64) of the citizen, who signed the transaction. 
* **$wallet** - numeric identifier (int64) of the wallet. Matches with *$citizen*. 
* **$state** - identifier of the ecosystem of which the user, who signed the transaction, is a member. 
* **$type** identifier of an external contract from where the current contract was called. 
* **$time** - time specified in the transaction in Unix format. 
* **$block** - block number in which this transaction is sealed. 
* **$block_time** - time specified in the block. 
* **$wallet_block** - numeric identifier (int64) of the node that signed the block. 

It should be kept in mind that these variables are available not only in the functions of the contract but also in other functions and expressions, for example, in conditions that are specified for contracts, pages and other objects. In this case, *$time and $block* variables related to the block and others are equal to 0.

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

DBRowExt(tblname string, columns string, val (int|string), column string) map
==============================
Function returns a massive of values (map) from database table with the search of record on the specified field and value.

* *tblname* - table name in the database;
* *columns* - the name of the columns you want to retrieve;
* *val* - the value by which the record will be searched;
* *column* - the name of the column on which the record will be searched. The table should have an index for this column.

.. code:: js

    var vals map
    vals = DBRowExt(Table("mytable"), "address,postindex,name", $Company, "company" )

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
	
LangRes(idres string, lang string) string
==============================
The function returns a language resource named idres for a language specified in lang as a two-character code, for example, *en,fr,ru*. The function searches in the corresponding ecosystem. If there is no resource for such language, the English language resource will be returned. 

* *idres* - имя языкового ресурса;
* *lang* - двухсимвольный код языка;

.. code:: js

    warning LangRes("confirm", $Lang)
    error LangRes("problems", "de")
    
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

DBInsertReport(tblname string, params string, val ...) int
==============================
The function adds an entry to the specified report table and returns **id** of the inserted entry. This function is almost identical to the DBInsert function, but it makes an entry only to the report table of the current ecosystem. 

* *tblname* – the name of the table in the database. The report table in the database must have a name in the format **[state_id]_reports_[tblname]**. 
* *params* – a comma-separated list of column names where the values listed in **val** will be written. 
* *val* – a comma-separated list of values for columns listed in **params**; values can be a string or a number. 

.. code:: js

    DBInsertReport(Table("mytable"), "name,amount", "John Dow", 100)

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

FindEcosystem(name string) int
==============================
The function looks for an ecosystem with the specified name and returns its identifier. If the indicated ecosystem is absent, then it returns identifier. The search is going without register accounting.

* *name* - ecosystem name.

.. code:: js

    id = FindEcosystem(`My Country`)

********************************************************************************
Work with contracts and language
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

EvalCondition(tablename string, name string, condfield string) 
==============================
Function takes from the *tablename* table the value of the *condfield* field from the record with the *’name’* field, which is equal to the *name* parameter and checks if the condition from the field *condfield* is made. 

* *tablename* - name of the table.
* *name* - value for searching by the field 'name'
* *condfield* - the name of the field where the condition to be checked is stored.

.. code:: js

    EvalCondition(PrefixTable(`menu`, $Global), $Name, `condition`)  

ValidateCondition(condition string, state int) 
==============================
The function tries to compile the condition specified in the *condition* parameter. If a mistake occurs during the compilation process, the mistake will be generated and the calling contract will complete is’s job. This function is designed to check the correctness of the conditions when they change.

* *condition* - verifiable condition.
* *state* - identifier of the state. Specifie 0 if checking for global conditions

.. code:: js

    ValidateCondition(`ContractAccess("@0MyContract")`, 0)
    
********************************************************************************
Operations with values of variables
********************************************************************************

AddressToId(address string) int
==============================
Function returns the the identification number of the citizen by the string value of the address of his wallet. If the wrong adress is specified, then 0 returns. 

* *address* - the wallet adress in the format XXXX-...-XXXX or in the form of number.

.. code:: js

    wallet = AddressToId($Recipient)
    
Contains(s string, substr string) bool
==============================
Returnes true if the string *s* containts the substring *substr*.

* *s* - checked string.
* *substr* - which is searched in the specified line.

.. code:: js

    if Contains($Name, `my`) {
    ...
    }    

Float(val int|string) float
==============================
The function converts an integer *int* or *string* to a floating-point number.

* *val* - an integer or string.

.. code:: js

    val = Float("567.989") + Float(232)

HasPrefix(s string, prefix string) bool
==============================
Function returns true, if the string bigins from the specified substring *prefix*.

* *s* - checked string.
* *prefix* - checked prefix for this string.

.. code:: js

    if HasPrefix($Name, `my`) {
    ...
    }

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
    
PrefixTable(tblname string, global int) string
==============================
The function returns the full name of the table with the numeric prefix of the state number, where the contract is calling, or with prefix **global**, if the value of *global* parameter is equel 1.

* *tblname* - the part of the table name in the database after the underscore character.
* *global* - if it is equel to 1, the prefix **global** will be added

.. code:: js

    Println( PrefixTable("pages", global)) // may be global_pages, 1_pages or 2_pages etc.

Replace(s string, old string, new string) string
============================
Function replaces in the *s* string all cccurrences of the *old* string to *new* string and returnes the result.  

* *s* - source string.
* *old* - changed string.
* *new* - new string.

.. code:: js

    s = Replace($Name, `me`, `you`)
 
Size(val string) int
==============================
The function returns the size of the specified string.

* *val* - the string for which we have to calculate the size.

.. code:: js

    var len int
    len = Size($Name) 
 
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

Substr(s string, offset int, length int) string
==============================
Function returns the substring from the specified string starting from the offset *offset* (calculating from the 0) and with length *length*. In case of not correct offsets or length the empty column is returned. If the sum of offset and *length* is more than string size, then the substring will be returned from the offset to the end of the string.

* *val* - string.
* *offset* - offset of substring.
* *length* - size of substring.

.. code:: js

    var s string
    s = Substr($Name, 1, 10)

Table(tblname) string
==============================
The function returns the full name of a table with the numeric prefix of the state number in which the contract is called and with an underscore symbol between the prefix and the name. Allows you to make contracts become independent of the state.

* *tblname* - part of the name of the table in the database after the underscore symbol.

.. code:: js

    Println( Table("citizens")) // may be 1_citizens or 2_citizens etc.

UpdateLang(name string, trans string)
==============================
Function updates the language source in the memory. Is used in the transactions that change language sources.

* *name* - name of the language source.
* *trans* - source with translations.

.. code:: js

    UpdateLang($Name, $Trans)


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
    
********************************************************************************
Работа с системными таблицами
********************************************************************************

SysParamString(name string) string
==============================
Функция возвращает значение указанного системного параметра.

* *name* - имя параметра;

.. code:: js

    url = SysParamString(`blockchain_url`)

SysParamInt(name string) int
==============================
Функция возвращает значение указанного системного параметра в виде числа.

* *name* - имя параметра;

.. code:: js

    maxcol = SysParam(`max_columns`)

SysCost(name string) int
==============================
Функция возвращает стоимость указанной встроенной транзакции.

* *name* - имя параметра;

.. code:: js

    cost = SysCost(`dlt_transfer`)


UpdateSysParam(name, value, conditions string)
==============================
Функция обновляет значение и условие системного параметра. Если значение или условие менять не нужно, то следует в соответствующем параметре указать пустую строку.

* *name* - имя параметра;
* *value* - новое значение параметра;
* *conditions* - новое условие изменения параметра;

.. code:: js

    UpdateSysParam(`fuel_rate`, `400000000000`, ``)

********************************************************************************
Работа с PostgreSQL
********************************************************************************

Функции не дают возможности напрямую отправлять запросы с select, update и т.д., но они позволяют использовать возможности и функции PostgrеSQL при получении значений и описания условий where в выборках. Это относится в том числе и к функциям по работе с датами и временем. Например, необходимо сравнить колонку *date_column* и текущее время. Если *date_column* имеет тип timestamp, то выражение будет следующим *date_column > now()*, а если *date_column* хранит время в Unix формате в виде числа, то тогда выражение будет *to_timestamp(date_column) > now()*. 

.. code:: js

    to_timestamp(date_column) > now()
    date_initial < now() - 30 * interval '1 day'

Рассмотрим ситуацию, когда у нас есть значение в формате Unix и необходимо записать его в поле имеющее тип *timestamp*. В этом случае, при перечислении полей, перед именем данной колонки необходимо указать **timestamp**. 

.. code:: js

   DBInsert(Table("mytable"), "name,timestamp mytime", "John Dow", 146724678424 )

Если же вы имеете строковое значение времени и вам нужно записать его в поле с типом *timestamp*. В этом случае,  **timestamp** необходимо указать перед самим значением. 

.. code:: js

   DBInsert(Table("mytable"), "name,mytime", "John Dow", "timestamp 2017-05-20 00:00:00" )
   var date string
   date = "2017-05-20 00:00:00"
   DBInsert(Table("mytable"), "name,mytime", "John Dow", "timestamp " + date )
   DBInsert(Table("mytable"), "name,mytime", "John Dow", "timestamp " + $txtime )

