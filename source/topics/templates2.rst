User Interfaces
###############

.. contents::
  :local:
  :depth: 2


Interfaces building
===================

Integrated Development Environment of the Molis software client, which is created using the *JavaScript React library*, includes an interface editor and a virtual interface designer. Interface pages are the essential part of applications that provides for retrieval and display of data from database tables, creation of forms for receipt of user input data, passing data to contracts, and navigation between application pages. Interface pages, just as contracts, are stored in the blockchain, which ensures their protection from falsification when loading them in the software client.  


Interface template engine
-------------------------

Interface elements (pages and menus) are formed on Validating Nodes in a so-called *template engine* from templates created by programmers in the interface editor of the Molis software client. All interface pages are built using the Protypo functional language developed by platform developers. Interfaces are requested from nodes on the network using the *content* API command. What the template engine sends as a reply to such request is not an HTML page, but a JSON code comprised of HTML tags that form a tree in accordance with the template structure. For testing purposes, a POST request can be sent to ``api/v2/content`` with the *template* parameter containing the name of a template to process.


Creating interface templates
----------------------------

Interfaces can be created and edited using a specialized editor, available in the **Interface** section of administrative tools in Molis. The editor provides for:

- Writing codes of interface pages with highlighting of keywords of the Protypo template language,
- Selecting a menu, which will be displayed on the page,
- Editing the page menu,
- Configuring permission to edit the page (typically, by way of specifying the name of the contract with permissions in the *ContractConditions* function, or by direct indication of access rights in the *Change conditions* field),
- Launching a visual interface designer,
- Page preview.


Visual interface designer
"""""""""""""""""""""""""

Visual Interface Designer allows for creating page designs without resorting to the interface source code in Protypo language. The Designer allows for setting the positions of form elements and text on the page using drag-and-drop, as well as configuring sizes and design of page blocks. The Designer provides a set of ready-to-use blocks for displaying typical data models: panels with headers, forms, and information panels. The program logics (receipt of data and conditional constructs) can be added in the page editor after the page design is created. (In the future, we plan to create a full-scale visual interface editor.)


Use of styles
"""""""""""""

By default, interface pages are displayed using Angular Bootstrap Angle classes. If needed, users can create their own styles. Storage of styles is implemented using a special stylesheet parameter of the ecosystem configuration table. 


Page blocks
"""""""""""

To use typical code fragments on multiple interface pages there is an option to create page blocks and embed them in the interface code using the Insert command. Such blocks can be created and edited on the Interface page of the administrative section in Molis. For blocks, just as for pages, permissions for editing can be defined.


Language resources editor
"""""""""""""""""""""""""

The Molis software client includes a mechanism for interface localization using a special function of the Protypo template language – LangRes, which substitutes the language resource labels on the page with corresponding text lines in the language selected by the user in the software client (or browser for the web-version of the client). A shorter syntax $lable$ can be used instead of the LangRes function. Translation of messages in pop-up windows, initiated by contracts, is carried out by the LangRes function of the Simvolio language.

Language resources can be created and edited in the Language resources section of the administrative tools of the Molis software client. A language resource consists of a label (name) and the translations of this name into different languages with the indication of corresponding two-character language identifiers (EN, FR, JP, etc.).

Rights to add and change language resources can be configured using the same way as for any other table in the languages table (Tables section of the Molis administrative tools). 


Protypo template language
=========================

Protypo functions provide for implementation of the following operations:

- retrieving values from the database: DBFind,
- representation of data retrieved from the database as tables and diagrams,
- assignment and display of values of variables, operations with data: SetVar, GetVar, Data,
- display and comparison of date/time values: DateTime, Now, CmpTime,
- building forms with various sets of user data input fields: Form, ImageInput, Input, RadioGroup, Select,
- validation of data in the form fields by displaying error messages: Validate, InputErr,
- display of navigation elements: AddToolButton, LinkPage, Button,
- calling contracts: Button,
- creation of HTML page layout elements – various containers with an option to specify css classes: Div, P, Span, etc.,
- embedding images onto a page and uploading of images: Image and ImageInput,
- conditional display of page layout fragments: ``If, ElseIf, Else``,
- creation of multi-level menus,
- interface localization.


Overview of Protypo
-------------------

Page template language is a functional language that allows for calling functions using ``FuncName(parameters)``, and for nesting functions into each other. Parameters can be specified without quote marks. Unnecessary parameters can be dropped.

.. code:: js

      Text FuncName(parameter number 1, parameter number 2) another text.
      FuncName(parameter 1,,,parameter 4)
      
If a parameter contains a comma, it should be enclosed in quotes marks (back quotes or double quotes). If a function can have only one parameter, commas can be used in it without quotes.  Also, quotes should be used in case a parameter has an unpaired closing parenthesis.

.. code:: js

      FuncName("parameter number 1, the second part of first paremeter")
      FuncName(`parameter number 1, the second part of first paremeter`)
      
If you put a parameter in quotes, but a parameter itself includes quotes, then you can use different type of quotes or double them in the text.
      
.. code:: js

      FuncName("parameter number 1, ""the second part of first"" paremeter")
      FuncName(`parameter number 1, "the second part of first" paremeter`)
      
In description of functions, every parameter has a specific name. You can call functions and specify parameters in the order they were declared, or specify any set of parameters in any order by their names: ''Parameter_name: Parameter_value''. This approach allows to safely add new function parameters without breaking the compatibility with current templates. For example, all of these calls are correct in terms of language use for a function described as ''FuncName(Class,Value,Body)'':

.. code:: js

      FuncName(myclass, This is value, Div(divclass, This is paragraph.))
      FuncName(Body: Div(divclass, This is paragraph.))
      FuncName(myclass, Body: Div(divclass, This is paragraph.))
      FuncName(Value: This is value, Body: 
           Div(divclass, This is paragraph.)
      )
      FuncName(myclass, Value without Body)
      
Functions can return text, generate HTML elements (for instance, ''Input''), or create HTML elements with nested HTML elements (''Div, P, Span''). In the latter case a parameter with a pre-defined name **Body** should be used to define nested elements. For example, two *div*, nested in another *div*, can look like this:

.. code:: js

      Div(Body:
         Div(class1, This is the first div.)
         Div(class2, This is the second div.)
      )
      
To define nested elements, which are described in the *Body* parameter, the following representation can be used: ``FuncName(...){...}``. Nested elements should be specified in curly braces. 

.. code:: js

      Div(){
         Div(class1){
            P(This is the first div.)
            Div(class2){
                Span(This is the second div.)
            }
         }
      }
      
If you need to specify the same function a number of times in a row, you can use points instead of writing the function name every time. For example, the following lines are equal:
     
.. code:: js

     Span(Item 1)Span(Item 2)Span(Item 3)
     Span(Item 1).(Item 2).(Item 3)
     
The language allows for assigning variables using the **SetVar** function. To substitute values of variables use ``#varname#``.

.. code:: js

     SetVar(name, My Name)
     Span(Your name: #name#)
     
To substitute the language resources of the ecosystem, you can use the ``$langres$``, where *langres* is the name of the language source.

.. code:: js

     Span($yourname$: #name#)
     
The following variables are predefined: 

* ``#key_id#`` - current user account identifier,
* ``#ecosystem_id#`` - current ecosystem identifier.
* ``#guest_key#`` - guest wallet identifier.
* ``#isMobile#`` - is 1 if the client is running on a mobile device.


Passing parameters to a page using PageParams
"""""""""""""""""""""""""""""""""""""""""""""

