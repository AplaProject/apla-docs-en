################################################################################
Language of contracts
################################################################################
.. contents::
  :local:
  :depth: 2
  
Contract is the basic structure for the Apla algorithms implementation. The complete code snippets ensuring the acceptance of input data from the user or any other contract, the analysis of their correctness and the performance of the necessary transactions are drawn up as contracts. The language of contracts is a scripting language with fast compilation to byte code. It supports variables with the main types of values, contains functions, a standard set of operators and structures, and a bug management database.

Contracts are compiled into a byte code and are available for all users. Upon the contract display, the isolated stack with еру input data and a set of variables is created, which the virtual machine processes when executing a byte code. Thus, multiple processes which will not influence each other can be performed on the same byte code, which will not influence each other.

********************************************************************************
Value types and variables
********************************************************************************

The language variables are declared with the indication of the value type. The automatic type conversion is applied in obvious cases. The following types of values are used:

*	**bool** - boolean which takes true or false value;
*	**bytes** - byte sequence;
*	**int** - 64-bit integer;
*	**address** - 64-bit unsigned integer;
*	**array** - an array of values with arbitrary types;
*	**map** - an associative array of values with arbitrary types with string keys;
*	**money** - an integer of the big integer type; the values are stored in the database without the decimal points, which are inserted upon display in the interface in accordance with the currency settings; 
float - 64-bit floating-point number;
*	**string** - a line; it is specified in double or reverse quotes - “This is a line” or This is a line.

All identifiers - names of variables, functions, contracts, etc. are a case sensitive ones (MyFunc and myFunc - these are different names).

The variables are declared using the **var** keyword, followed by the variable name or names and their type. The variables are defined and operate inside the curly brackets. When describing the variables, the default value is automatically assigned to them: for the *bool* type it is *false*, for all numeric types - zero values, and for strings - empty string. Examples of variable declarations:

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

********************************************************************************
Arrays
********************************************************************************

The language supports two types of arrays:

* **array** - a simple array with a numeric index, starting with 0;
* **map** - an associative array with string keys.

The components are assigned and received by specifying the index in square brackets.

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

********************************************************************************
"If" and "while" constructions
********************************************************************************

The contracts description language contains a conditional structure **if** and the **while** loop structure, which are used within functions, and contracts. These structures can be inserted into each other. 

The keyword must be followed by the conditional expression. If the conditional expression returns a number, then it is considered to be *false* at the 0 value. For example, *val == 0* is equivalent to *!val*, and *val != 0* is the same as simply *val*. The **if** structure may have an **else** block which is executed if the conditional if statement is false. Comparison operations can be used in conditional expressions: *<, >, >=, <=, ==, !=, as well as || (OR) and && (AND)*.

.. code:: js

    if val > 10 || id != $citizen {
      ...
    } else {
      ...
    }

The **while** structure is intended for the loops implementation. The while **block** is executed as long as its condition is true. The **break** statement is used to stop the loop inside the block. The **continue** statement is used first to implement the loop block.

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

In addition to the conditional expressions, the language supports the standard arithmetic operations: +,-,*,/
If you specify a variable of the **string** or **bytes** type as a condition, the condition will be true in cases where the length of the string (bytes) is greater than zero. The condition will be false for an empty string.

********************************************************************************
Functions
********************************************************************************

The function is defined by using the **func** keyword, followed by the function name, in parentheses, separated by commas, the transmitted parameters indicating type, and the returned value type after the closing parenthesis. The function body is enclosed in curly brackets. If the function has no parameters, the parentheses may be omitted. The return keyword is used to **return** a value from a function.

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

You can pass any number of parameters to a function. To do this, use **...** as the last parameter type. In this case, the last parameter will be of *array* type and will contain all variables specified in the call, starting from this parameter. You can pass variables of any types, so please make sure you avoid conflicts caused by possible type mismatches.

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

Let's consider a case, where a function has many parameters, but only a few of them are needed when calling this function. In this case, optional parameters can be specified as **func myfunc(name string).Param1(param string).Param2(param2 int) {...}**. You can specify any of the additional parameters in any order **myfunc("name").Param2(100)**. In the function body you can access these variables as usual. If the extended parameter is not specified with a call, default values are applied, for example, a string variable would be an empty string, and a number would be zero. You can also specify several extended parameters and use **...** - **func DBFind(table string).Where(request string, params ...)** and call **DBFind("mytable").Where("id > ? and type = ?", myid, 2)**

.. code:: js
 
   func DBFind(table string).Columns(columns string).Where(format string, tail ...)
             .Limit(limit int).Offset(offset int) string  {
       ...
    }
     
    func names() string {
       ...
       return DBFind("table").Columns("name").Where("id=?", 100).Limit(1)
    }


********************************************************************************
Contracts
********************************************************************************

Contract is the basic language structure, which assists the implementation of a single action initiated by the user or other contract in the interface. The entire application source code is made as a system of contracts, interacting through the database or by displaying each other in the contract body.

The contract is defined by the contract keyword, followed by the name of the contract. The contract body is enclosed in curly brackets. The contract consists of three sections: 

1.	**data** is used to describe the input data (variable names and types);
2.	**conditions** implements the input data validation;
3.	**action contains** a description of the contract action.

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


Data description in the data section
==============================

The contract input data, as well as the parameters of the form for the reception of the data are described in the **data** section. 
The data are listed line by line: first, the variable name is specified (only variables, but not arrays are transferred), then the type and the parameters for building of the interface form are indicated optionally through a gap in double quotation marks:


*	*hidden* - hidden element of the form;
*	*optional* - form element without obligatory filling in;
*	*date* - field of the date and time selection;
*	*polymap* - map with the selection of coordinates and areas;
*	*map* - map with the ability to mark the place;
*	*image* - images upload;
*	*text* - entry of the text of HTML-code in the textarea field;
*crypt:Field* - create and encrypt a private key for the destination specified in the *Field* field. If only * crypt * is specified, then the private key will be created for the user who signs the contract.
*	*address* - field for input of the wallet address;
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
  
The variables in the contract
==============================
The pre-defined variables that contain data about the transaction from which the contract was displayed are also available in the contract.

*	*$time* - int. transaction time
*	*$state* - int. ecosystem identifier
*	*$block* - block number in which the int. transaction is sealed 
*	*$citizen* - address of the citizen who signed the int transaction.
*	*$wallet* - address of the wallet signatory to the transaction if the contract is outside the ecosystem, where the state == 0.
*	*$wallet_block* - address of the node that has formed the block that includes the transaction.
*	*$block_time* - time of formation of the block containing the transaction with the current int. contract.

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
  
Conditions section
==============================
Validation of the data obtained is performed in the section. The following commands are used to warn of the presence of errors: **error, warning, info**. In fact, all they generate an error that stops the contract operation, but display different messages in the interface: critical error, warning, and informative error. For instance, 

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


Enclosed contracts
==============================
Another contract may be displayed in the **conditions** and **action** sections of the contract. To do this, the name of such contract must be specified, and the necessary parameters shall be described in parentheses: the names of transmitted data (from the **data** section of the displayed contract) are indicated in quotation marks, separated by commas, followed by a comma-separated list of variables that contain the values transmitted. For instance,

.. code:: js
MoneyTransfer("SenderAccountId,RecipientAccountId,Amount",sender_id,recipient_id,$Price)

An enclosed contract may return the value obtained in it through the global variables declared therein (with the **$** sign in front). 
Display of the enclosed contract is also possible through the **CallContract()** function for which the contract name is transferred through a string variable. 

Contracts with signature
==============================
Since the language of contracts writing allows performing enclosed contracts, it is possible to fulfill such an enclosed contract without the knowledge of the user who has run the external contract that may lead to the user's signature of transactions unauthorized by it, let's say the transfer of money from its account.

Let's suppose there is a MoneyTransfer Contract *MoneyTransfer*:

.. code:: js

    contract MoneyTransfer {
        data {
          Recipient int
          Amount    money
        }
        ...
    }

If in a contract launched by the user the string MoneyTransfer("Recipient,Amount", 12345, 100) is inscribed, 100 coins will be transferred to the wallet 12345. In such a case the user who signs an external contract will remain not in the know of the transaction. This situation may be excluded if the MoneyTransfer contract requires the additional user's signature upon its calling in of contracts. To do this:

1. Adding a field with the name **Signature** with the *optional* and *hidden* parameters in the *data* section of the *MoneyTransfer* contract, which allow not to require the additional signature in the direct calling of the contract, since there will be the signature in the **Signature** field so far.

.. code:: js

    contract MoneyTransfer {
        data {
          Recipient int
          Amount    money
          Signature string "optional hidden"
        }
        ...
    }