There is a number of functions that support the **PageParams** parameter, which serves for passing parameters when redirecting to a new page. For example, ``PageParams: "param1=value1,param2=value2"``. Parameter values can be both simple strings or rows with value substitution. When parameters are passed to a page, variables with parameter names are created; for example, ``#param1#`` and ``#param2#``.  

* ``PageParams: "hello=world"`` - the page will receive the hello parameter with world as value,
* ``PageParams: "hello=#world#"`` - the page will receive the hello parameter with the value of the world variable.

Additionally, the **Val** function allows for obtaining data from forms, which were specified in redirect. In this case,

* ``PageParams: "hello=Val(world)"`` - the page will receive the hello parameter with the value of the world form element.


Calling contracts
"""""""""""""""""

Protypo implements contract calling by clicking on a button in a form (*Button* function). Once  this event is initiated, the data entered by the user in the fields of the interface forms is passed to the contract (if the names of form fields correspond to the names of variables in the data section of the called contract, data is transferred automatically). The Button function allows for opening a modal window for user verification of the contract execution (Alert), and initiation of redirect to a specified page after the successful execution of the contract, and passing certain parameters to this page.    


Protypo functions by purpose
============================


Operations with variables
-------------------------

.. hlist::
    :columns: 3

    - :ref:`protypo-GetVar`
    - :ref:`protypo-SetVar`


Navigation
----------

.. hlist::
    :columns: 3

    - :ref:`protypo-AddToolButton`
    - :ref:`protypo-Button`
    - :ref:`protypo-LinkPage`


Data operations
---------------

.. hlist::
    :columns: 3

    - :ref:`protypo-Calculate`
    - :ref:`protypo-CmpTime`
    - :ref:`protypo-DateTime`
    - :ref:`protypo-Now`
    - :ref:`protypo-Money`



Displaying data
---------------

.. hlist::
    :columns: 3

    - :ref:`protypo-Code`
    - :ref:`protypo-CodeAsIs`
    - :ref:`protypo-Chart`
    - :ref:`protypo-ForList`
    - :ref:`protypo-Hint`
    - :ref:`protypo-Image`
    - :ref:`protypo-MenuGroup`
    - :ref:`protypo-MenuItem`
    - :ref:`protypo-QRcode`
    - :ref:`protypo-Table`


Receiving data
--------------

.. hlist::
    :columns: 3

    - :ref:`protypo-Address`
    - :ref:`protypo-AddressToId`
    - :ref:`protypo-AppParam`
    - :ref:`protypo-Data`
    - :ref:`protypo-DBFind`
    - :ref:`protypo-EcosysParam`
    - :ref:`protypo-GetHistory`
    - :ref:`protypo-GetColumnType`
    - :ref:`protypo-JsonToSource`
    - :ref:`protypo-ArrayToSource`
    - :ref:`protypo-LangRes`
    - :ref:`protypo-Range`
    - :ref:`protypo-SysParam`
    - :ref:`protypo-Binary`
    - :ref:`protypo-TransactionInfo`


Elements of data formatting
---------------------------

.. hlist::
    :columns: 3

    - :ref:`protypo-Div`
    - :ref:`protypo-Em`
    - :ref:`protypo-P`
    - :ref:`protypo-SetTitle`
    - :ref:`protypo-Label`
    - :ref:`protypo-Span`
    - :ref:`protypo-Strong`


Elements of forms
-----------------

.. hlist::
    :columns: 3


    - :ref:`protypo-Form`
    - :ref:`protypo-ImageInput`
    - :ref:`protypo-Input`
    - :ref:`protypo-InputErr`
    - :ref:`protypo-RadioGroup`
    - :ref:`protypo-Select`
    - :ref:`protypo-InputMap`
    - :ref:`protypo-Map`


Operations with code
--------------------

.. hlist::
    :columns: 3

    - :ref:`protypo-If`
    - :ref:`protypo-And`
    - :ref:`protypo-Or`
    - :ref:`protypo-Include`


Protypo functions reference
===========================

.. _protypo-Address:

Address
-------

This function returns the account address in the ``1234-5678-...-7990`` format given the numerical value of the address; if the address is not specified, the address of the current user will be taken as the argument. 


Syntax
""""""

.. code-block:: text

    Address (account)

.. describe:: Address

    .. describe:: account

        Account identifier.


Example
"""""""

.. code:: js

    Span(Your wallet: Address(#account#))


.. _protypo-AddressToId:

AddressToId
-----------

Returns the account identifier for the specified account address in the ``1234-5678-...-7990`` format.

Syntax
""""""

.. code-block:: text

    AddressToId(Wallet)


.. describe:: AddressToId

    .. describe:: Wallet

        Account address in the ``XXXX-...-XXXX`` format or as a number.


Example
"""""""

.. code:: js

  AddressToId(#wallet#)



.. _protypo-AddToolButton:

AddToolButton
-------------

Adds a button to the buttons panel. Creates **addtoolbutton** element. 


Syntax
""""""

.. code-block:: text

    AddToolButton(Title, Icon, Page, PageParams) 
        [.Popup(Width, Header)]


.. describe:: AddToolButton

    .. describe:: Title

        Button title.

    .. describe:: Icon

        Icon for the button.

    .. describe:: Page

        Page name for the jump.

    .. describe:: PageParams

        Parmeters that are passed to the page.

.. describe:: Popup

    Outputs a modal window.

    .. describe:: Header

        Window header.
    
    .. describe:: Width

        Window width in percent.

        Range of values for this parameter is from 1 to 100.


Example
"""""""

.. code:: js

      AddToolButton(Help, help, help_page) 


.. _protypo-And:

And
---

This function returns the result of execution of the **and** logical operation with all parameters listed in parentheses and separated by commas. The parameter value will be ``false`` if it equals an empty string (``""``), zero or *false*. In all other cases the parameter value is ``true``. The function returns 1 if true or 0 in all other cases. The element named ``and`` is created only when a tree for editing is requested. 

Syntax
""""""

.. code-block:: text

    And(parameters)


Example
"""""""

.. code:: js

      If(And(#myval1#,#myval2#), Span(OK))


.. _protypo-AppParam:

AppParam
--------

Outputs the value of an app parameter. The value is taken from the app_param table of the current ecosystem. If there is a language resource with the given name, then its value will be substituted automatically.

.. todo::

    Resulting or given name?

Syntax
""""""

.. code-block:: text

    AppParam(App, Name, Index, Source) 

.. describe:: AppParam
 
    .. describe:: App

        Application identifier.

    .. describe:: Name

        Parameter name.

    .. describe:: Index

        This parameter can be used when the parameter value is a list of items separated by commas.

        Index of a parameter element, starting from 1.  For example if ``type = full,light`` then ``AppParam(1, type, 2)`` returns ``light``.

        This parameter cannot be used with *Source* parameter.

    .. describe:: Source

        This parameter can be used when the parameter value is a list of items separated by commas.

        Creates a *data* object. Elements of this object are values of the specified parameter. The object can be used as a data source in :ref:`protypo-Table` and :ref:`protypo-Select` functions.

        This parameter cannot be used with *Index* parameter.

Example
"""""""

.. code:: js

     AppParam(1, type, Source: mytype)


.. _protypo-ArrayToSource:

ArrayToSource
-------------

Creates an **arraytosource** element and populates it with *key* - *value* pairs that were passed in a JSON array. The resulting data is put into the *Source* element, which can later be used in functions that use source inputs (such as :ref:`protypo-Table`).


Syntax
""""""

.. code-block:: text

    ArrayToSource(Source, Data)

.. describe:: ArrayToSource
    
    .. describe:: Source

        Data source name.

    .. describe:: Data

        A JSON array or a name of a variable (``#name#``) that holds a JSON array.


Example
"""""""

.. code:: js

   ArrayToSource(src, #myjsonarr#)
   ArrayToSource(dat, [1, 2, 3])

.. _protypo-Binary:

Binary
------

Returns a link to a static file that is stored in the *binaries* table.


Syntax
""""""

.. code-block:: text

    Binary(Name, AppID, MemberID)[.ById(ID)][.Ecosystem(ecosystem)]
 
.. describe:: Binary

    .. describe:: Name

        File name.

    .. describe:: AppID

        Application identifier.

    .. describe:: MemberID

        Account identifier. The default value is 0.

    .. describe:: ID

        Static file identifier.

    .. describe:: ecosystem

        Ecosystem identifier. If this parameter is not specified, binary file is requested from the current ecosystem.

Example
"""""""

.. code:: js

     Image(Src: Binary("my_image", 1))
     Image(Src: Binary().ById(2))
     Image(Src: Binary().ById(#id#).Ecosystem(#eco#))


.. _protypo-Button:

Button
------

Creates a **button** HTML element. This element creates a button, which executes a contract or opens a page.

Syntax
""""""

.. code-block:: text

    Button(Body, Page, Class, Contract, Params, PageParams)
        [.CompositeContract(Contract, Data)]
        [.Alert(Text, ConfirmButton, CancelButton, Icon)]
        [.Popup(Width, Header)]
        [.Style(Style)]
        [.ErrorRedirect((ErrorID,PageName,PageParams)]

.. describe:: Button

    .. describe:: Body

        Child text or elements.

    .. describe:: Page

        Name of the page to redirect to.

    .. describe:: Class

        Classes for the button.

    .. describe:: Contract

        Name of the contract to execute.

    .. describe:: Params

        List of values to pass to the contract. By default, values of contract parameters (data ``section``) are obtained from HTML elements (for example, input fields) with similarly-named identifiers (``id``). If the element identifiers differ from the names of contract parameters, then the assignment in the ``contractField1=idname1, contractField2=idname2`` format should be used. This parameter is returned to *attr* as an object ``{field1: idname1, field2: idname2}``.

    .. describe:: PageParams

        Parameters for redirection to a page in the following format: ``contractField1=idname1, contractField2=idname2``. In this case, variables with parameter names ``#contractField1#`` and ``#contractField2`` are created on the target page, and are assigned the specified values (see the parameter passing specifications in the "*Passing Parameters to a Page Using PageParams*" section above).

.. describe:: CompositeContract

        Used for adding extra contracts for a button. CompositeContract can be used several times.

        .. describe:: Name

            Contract name.

        .. describe:: Data

            Contract parameters as a JSON array.

.. describe:: Alert

    Displays a message.

    .. describe:: Text

        Message text.

    .. describe:: ConfirmButton

        Confirm button caption.

    .. describe:: CancelButton

        Cancel button caption.

    .. describe:: Icon

        Icon.

.. describe:: Popup

    Outputs a modal window.

    .. describe:: Header

        Window header.
    
    .. describe:: Width

        Window width in percent.

        Range of values for this parameter is from 1 to 100.

.. describe:: Style

    Specifies CSS styles.

    .. describe:: Style

        CSS styles.

.. describe:: ErrorRedirect

    Specifies a redirect page. This redirect page is used when the *Throw* function generates an error during the contract execution. There may be several *ErrorRedirect* calls. As a result, an *errredir* attribute is returnes with *ErrorID* list of keys and parameters as values.

    .. describe:: ErrorID

        Error identifier.

    .. describe:: PageName

        Name of the redirect page.

    .. describe:: PageParams

        Parameters passed to this page.


Example
"""""""

.. code:: js

      Button(Submit, default_page, mybtn_class).Alert(Alert message)
      Button(Contract: MyContract, Body:My Contract, Class: myclass, Params:"Name=myid,Id=i10,Value")


.. _protypo-Calculate:

Calculate
---------

This function returns the result of an arithmetic expression passed in the **Exp** parameter. The following operations can be used: +, -, \*, /, and parenthesis (). 

Syntax
""""""

.. code-block:: text

    Calculate(Exp, Type, Prec)

.. describe:: Calculate

    .. describe:: Exp

        Arithmetic expression. Can contain numbers and *#name#* variables.

    .. describe:: Type

        Result data type: **int, float, money**. If not specified, then the result type will be *float* in case there are numbers with a decimal point, or *int* in all other cases.

    .. describe:: Prec

        The number of significant digits after the point can be specified for *float* and *money* types.

Example
"""""""

.. code:: js

    Calculate( Exp: (342278783438+5000)\*(#val#-932780000), Type: money, Prec:18 )
    Calculate(10000-(34+5)\*#val#)
    Calculate("((10+#val#-45)\*3.0-10)/4.5 + #val#", Prec: 4)      


.. _protypo-Chart:

Chart
-----

Creates an HTML diagram.

Syntax
""""""

.. code-block:: text

    Chart(Type, Source, FieldLabel, FieldValue, Colors)

.. describe:: Chart

    .. describe:: Type

        Diagram type.

    .. describe:: Source

        Name of the data source, for example, a source taken from the *DBFind* command.

    .. describe:: FieldLabel

        Name of a field that will be used for headers.

    .. describe:: FieldValue

        Name of a field that will be used for values.

    .. describe:: Colors

        List of used colors.


Example
"""""""

.. code:: js

      Data(mysrc,"name,count"){
          John Silver,10
          "Mark, Smith",20
          "Unknown ""Person""",30
      }
      Chart(Type: "bar", Source: mysrc, FieldLabel: "name", FieldValue: "count", Colors: "red, green")