2. Adding in the *Signatures* table (on the page **Signatures** of Apla client) the entry containing:

•	*MoneyTransfer* contract name,
•	field names whose values will be displayed to the user, and their text description,
•	text to be displayed upon confirmation.
  
In the current example it will be enough specifying two fields **Receipient** and **Amount**:

* **Title**: Are you agree to send money this recipient?
* **Parameter**: Receipient Text: Wallet ID
* **Parameter**: Amount Text: Amount (qEGS)

Now, if inserting the *MoneyTransfer(“Recipient, Amount”, 12345, 100)* contract calling in, the system error *“Signature is not defined”* will be displayed. If the contract is called in as follow: *MoneyTransfer(“Recipient, Amount, Signature”, 12345, 100, ”xxx...xxxxx”)*, the system error will occur upon signature verification. Upon the contract calling in, the following information is verified: ""time of the initial transaction, user ID, the value of the fields specified in the signatures table"", and it is impossible to forge the signature.

In order for the user to see the money transfer confirmation upon the *MoneyTransfer* contract calling in, it is necessary to add a field with an arbitrary name and the type *string*, and with the optional parameter *signature:contractname*. Upon calling in of the enclosed *MoneyTransfer* contract, you just need to forward this parameter. It should also be borne in mind that the parameters for the secured contract calling in must also be described in the *data* section of the external contract (they may be hidden, but they will still be displayed upon confirmation). For instance,

.. code:: js

    contract MyTest {
      data {
          Recipient int "hidden"
          Amount  money
          Signature string "signature:send_money"
      }
      func action {
          MoneyTransfer("Recipient,Amount,Signature",$Recipient,$Amount,$Signature)
      }
    }

When sending a *MyTest* contract, the additional confirmation of the money transfer to the indicated wallet will be requested from user. If other values, such as *MoneyTransfer(“Recipient,Amount,Signature”,$Recipient, $Amount+10, $Signature)*, are listed in the enclosed contract, the invalid signature error will occur.



********************************************************************************
Rights of access to system components
********************************************************************************
Apla has a multi-level rights management system for the creation and editing of database tables, contracts, pages and interface menu, and parameters of the state setting pattern. Rights are specified when creating and editing the above elements in the "Permissions" fields in the relevant sections of the state setting (smart contracts, tables, interface). Rights are recorded as logical expressions, and are provided if the expression is *true* at the time of access. If the "Permissions" field remains empty, then it automatically becomes *false*, and the access to perform the relevant actions is completely closed.

The rights to the following actions are fixed:

1.	*Table column permission* - the right to change the values in the table column;
2.	*Table Insert permission* - the right to write a new line in the table;
3.	*Table New Column permission* - the right to add a new column;
4.	*Conditions for changing of Table permissions* - the right to change the rights listed in paragraphs 1-3;
5.	*Conditions for change cmart contract* - the right to change the contract;
6.	*Conditions for changepage* - the right to change the interface page;
7.	*Conditions for change menu* - the right to change the menu;
8.	*Conditions for change of State parameters* - the right to change a certain parameter of the ecosystem setting pattern.

The simplest way to provide the rights is the prescription of the logical expression *$citizen == 2263109859890200332* in the “Conditions” field, with the indication of the identification number of a particular user. The use of the ContractAccess(“NameContract”) function, to which the list of contracts entitled to implement appropriate action is transferred, is the multi-purpose and recommended method to define the rights. For example, in the table of accounts after the prescription of the ContractAccess("MoneyTransfer") in the "Conditions" field of the amount column, the change of the amount value will be available only to the MoneyTransfer smart contract (all contracts involving the transfer of money from one account to another should only do so by displaying the MoneyTransfer contract). Conditions for obtaining access to the contracts is controlled in the conditions section and can be quite complex, involving many other contracts and smart-laws.

In order to solve conflicts or dangerous situations for the system operation, the State parameters table includes the special parameters (*state_changing_smart_contracts, state_changing_tables, state_changing_pages*), in which the conditions for obtaining access rights to any smart contracts, tables or pages are specified. These rights are set by special smart-laws, for example, providing for a judicial decision or a few signatures of responsible persons.

Through the use of contracts to secure the rights we obtain the flexibly tunable resources access control system, allowing, among other things, to automatically track the transfer of authorities from a person to another person, for example, upon change of the positions occupied.