.. _protypo-CmpTime:

CmpTime
-------

This function compares two time values in the same format.

Supports unixtime, ``YYYY-MM-DD HH:MM:SS``, and any arbitrary format, if the sequence is followed from years to seconds, for example ``YYYYMMDD``). 

Syntax
""""""

.. code-block:: text

    CmpTime(Time1, Time2)


Return values
"""""""""""""

* ``-1`` - Time1 < Time2, 
* ``0`` - Time1 = Time2, 
* ``1`` - Time1 > Time2.


Example
"""""""

.. code:: js

     If(CmpTime(#time1#, #time2#)<0){...}


.. _protypo-Code:

Code
----

Creates a **code** element for displaying the specified code.

This function replaces variables (e.g. ``#name#``) with their values. 

Syntax
""""""

.. code-block:: text

    Code(Text)

.. describe:: Code

    .. describe:: Text  

        Source code.

Example
"""""""

.. code:: js

      Code( P(This is the first line.
          Span(This is the second line.))
      )  


.. _protypo-CodeAsIs:

CodeAsIs
--------

Creates a **code** element for displaying the specified code.

This function does not replace variables with their values. For example, ``#name#`` will be displayed as is. 

Syntax
""""""

.. code-block:: text

    CodeAsIs(Text)

.. describe:: CodeAsIs

    .. describe:: Text  

        Source code.

Example
"""""""

.. code:: js

      CodeAsIs( P(This is the #test1#.
          Span(This is the #test2#.))
      )

.. _protypo-Data:

Data
----

Creates a **data** element and fills it with specified data and put into the *Source*, that then should be specified in *Table* and other commands resivieng *Source* as the input data. The sequence of column names corresponds to that of *data* entry values.

Syntax
""""""

.. code-block:: text

    Data(Source,Columns,Data) 
        [.Custom(Column){Body}]

.. describe:: Data

    .. describe:: Source

        Data source name. You can specify any name, which can be included in other commands later as a data source (e.g. :ref:`protypo-Table`).

    .. describe:: Columns

        List of columns, separated by commas.

    .. describe:: Data

        Data.

        One record per line. Column values must be separated by commas. Data should be in the same order as set in *Columns*.

        For values with commas, put the value in double quotes (``"example1, example2", 1, 2``).
        For values with quotes, put the value in double double quotes (``"""example", "example2""", 1, 2``).

.. describe:: Custom

    Allows for assigning calculated columns for data. For example, you can specify a template for buttons and additional page layout elements. These fields are usually assigned for output to *Table* and other commands that use received data.

    If you want to assign several calculated columns, use multiple *Custom* tail functions.

    .. describe:: Column

        Column name. A unique name must be assigned.
  
    .. describe:: Body

        A code fragment. You can obtain values from other columns in this entry using ``#columnname#``, and then use these values in the code fragment.


Example
"""""""

.. code:: js

    Data(mysrc,"id,name"){
    "1",John Silver
    2,"Mark, Smith"
    3,"Unknown ""Person"""
     }.Custom(link){Button(Body: View, Class: btn btn-link, Page: user, PageParams: "id=#id#"}    


.. _protypo-DateTime:

DateTime
--------

Displays time and date in the specified format. 


Syntax
""""""

.. code-block:: text

    DateTime(DateTime, Format)

.. describe:: DateTime

    .. describe:: DateTime

        Time and date in unix time or in a standard format ``2006-01-02T15:04:05``.
 
    .. describe:: Format

        Format template: ``YY`` 2-digit year format, ``YYYY`` 4-digit year format, ``MM`` - month, ``DD`` - day, ``HH`` - hours, ``MM`` - minutes, ``SS`` – seconds. Example: ``YY/MM/DD HH:MM``. 

        If the format is not specified, the *timeformat* parameter value set in the *languages* table will be used. If this parameter is absent, the ``YYYY-MM-DD HH:MI:SS`` format will be used instead.


Example
"""""""

 .. code:: js

    DateTime(2017-11-07T17:51:08)
    DateTime(#mytime#,HH:MI DD.MM.YYYY)


.. _protypo-DBFind:

DBFind
------

Creates a **dbfind** element, fills it with data from the *table* table, and puts it to the *Source* structure. The *Source* structure can be then used in *Table* and other commands that receive *Source* as input data. The sequence of records in *data* must correspond to the sequence of column names.

Syntax
""""""

.. code-block:: text

    DBFind(table, Source)
        [.Columns(columns)]
        [.Where(conditions)]
        [.WhereId(id)]
        [.Order(name)]
        [.Limit(limit)]
        [.Offset(offset)]
        [.Count(countvar)]
        [.Ecosystem(id)]
        [.Cutoff(columns)]
        [.Custom(Column){Body}]
        [.Vars(Prefix)]

.. describe:: DBFind

    .. describe:: table

        Table name.

    .. describe:: Source

        Data source name.
 
.. describe:: Columns

    .. describe:: columns

        List of columns to be returned. If not specified, all columns will be returned. If there are columns of JSON type, you can address the record fields using the following syntax: **columnname->fieldname**. In this case, the resulting column name will be **columnname.fieldname**.


.. describe:: Where


    .. describe:: conditions

        Data search conditions. For example, ``.Where(name = '#myval#')``. 

        If there are columns of JSON type, you can address record fields using the following syntax: **columnname->fieldname**.


.. describe:: WhereId

    Search by ID. For example, ``.WhereId(1)``.

    .. describe:: id
        
        Record identifier.

.. describe:: Order

    Sorting by field.

    For more information about sorting syntax, see :ref:`simvolio-DBFind`.
    
    .. describe:: name

        Field name.

.. describe:: Limit

    .. describe:: limit

        Number of returned rows. Default value is 25, maximum value is 250.

.. describe:: Offset

    .. describe:: offset

        Offset for returned rows.

.. describe:: Count

        Total number of rows for the specified *Where* condition.

        In addition to being stored in a variable, the total count is also returned in the *count* parameter of the *dbfind* element.

        If *Where* and *WhereID* were not specified, then the total number of rows in a table will be returned. 

        .. describe:: countvar

            Name of a variable that will hold the row count.

.. describe:: Ecosystem

    .. describe:: id
        
        Ecosystem ID. By default, data is taken from the specified table in the current ecosystem.

.. describe:: Cutoff

    Is used for trimming and displaying a large volume of text data.

    .. describe:: columns

        List of columns separated by commas that must be processed by the *Cutoff* tail function.

        As a result, column value is replaced by a JSON obkect with two fields: *link* and *title*. If the value in a column is longer than 32 symbols, then a link to a full text and first 32 symbols are returned. If the value is 32 symbols and shorter, then the link is empty, and the title holds the full column value.

.. describe:: Custom

    Allows for assigning calculated columns for data. For example, you can specify a template for buttons and additional page layout elements. These fields are usually assigned for output to *Table* and other commands that use received data.

    If you want to assign several calculated columns, use multiple *Custom* tail functions.

    .. describe:: Column

        Column name. A unique name must be assigned.
  
    .. describe:: Body

        A code fragment. You can obtain values from other columns in this entry using ``#columnname#``, and then use these values in the code fragment.
  
.. describe:: Vars

    Generates a set of variables with values from the first row obtained by the query. When specifying this function, the *Limit* parameter automatically becomes equal to 1 and only one record is returned.

    .. describe:: Prefix

        Prefix that is added to variable names. The format is *#prefix_columnname#*, where the column name follows the underscore sign. If there are columns containing JSON fields, then the resulting variable will be in the following format *#prefix_columnname_field#*.

Example
-------

.. code:: js

    DBFind(parameters,myparam)
    DBFind(parameters,myparam).Columns(name,value).Where(name='money')
    DBFind(parameters,myparam).Custom(myid){Strong(#id#)}.Custom(myname){
       Strong(Em(#name#))Div(myclass, #company#)
    }


.. _protypo-Div:

Div
---

Creates a **div** HTML element.

Syntax
""""""

.. code-block:: text

    Div(Class, Body)
        [.Style(Style)]
        [.Show(Condition)]
        [.Hide(Condition)]

.. describe:: Div


    .. describe:: Class

        Classes for this *div*.

    .. describe:: Body

        Child elements.


.. describe:: Style

    Specifies CSS styles.

    .. describe:: Style

        CSS styles.


.. describe:: Show

    Defines conditions to show this block.

  .. describe:: Condition

    See *Hide* below.


.. describe:: Hide

    Defines conditions to hide this block.

    .. describe:: Condition

    Sequence of ``InputName=Value`` expressions. *Condition* is true when all expressions that it contains are true. An expression is true when ``InputName`` input has the ``Value`` text. If several *Show* or *Hide* calls are specified, then at least one of the *Condition* parameters must be true.


Example
"""""""

.. code:: js

    Div(class1 class2, This is a paragraph.).Show(inp1=test,inp2=none)


.. _protypo-EcosysParam:

EcosysParam
-----------

This function gets a parameter value from the parameters table of the current ecosystem. If there is a language resource for the resulting name, it will be translated accordingly.

Syntax
""""""

.. code-block:: text

    EcosysParam(Name, Index, Source)

.. describe:: EcosysParam

    .. describe:: Name
     
        Parameter name.

    .. describe:: Index

        In cases where the requested parameter is a list of elements separated by commas, you can specify an index starting from 1. For example, if ``gender = male,female``, then ``EcosysParam(gender, 2)`` will return ``female``.

        This parameter cannot be used with *Source* parameter.

    .. describe:: Source

        This parameter can be used when the parameter value is a list of items separated by commas.

        Creates a *data* object. Elements of this object are values of the specified parameter. The object can be used as a data source in :ref:`protypo-Table` and :ref:`protypo-Select` functions.

        This parameter cannot be used with *Index* parameter.

.. code:: js

     Address(EcosysParam(founder_account))
     EcosysParam(gender, Source: mygender)
 
     EcosysParam(Name: gender_list, Source: src_gender)
     Select(Name: gender, Source: src_gender, NameColumn: name, ValueColumn: id)


.. _protypo-Em:

Em
--

Creates an **em** HTML element.

.. todo::

    Style tail function?


Syntax
""""""

.. code-block:: text

    Em(Body, Class)

.. describe:: Em


    .. describe:: Body

        Сhild text or elements.

    .. describe:: Class

        Classes for this *em*.

Example
"""""""

.. code:: js

      This is an Em(important news).



.. _protypo-ForList:

ForList
-------

Displays a list of elements from the *Source* data source in the template format set out in *Body*, and creates the **forlist** element.

Syntax
""""""

.. code-block:: text

    ForList(Source, Index){Body}

.. describe:: ForList

    .. describe:: Source

        Data source from *DBFind* or *Data* functions.

    .. describe:: Index

        Variable for the iteration counter. Count starts from 1.

        This parameter is optional. If it is not specified, the iteration count value is written to the *[Source]_index* variable.

    .. describe:: Body

        A template to insert the elements in.

.. code:: js

      ForList(mysrc){Span(#mysrc_index#. #name#)}


.. _protypo-Form:

Form
----

Creates a **form** HTML element.


Syntax
""""""

.. code-block:: text

    Form(Class, Body) [.Style(Style)]


.. describe:: Form

    .. describe:: Body
        
        Child class or elements.
    
    .. describe:: Class
    
        Classes for this *form*.


.. describe:: Style

    Specifies CSS styles.

    .. describe:: Style

        CSS styles.


Example
"""""""

.. code:: js

      Form(class1 class2, Input(myid))


.. _protypo-GetColumnType:

GetColumnType
-------------

Returns the type of a column in a specified table.

Following column types can be returned: *text, varchar, number, money, double, bytes, json, datetime, double*.


Syntax
""""""

.. code-block:: text

    GetColumnType(Table, Column)


.. describe:: GetColumnType

    .. describe:: Table

        Table name.

    .. describe:: Column

        Column name.


Example
"""""""

.. code:: js

    SetVar(coltype,GetColumnType(members, member_name))Div(){#coltype#}


.. _protypo-GetHistory:

GetHistory
----------

Creates a **gethistory** element and popuates it with the history of changes of a record from the specified table. The resulting data is put into the *Source* element, which can later be used in functions that use source inputs (such as :ref:`protypo-Table`).

The resulting list is sorted in the order from recent changes to earlier ones.

The *id* field in the resulting table points to the id in the *rollback_tx* table. The *block_id* field contains the block number. The *block_time* field contains the block timestamp.


Syntax
""""""

.. code-block:: text

    GetHistory(Source, Name, Id, RollbackId)  

.. describe:: GetHistory

    .. describe:: Source

        Name for the data source.

    .. describe:: Name

        Table name.

    .. describe:: Id

        Identifier of a record.

    .. describe:: RollbackId

        Optional parameter. If specified, only one record with the specified identifier will be returned from the *rollback_tx* table.


Example
"""""""

.. code:: js

    GetHistory(blocks, BlockHistory, 1)


.. _protypo-GetVar:

GetVar
------

This function returns the value of the current variable if it exists, or returns an empty string if a variable with this name is not defined. An element with **getvar** name is created only when a tree for editing is requested. The difference between ``GetVar(varname)`` and ``#varname#`` is that in case *varname* does not exist, *GetVar* will return an empty string, whereas *#varname#* will be interpreted as a string value.


Syntax
""""""

.. code-block:: text

    GetVar(Name)

.. describe:: GetVar

    .. describe:: Name

        Variable name.

Example
"""""""

.. code:: js

     If(GetVar(name)){#name#}.Else{Name is unknown}


.. _protypo-Hint:

Hint
----

Creates a **hint** element to display hints.

Syntax
""""""

.. code-block:: text

    Hint(Icon,Title,Text)

.. describe:: Hint

    .. describe:: Icon

        Icon name.

    .. describe:: Title

        Hint title.

    .. describe:: Text

        Hint text.

Example
"""""""

.. code:: js

    Hint(myicon, My Header, This is a hint text)


.. _protypo-If:

If
--

Conditional statement. 

Child elements of the first *If* or *ElseIf* with fulfilled *Condition* are returned. Otherwise, child elements of *Else* are returned.

Syntax
""""""

.. code-block:: text

    If(Condition){ Body } 
        [.ElseIf(Condition){ Body }]
        [.Else{ Body }]

.. describe:: If

    .. describe:: Condition

    A condition is considered to be not fulfilled if it equals an *empty string*, *0* or *false*. In all other cases the condition is considered fulfilled.

    .. describe:: Body

        Child elements.

Example
"""""""

.. code:: js

      If(#value#){
         Span(Value)
      }.ElseIf(#value2#){Span(Value 2)
      }.ElseIf(#value3#){Span(Value 3)}.Else{
         Span(Nothing)
      }


.. _protypo-Image:

Image
-----

Creates an **image** HTML element.


Syntax
""""""

.. code-block:: text

    Image(Src, Alt, Class)
        [.Style(Style)]

.. describe:: Image

    .. describe:: Src

        Image source, file or ``data:...``.

    .. describe:: Alt

        Alternative text for the image.

    .. describe:: Сlass

        List of classes.

.. todo::

    Style not documented. What Class does?


Example
"""""""

.. code:: js

    Image(\images\myphoto.jpg)    


.. _protypo-ImageInput:

ImageInput
----------

Creates an **imageinput** element for image upload. In the third parameter you can specify either image height or aspect ratio to apply: *1/2*, *2/1*, *3/4*, etc. The default width is 100 pixels with *1/1* aspect ratio.


Syntax
""""""

.. code-block:: text

    ImageInput(Name, Width, Ratio, Format) 

.. describe:: ImageInput

    .. describe:: Name

        Element name.

    .. describe:: Width

        Width of the cropped image.

    .. describe:: Ratio

        Aspect ratio (width to height) or height of the image.

    .. describe:: Format

        Format of the uploaded image.


Example
"""""""

.. code:: js

   ImageInput(avatar, 100, 2/1)


.. _protypo-Include:

Include
-------

Inserts a template with a specified name to the page code. 

.. todo::

    How this is used?


Syntax
""""""

.. code-block:: text

    Include(Name)

.. describe:: Include

    .. describe:: Name

    Template name.


Example
"""""""

.. code:: js

      Div(myclass, Include(mywidget))
      

.. _protypo-Input:

Input
-----

Creates an **input** HTML element.

Syntax
""""""

.. code-block:: text

    Input(Name, Class, Placeholder, Type, Value, Disabled)
        [.Validate(validation parameters)]
        [.Style(Style)]

.. describe:: Input

    .. describe:: Name

        Element name.

    .. describe:: Class

        Classes for this *input*.

    .. describe:: Placeholder

        The *placeholder* element for this *input*.

    .. describe:: Type

        Type of the *input*.

    .. describe:: Value

        Element value.

    .. describe:: Disabled

        If the *input* is disabled or not.

        .. todo::

            Values? Like HTML?

.. describe:: Validate

    Validation parameters.

    .. todo::

        Syntax?

.. describe:: Style

    Specifies CSS styles.

    .. describe:: Style

        CSS styles.

Example
"""""""

.. code:: js

      Input(Name: name, Type: text, Placeholder: Enter your name)
      Input(Name: num, Type: text).Validate(minLength: 6, maxLength: 20)



.. _protypo-InputErr:

InputErr
--------

Creates an **inputerr** element with validation error texts.

.. todo::

    How this is used?


Syntax
""""""

.. code-block:: text

    InputErr(Name,validation errors)]

.. describe:: InputErr

    .. describe:: Name

        Name of the corresponding :ref:`protypo-Input` element.

    .. describe:: validation errors

        One or more parameters for validation error messages.


Example
"""""""

.. code:: js

      InputErr(Name: name, 
          minLength: Value is too short, 
          maxLength: The length of the value must be less than 20 characters)
      

.. _protypo-InputMap:

InputMap
--------

Creates a text input field for an address. Provides an ability to select coordinates on a map.

Syntax
""""""

.. code-block:: text

    InputMap(Name, Type, MapType, Value)

.. describe:: InputMap


    .. describe:: Name

        Element name.

    .. describe:: Value

        Default value.

        This value is an object in the string format. For example, ``{"coords":[{"lat":number,"lng":number},]}`` or ``{"zoom":int, "center":{"lat":number,"lng":number}}``. The *address* field can be used to save the address value for cases when InputMap is created with a predefined *Value*, so that address field is not empty.

    .. describe:: Type

        Use ``polygon`` value for this parameter.

    .. describe:: MapType

        Map type.

        This parameter can have the following values: ``hybrid``, ``roadmap``, ``satellite``, ``terrain``.


Example
"""""""

.. code:: js

    InputMap(Name: Coords,Type: polygon, MapType: hybrid, Value: `{"zoom":8, "center":{"lat":55.749942860682545,"lng":37.6207172870636}}`)


.. _protypo-JsonToSource:

JsonToSource
------------

Creates a **jsontosource** element and populates it with *key* - *value* pairs that were passed in a JSON oblect. The resulting data is put into the *Source* element, which can later be used in functions that use source inputs (such as :ref:`protypo-Table`).

The records in the resulting data is sorted by JSON keys, in alphabetical order. 

Syntax
""""""

.. code-block:: text

    JsonToSource(Source, Data)


.. describe:: JsonToSource
    
    .. describe:: Source

        Data source name.

    .. describe:: Data

        A JSON oblect or a name of a variable (``#name#``)that holds a JSON array.


Example
"""""""

.. code:: js

   JsonToSource(src, #myjson#)
   JsonToSource(dat, {"param":"value", "param2": "value 2"})


.. _protypo-Label:

Label
-----

Creates a **label** HTML element.

Syntax
""""""

.. code-block:: text

    Label(Body, Class, For)
        [.Style(Style)]

.. describe:: Label


    .. describe:: Body

        Child text or elements.

    .. describe:: Class

        Classes for this *label*.

    .. describe:: For

        This label's *for* value.

.. describe:: Style

    Specifies CSS styles.

    .. describe:: Style

        CSS styles.

Example
"""""""

.. code:: js

      Label(The first item).


.. _protypo-LangRes:

LangRes
-------

Returns a specified language resource. In case of request to a tree for editing it returns the **langres** element. A short notation in the ``$langres$`` format can be used.


Syntax
""""""

.. code-block:: text
    
    LangRes(Name, Lang)

.. describe:: LangRes


    .. describe:: Name

        Name of language resource.

    .. describe:: Lang

        Two-character language identifier.

        By default, the language defined in the *Accept-Language* request is returned. 

        Lcid identifiers can be specified, for example, *en-US,en-GB*. In this case, if the requested values will not be found, for example, for *en-US*, then the language resource will be looked for in *en*.


Example
"""""""

.. code:: js

      LangRes(name)
      LangRes(myres, fr)


.. _protypo-LinkPage:

LinkPage
--------

Creates a **linkpage** element – a link to a page.


Syntax
""""""

.. code-block:: text

    LinkPage(Body, Page, Class, PageParams)
        [.Style(Style)]


.. describe:: LinkPage

    .. describe:: Body

        Child elements or text.

    .. describe:: Page

        Page to redirect to.

    .. describe:: Class

        Classes for this button.

    .. describe:: PageParams

        Redirection parameters.


.. describe:: Style

    Specifies CSS styles.

    .. describe:: Style

        CSS styles

Example
"""""""

.. code:: js

      LinkPage(My Page, default_page, mybtn_class)


.. _protypo-Map:

Map
---

Creates a visual representation of a map and displays coordinates in an arbitrary format.

Syntax
""""""

.. code-block:: text

    Map(Hmap, MapType, Value)

.. describe:: Map

    .. describe:: Hmap

        HTML element height on a page.

        The default value is 100.

    .. describe:: Value

        Map value, an object in the string format.

        For example: ``{"coords":[{"lat":number,"lng":number},]}`` or ``{"zoom":int, "center":{"lat":number,"lng":number}}``. If ``center`` is not specified, then map window will be automatically adjusted for the specified coordinates.

    .. describe:: MapType

        Map type.

        This parameter can have the following values: ``hybrid``, ``roadmap``, ``satellite``, ``terrain``.


Example
"""""""

.. code:: js

      Map(MapType:hybrid, Hmap:400, Value:{"coords":[{"lat":55.58774531752405,"lng":36.97260184619233},{"lat":55.58396161622043,"lng":36.973803475831005},{"lat":55.585222890513975,"lng":36.979811624024364},{"lat":55.58803635636347,"lng":36.978781655762646}],"area":146846.65783403456,"address":"Unnamed Road, Moscow, Russia, 143041"})


.. _protypo-MenuGroup:

MenuGroup
---------

Creates a nested submenu in the menu and returns the **menugroup** element. The *name* parameter will also return the value of *Title* before replacement with language resources.


Syntax
""""""

.. code-block:: text

    MenuGroup(Title, Body, Icon)

.. describe:: MenuGroup

    .. describe:: Title

        Menu item name.

    .. describe:: Body

        Child elements in submenu.

    .. describe:: Icon

        Icon.


Example
"""""""

.. code:: js

      MenuGroup(My Menu){
          MenuItem(Interface, sys-interface)
          MenuItem(Dahsboard, dashboard_default)
      }


.. _protypo-MenuItem:

MenuItem
--------

Creates a menu item and returns the **menuitem** element. 

Syntax
""""""

.. code-block:: text

    MenuItem(Title, Page, Params, Icon, Vde)

.. describe:: MenuItem

    .. describe:: Title

        Menu item name.

    .. describe:: Page

        Page to redirect to.

    .. describe:: Params

        Parameters, passed to the page in the *var:value* format, separated by commas.

    .. describe:: Icon

        Icon.

    .. describe:: Vde

        This parameter that defines the transition to a virtual ecosystem. If ``Vde: true``, then the link redirects to VDE; if ``Vde: false``, then the link redirects to the blockchain; if the parameter was not specified, then it is defined based on where the menu was loaded.


Example
"""""""

.. code:: js

       MenuItem(Interface, interface)


.. _protypo-Money:

Money
-----

Returns a string value of ``exp/10^digit``. If *Digit* parameter is not specified, it is taken from the **money_digit** ecosystem parameter.


Syntax
""""""

.. code-block:: text

    Money(Exp, Digit)


.. describe:: Money

    .. describe:: Exp

        Numeric value as a string.

    .. describe:: Digit

        Exponent of the base 10 in the ``exp/10^digit`` expression. This value can be positive or negative. Positive value determines the number of digits after the comma.


Example
"""""""

.. code:: js

    Money(Exp, Digit)


.. _protypo-Now:

Now
---

This function returns the current time in the specified format, which by default is the UNIX format (number of seconds elapsed since January 1, 1970). If the requested time format is *datetime*, then date and time are shown as ``YYYY-MM-DD HH:MI:SS``. An interval can be specified in the second parameter (for instance, *+5 days*).


Syntax
""""""

.. code-block:: text

    Now(Format, Interval)

.. describe:: Now

    .. describe:: Format

        Output format with a desired combination of ``YYYY, MM, DD, HH, MI, SS`` or *datetime*.

    .. describe:: Interval

        Time offset, backward or forward in time.

        Example: ``+5 days``.


Example
"""""""

.. code:: js

    Now()
    Now(DD.MM.YYYY HH:MM)
    Now(datetime,-3 hours)


.. _protypo-Or:

Or
--

Returns a result of the **IF** logical operation with all parameters specified in parentheses and separated by commas. The parameter value is considered ``false`` if it equals an empty string (``""``), 0 or ``false``. In all other cases the parameter value is considered ``true``. The function returns 1 for true or 0 in all other cases. Element named **or** is created only when the tree for editing is requested. 

Syntax
""""""

.. code-block:: text

    Or(parameters)


Example
"""""""

.. code:: js

      If(Or(#myval1#,#myval2#), Span(OK))


.. _protypo-P:

P
-

Creates a **p** HTML element.

Syntax
""""""

.. code-block:: text

    P(Body, Class) 
        [.Style(Style)]

.. describe:: P

    .. describe:: Body

        сhild text or elements.

    .. describe:: Class

        classes for this *p*.


.. describe:: Style

    Specifies CSS styles.

    .. describe:: Style

        CSS styles.


Example
"""""""

.. code:: js

      P(This is the first line.
        This is the second line.)


.. _protypo-QRcode:

QRcode
------

Returns a *qrcode* element with a QR code for the specified text.

Syntax
""""""

.. code-block:: text

    QRcode(Text)

.. describe:: QRcode

    .. describe:: Text

        Text for the QR code.

Example
"""""""

.. code:: js

     QRcode(#name#)


.. _protypo-RadioGroup:

RadioGroup
----------

Creates a **radiogroup** element.

Syntax
""""""

.. code-block:: text

    RadioGroup(Name, Source, NameColumn, ValueColumn, Value, Class) 
        [.Validate(validation parameters)] 
        [.Style(Style)]

.. describe:: RadioGroup


    .. describe:: Name

        Element name.

    .. describe:: Source

        Data source name from :ref:`protypo-DBFind` or :ref:`protypo-Data` functions.

    .. describe:: NameColumn

        Column name to use as a source of element names.

    .. describe:: ValueColumn

        Column name to use as a source of element values. 

        Columns created using :ref:`Custom <protypo-Data>` must not be used in this parameter.

    .. describe:: Value

        Default value.

    .. describe:: Class

        Classes for the element.

.. describe:: Validate

    Validation parameters.

    .. todo::

        Syntax?

.. describe:: Style

    Specifies CSS styles.

    .. describe:: Style

        CSS styles.


Example
"""""""

.. code:: js

    DBFind(mytable, mysrc)
    RadioGroup(mysrc, name)   


.. _protypo-Range:

Range
-----

Creates a **range** element and fills it with integer values from *From* to *To* (*To* is not included) with a *Step* step. The resulting data is put into the *Source* element, which can later be used in functions that use source inputs (such as :ref:`protypo-Table`). Values are written to the *id* column. If invalid parameters are specified, an empty *Source* is returned.


Syntax
""""""

.. code-block:: text

    Range(Source,From,To,Step)

.. describe:: Range

    .. describe:: Source

        Data source name.

    .. describe:: From

        Starting value (i = From).

    .. describe:: To

        End value (i < To).

    .. describe:: Step

        Value change step. If this parameter is not specified a value of 1 is used.


Example
"""""""

.. code:: js

     Range(my,0,5)
     SetVar(from, 5).(to, -4).(step,-2)
     Range(Source: neg, From: #from#, To: #to#, Step: #step#)


.. _protypo-Select:

Select
------

Creates a **select** HTML element.

Syntax
""""""

.. code-block:: text

    Select(Name, Source, NameColumn, ValueColumn, Value, Class) 
        [.Validate(validation parameters)]
        [.Style(Style)]


.. describe:: Select

    .. describe:: Name

        Element name.

    .. describe:: Source

        Data source name from :ref:`protypo-DBFind` or :ref:`protypo-Data` functions.

    .. describe:: NameColumn

        Column name to use as a source of element names.

    .. describe:: ValueColumn

        Column name to use as a source of element values. 

        Columns created using :ref:`Custom <protypo-Data>` must not be used in this parameter.

    .. describe:: Value

        Default value.

    .. describe:: Class

        Classes for the element.

.. describe:: Validate

    Validation parameters.

    .. todo::

        Syntax?

.. describe:: Style

    Specifies CSS styles.

    .. describe:: Style

        CSS styles.

Example
"""""""

.. code:: js

    DBFind(mytable, mysrc)
    Select(mysrc, name) 


.. _protypo-SetTitle:

SetTitle
--------

Sets the page title. The element **settitle** IS be created.

Syntax
""""""

.. code-block:: text

    SetTitle(Title)

.. describe:: SetTitle

    .. describe:: Title

        Page title.

Example
"""""""

.. code:: js

     SetTitle(My page)


.. _protypo-SetVar:

SetVar
------

Assigns a *Value* to a *Name* variable.

Syntax
""""""

.. code-block:: text

    SetVar(Name, Value)

.. describe:: SetVar

    .. describe:: Name

        Variable name.

    .. describe:: Value

        Value of the variable, which can contain a reference to another variable.

Example
"""""""

.. code:: js

     SetVar(name, John Smith).(out, I am #name#)
     Span(#out#)      


.. _protypo-Span:

Span
----

Creates a **span** HTML element.

Syntax
""""""

.. code-block:: text

    Span(Body, Class)
        [.Style(Style)]

.. describe:: Span

    .. describe:: Body
        
        Child class or elements.
    
    .. describe:: Class
    
        Classes for this *span*.

.. describe:: Style

    Specifies CSS styles.

    .. describe:: Style

        CSS styles.

Example
"""""""

.. code:: js

      This is Span(the first item, myclass1).


.. _protypo-Strong:

Strong
------

Creates a **strong** HTML element.

Syntax
""""""

.. code-block:: text

    Strong(Body, Class)

.. describe:: Strong

    .. describe:: Body
        
        Child class or elements.
    
    .. describe:: Class
    
        Classes for this *strong*.

Example
"""""""

.. code:: js

      This is Strong(the first item, myclass1).


.. _protypo-SysParam:

SysParam
--------

Displays the value of a system parameter from the system_parameters table. 


Syntax
""""""

.. code-block:: text
    
    SysParam(Name) 

.. describe:: SysParam

    .. describe:: Name

        Parameter name.


Example
"""""""

.. code:: js

     Address(SysParam(founder_account))


.. _protypo-Table:

Table
-----

Creates a **table** HTML element.

Syntax
""""""

.. code-block:: text

    Table(Source, Columns)
        [.Style(Style)]

.. describe:: Table

    .. describe:: Source

        Data source name as specified, for example, in the *DBFind* command.

    .. describe:: Columns

        Headers and corresponding column names, as follows: ``Title1=column1,Title2=column2``.

.. describe:: Style

    Specifies CSS styles.

    .. describe:: Style

        CSS styles.


Example
"""""""

.. code:: js

    DBFind(mytable, mysrc)
    Table(mysrc,"ID=id,Name=name")


.. _protypo-TransactionInfo:

TransactionInfo
---------------

The function searches a transaction by the specified hash and returns information about the executed contract and its parameters.

Syntax
""""""

.. code-block:: text

    TransactionInfo(Hash)

.. describe:: TransactionInfo


    .. describe:: Hash

        Transaction hash in a hex string format.


Return value
""""""""""""

The function returns a string in the json format: 

  ``{"contract":"ContractName", "params":{"key": "val"}, "block": "N"}``

Above,  

  * *contract* - contract name
  * *params* - parameters passed to the contract
  * *block* - block ID where this transaction was processed.

Example
"""""""

.. code:: js

    P(TransactionInfo(#hash#))


Styles for mobile app
=====================

Typography
----------


Headings
""""""""

* ``h1`` ... ``h6``


Emphasis Classes
""""""""""""""""

* ``.text-muted``
* ``.text-primary``
* ``.text-success``
* ``.text-info``
* ``.text-warning``
* ``.text-danger``


Colors
""""""

* ``.bg-danger-dark``
* ``.bg-danger``
* ``.bg-danger-light``
* ``.bg-info-dark``
* ``.bg-info``
* ``.bg-info-light``
* ``.bg-primary-dark``
* ``.bg-primary``
* ``.bg-primary-light``
* ``.bg-success-dark``
* ``.bg-success``
* ``.bg-success-light``
* ``.bg-warning-dark``
* ``.bg-warning``
* ``.bg-warning-light``
* ``.bg-gray-darker``
* ``.bg-gray-dark``
* ``.bg-gray``
* ``.bg-gray-light``
* ``.bg-gray-lighter``


Grid
----

* ``.row``
* ``.row.row-table``
* ``.col-xs-1`` ... ``.col-xs-12`` works only when the parent has ``.row.row-table`` class


Panel
-----

* ``.panel``
* ``.panel.panel-heading``
* ``.panel.panel-body``
* ``.panel.panel-footer``


Form
----

* ``.form-control``


Button
------

* ``.btn.btn-default``
* ``.btn.btn-link``
* ``.btn.btn-primary``
* ``.btn.btn-success``
* ``.btn.btn-info``
* ``.btn.btn-warning``
* ``.btn.btn-danger``


Icons
-----

* All icons from FontAwesome: ``fa fa-<icon-name></icon-name>``.
* All icons from SimpleLineIcons: ``icon-<icon-name>``.
   
      
