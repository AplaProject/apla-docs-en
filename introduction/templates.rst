################################################################################
Pages template engine
################################################################################
********************************************************************************
General description of functions
********************************************************************************
Types of functions
==============================
The application pages templates are created with the help of a set of functions that can be regarded as a specialized language for creating eGaaS application interfaces. Functions can be divided into several groups according to the type of operations performed:

* obtaining values from the database;
* handling of formats and values of variables;
* presentation of data in tables and charts;
* building forms with the required set of fields for entry of the contracts data;
* display of the navigation elements and contracts displaying;
* creation of HTML page layout - different containers with the ability to specify css classes;
* implementation of the conditional display of page templates fragments; and
* creation of a multi-level menu.

Function formats
==============================
Two formats - FuncName() and FuncName{} - are used to implement the functions of page templates construction language. In the first case, the parameters are transferred as an array of strings, and in the second - as an associative array with named parameters (keys) and values. The values may not be enclosed in quotes. If the value contains a comma or a closing bracket, it should be enclosed in double or back (``) quote. If the function has no parameters or only one parameter, then a colon may be put instead of the parentheses: *MyFunc:param* ie equivalent to *MyFunc(param)*.

For instance,

.. code:: js

      FuncName( string 1, string 2, "Text, text")

.. code:: js

      FuncName{ ParamName1: string 1, ParamName2: string 2, ParamName3: "Text, text" }

КClasses of HTML elements
==============================
The functions for creating HTML elements of the page layout and the navigation system contain parameters for the names of the css classes, which shall be enumerated through a space. Currently, the classes from  `Angular Bootstrap Angle <http://themicon.co/theme/angle/v3.5.3/backend-jquery/app/buttons.html#/>`_ are used. In addition to classes, these parameters may include additional attributes that will be inserted into the relevant element. The attribute values shall be indicated using an equal mark. If the attribute has no value, then the equal mark shall be preserved, but nothing shall be indicated after it (the equal mark may be omitted only if the attribute starts with *data-*). For instance

.. code:: js

      Div(panel data-widget=panel-scroll myclass data-sweet-alert=, Text)

will be transformed into

.. code:: js

      <div class="panel myclass" data-sweet-alert data-widget="panel-scroll">Text</div>
   
Using language resources
==============================
The template engine supports the function of replacing the page text elements with an arbitrary language if the translation of this element is specified in the *languages* table Language Resources (the Languages in the Government dashboard menu). For translation, the name of the text resource must be framed with the signs **$**, for instance, $Call contract$, or the LangRes("Call contract") function may be used.

Such functions as Label(), Select(), StateValue(), as well as the menu creation functions automatically replace texts with language resources.


.. note::

      Functions in the future can be supplemented with new parameters.


********************************************************************************
Operations with variables
********************************************************************************

SetVar( name=value,.....)
==============================
The function assigns values to variables without displaying them on the page.

* *name* - name of the variable, 
* *value* - value; If the value contains commas, then it must be enclosed in backquotes  ``; if it is necessary to substitute the expression values, the format **#=** is used instead of **=**. 
For instance,
 
.. code:: js

      SetVar( var1= value1, var2 = "Значение 2", var3=10, `var4 #= #citizen_id#, #state_id#` )
 
In future, the variables may be referred as #var1#, #var2# …

StateLink(prefix,name) 
==============================
The function displays a meaning of the variable with the name prefix_name.

********************************************************************************
Operations with values
********************************************************************************

And(param, [param,...]) 
==============================
The function returns the result of a logical operation **AND** with all parameters listed in brackets separated by commas. The parameter value is assumed to be **false**, if it is equal to an empty string (""), 0 or *false*. In all other cases the parameter value is assumed to be **true**.

Or(param, [param,...]) 
==============================
The function returns the result of a logical operation **OR** with all parameters listed in brackets separated by commas. The parameter value is assumed to be **false**, if it is equal to an empty string (""), 0 or *false*. In all other cases the parameter value is assumed to be **true**.

CmpTime(time1,time2) 
==============================
The function compares two time values in the same format (preferably, the standard one - YYYY-MM-DD HH:MM:SS, but it can also be the arbitrary one, subject to preservation of a sequence, from years to seconds, for example, YYYYMMDD). It returns:

* **-1** - time1 < time2, 
* **0** - time1 = time2, 
* **1** - time1 > time2.

If(condition, iftrue, iffalse) 
==============================
The function displays one of two values depending on the truth or falsity of the condition.

* *condition* - the conditional expression, it takes a false value if equal to an empty string or 0;
* *iftrue* - the value returned if the condition is true;
* *iffalse* - the value returned if the condition is false.

The functions enclosure is allowed.

Mult(num1,num2) 
==============================
The function displays the result of multiplying two numbers (the parameters may be decimals), rounded to the nearest integer.

Trim(text) 
==============================
The function removes spaces and invisible characters from the beginning and from the end of the *text* line.

********************************************************************************
Value mapping
********************************************************************************

Address([wallet_id]) 
==============================
The function displays the result of multiplying two numbers (the parameters may be decimals), rounded to the nearest integer.

Money(value) 
==============================
The function displays the value in the money format, and the decimal places is defined by the **money_digit** parameter value in the state_parameters table.

Date(date,[format]) 
==============================
The function displays the date value in the specified format.

*  *date* - time in the standard format 2006-01-02T15:04:05
*  *format* - format template: YY short year, YYYY full year, MM - month, DD – day, for example, DD.MM.YY. If the format is not specified, then the *dateformat* parameter value, indicated in the languages table will be used, and if it is absent, then YYYY-MM-DD.

DateTime(datetime,[format]) 
==============================
The function displays the date and time value in the specified format.

*  *datetime* - time in the standard format  2006-01-02T15:04:05
*  *format* -  format template: YY short year, YYYY full year, MM - month, DD – day, for example, DD.MM.YY. If the format is not specified, then the *timeformat* parameter value, indicated in the languages table will be used, and if it is absent, then YYYY-MM-DD HH:MI:SS.

Now([format]) 
==============================
The function displays current time in the specified format, and it is displayed in UNIX-format (number of seconds since 1970) by default, if datetime is indicated as a format, then the date and time is displayed as YYYY-MM-DD HH:MI:SS.

********************************************************************************
HTML elements
********************************************************************************

A(class,text, href) 
==============================
The function creates a <a href="*href*">*text*</a> container with the specified classes (*class*)

Div(class,text) 
==============================
The function creates a <div>text</div> container with the specified classes (*class*)

Divs(class,[class,]) … DivsEnd 
==============================
The function encloses a part of the page template in the div containers enclosed into each other; the number of containers is equal to the number of parameters with the specified classes.

.. code:: js

      Divs(md-5, panel) 
          Any content.
      DivsEnd:


P(class, text) 
==============================
The function creates a <a href="*href*">*text*</a> container with the specified classes (*class*).

Em(class,text) 
==============================
The function creates a <em>*text*</em> container with the specified classes (*class*).

Small(class,text) 
==============================
The function creates a <small>*text*</small> container with the specified classes (*class*).

Span(class,text) 
==============================
The function creates a <span>*text*</span> container with the specified classes (*class*).

Strong(class,text) 
==============================
The function creates a <strong>*text*</strong> container with the specified classes (*class*).

Label(text,[class]) 
==============================
The function creates a <label>*text*</label> container with the specified classes. If the language resource with the value indicated in *text* is included in the languages table, the text will be automatically translated

Legend(class, text) 
==============================
The function creates a <legend>*text*</legend> container with the specified classes (*сlass*). 

Tag(tagname, [text], [class]) 
==============================
The function creates a <tagname>*text*</tagname> container with the specified classes; the h1-h6, button tags are supported.

Image(src, [alt], [class] ) 
==============================
The function inserts an image into the page.

*	*src* - image source indicator;
*	*alt* - alternative text for images;
*	*class* - list of classes.


MarkDown(text) 
==============================
The function converts into to HTML the text with markdown layout. For instance,

.. code:: js

      MarkDown(`## Header
            Any Text
      `)

Val(idname) 
==============================
The function returns the value of the HTML element by its identifier (id).

********************************************************************************
Conditional construct
********************************************************************************

If(condition) … Else … ElseIf … IfEnd 
==============================
A conditional construct allows displaying different fragments of the page template, depending on the truth or falsity of the conditional expression. **If** constructs may be the enclosed ones, for instance,

.. code:: js

      If(#value#) 
          Divs(myclass)
              If(#par#)
                 ...
              IfEnd:
          DivsEnd:
      ElseIf(#param2#)
          P(class, Text)
      Else:
          Divs(myclass2)
              .....
          DivsEnd:
      IfEnd:

********************************************************************************
Displaying form elements
********************************************************************************

Form(class) … FormEnd 
==============================
The function frames part of the page template with a <form>…</form> container with the indicated classes (*class*).

Input(idname,[class],[placeholder],[type],[value]) 
==============================
The function creates a form input field;

* *idname* - field identifier name;
* *class* - list of classes,
* *placeholder* - hint text,
* *type* - field type, text by default,
* *value* - default value

Textarea(idname,[class],[value]) 
==============================
The function displays the textarea type form field

* *idname* - field identifier name,
* *class* - list of classes,
* *value* - default value.


InputAddress(idname,[class],[value] ) 
==============================
The function creates a form field for entering the wallet address, when entering the address, the suggested options are shown in the drop-down list 

* *idname* - field identifier name,
* *class*  - list of classes,
* *value* - default value.

InputDate(idname,[class],[value] ) 
==============================
The function creates the form field for data and time entry.

* *idname* - field identifier name,
* *class* - list of classes,
* *value* - default value.

InputMoney(idname,[class],[value]) 
==============================
The function creates the form field for entering money values.

* *idname* - field identifier name,
* *class* - list of classes,
* *value* - default value.

Select(idname, list, [class], [value]) 
==============================
The function creates a <select> drop-down list. 

* *idname* - identifier, 
* *list* - transfers a list of values, 
* *value* - the list value, selected by default,
* *class*  - list of classes.

There are two options for defining the *list*: 

1. listing the list names by commas, and the value in <option … > will be equal to the sequence number of the name starting with 1;

2. obtaining values from the database table in the **tablename.column.idname** format, where tablename is the table name, column is the name of the column, whose values are displayed as the list names, idname -the name of the column, whose values are used as a value in <option … >. If *idname* is not indicated, the id column is used by default; the number of entries in the table may not be more than 50. 
If the language resources with the list name values are present in the languages table, they will be automatically translated.

TextHidden(idname,....) 
==============================
The function creates a lot of hidden textarea fields; the names separated by commas are set as identifiers (id); the field values are taken from the variables having the same name. For instance, if the variable #test# = "Line" is present, the TextHidden(test) will create a textarea with id="test" and the "Line" value.

Source(idname,[value]) 
==============================
The function displays a text input field with highlighting of operators, keywords, etc. It is used, for example, to edit contracts

* *idname* - identifier; 
* *value* - default value.

********************************************************************************
Obtaining values from the database
********************************************************************************

ValueById(table,idval,columns,[aliases]) 
==============================
The function takes values from the database table entry by the id value of the string.

* *table* - table name; 
* *idval* - id value of the entry obtained;
* *columns*  - the names of the columns separated by commas; the variables with column names will be created by default, to which the received values will be transferred; 
* *aliases*  - the variable names other than column names separated by commas in the same sequence as the column names, 
For instance,  * ValueById(#state_id#_citizens, #citizen#, "name,avatar", "FirstName,Image") *

GetList(name, table, colnames, [where], [order], [limit]) 
==============================
The function obtains entries from the table.

*  *name* - the name by which a particular record is extracted from the resulting list using **ListVal** or **ForList** functions;
*  *colnames* - the list of resulting columns separated by commas; first the column with the index shall be specified, and by this value the access to values **ListVal** and **ForList** will be implemented; 
*  *where*, *order *, *limit * - Condition, sorting and number of resulting strings.

ListVal(name, index, column]) 
==============================
The function returns the value from the list obtained by **GetList** function; 

* *name* - the name that was indicated in the *GetList* function shall be used as the parameter value;
* *index* - the value of the identifier of search by the first column, indicated in *GetList*; 
* *column* - the name of the column with the returned value.

ForList(name) … FormListEnd 
=============================
The function displays a full list of entries obtained using **GetList** function; the *name* that was specified in the *GetList* function shall be used as the name parameter value. The end of the one record display template is fixed by the **FormListEnd** closing function. The values of the entry columns contain variables of #name_column# type, where the name of the table column is indicated after the underscore; the #index# variable is available, which contains the sequence number of the entry, starting with 1.

.. code:: js

      GetList(my, #state#_mytable, "id,param,value")
      ForList(my)
          Divs(md-5, panel) 
             Strong(#my_index#: #my_ param #)
             P(pclass, #my_value#)
          DivsEnd: 
      ForListEnd:

GetOne(colname, table, where, [value]) 
==============================
The function returns the value from the database table by condition.

* *colname* - the name of the returned column;
* *table* - full name of the table (#state#_mytable); 
* *where* - condition,
* *value* - the condition value, if the *value* parameter is not indicated, then the *where* parameter shall contain the complete query.

GetRow(prefix, table, colname, [value]) 
==============================
The function forms a set of variables with values from the database table entry obtained by searching by specified field and value, or on request.

* *prefix* - the prefix used to form the names of variables into which the values of the resulting entry are written: the variables have the form *#prefix_id#, #prefix_name#*, where after the underscore the name of the table column is specified.
* *table* - full name of the table (#state#_mytable); 
* *colname* - the name of the column to search forthe entry;
* *value* - the value by which the entry is searched, if the *value* parameter is not specified, then the  *colname* parameter shall contain full "where" query to the table.

StateVal(name, [index]) 
==============================
The function displays the parameter value from the state_parameters table.

* *name* - the value name;
* *index* - the value sequence number, and if their list is indicated by a comma, for example, *gender | male,female*, then StateValue(gender, 2) will return *female*
If the language resource with the resulting name is available, then its value will be substituted.

Table 
==============================
The function creates a table with values from the database. The function has the named parameters, which are displayed in the shape buttons: 

* *Table* - table name;
* *Order* - column name for sorting table rows, an optional parameter;
* *Where* - sampling condition, an optional parameter;
* *Columns* - an array of displayed columns, consisting of a title and the values [[ColumnTitle, value],...]; the column values from the base table corresponding to the line are returned as a variable with the column name (#column_name#).

.. code:: js
     Table{
         Table:  citizens
         Order: id
         Columns: [[Avatar, Image(#avatar#)],  [ID, Address(#id#)],  [Name, #name#]]
     }

********************************************************************************
Display of contracts
********************************************************************************

BtnContract(contract, name, message, params, [class], [onsuccess], [pageparams]) 
==============================
The function creates a button, that when clicked, opens a modal window prompting to refuse or confirm the display of the contract. 

* *contract* - contract name;
* *name* - column name;
* *message* - text for modal window;
* *params* - parameters transferred to the contract;
* *class*  - list of button classes;
* *onsuccess* - the name of the page to which the shift should be made if the contract is successfully executed;
* *pageparams* - parameters transferred to the page in the *var:value* format separated by commas..

For instance, *BtnContract(DelContract, Delete, Delete Item?, "IdItem:id_item",'btn btn-default')*

TxButton 
==============================
The function creates a button that when clicking starts the contract execution. The function has the named parameters, which are displayed in the shape buttons:

* *Contract* - contract name;
* *Name* - button name, **Send** by default;
* *Class* - list of classes for the <div> container, with the button;
* *ClassBtn* - list of classes for the button;
* *Inputs* - list of values transferred to the contract. By default, the contract parameter values (*data* section) are taken from HTML elements (say, form fields) with the same identifiers (*id*). If the element identifiers differ from the contract parameter names, the assignment in  *Inputs: "contractField1=idname1, contractField2=idname2" format is used. The variable values in the format  *Inputs: "contractField1#=var1, contractField2=var2" (the values of the variables #var1# and #var2# will be transferred) may be assigned;
* *OnSuccess* - the name of the page to which the shift will be made in case of successful performance of the contract, and the parameters in the format *var:value* transferred to the page separated by commas, for instance * OnSuccess: "CompanyDetails, CompanyId:#CompanyId#" *;
* *Silent* - in case of value 1 - the display of the message on the successful completion of the contract;
* *AutoClose* - in case of value 1 - the automatic closure of the message on the successful completion of the contract.

For instance,

.. code:: js

      TxButton {
          Contract: MyContract,
          Inputs: 'Name=myname, Request #= myreq',
         OnSuccess: "MyPage, RequestId:# myreq#"
      }


TxForm 
==============================
The function creates a form for entering contract data. The function has the named parameters, which are displayed in the shape buttons:

* *Contract* - contract name;;
* *OnSuccess* - the name of the page to which the shift will be made in case of successful performance of the contract, and the parameters in the format *var:value* transferred to the page separated by commas, for instance * OnSuccess: "CompanyDetails, CompanyId:#CompanyId#" *;
* *Silent* - in case of value 1 - the message on the successful completion of the contract is displayed;
* *AutoClose* - in case of value 1 - the message on the successful completion of the contract is automatically closed.

.. code:: js

      TxForm {
          Contract: MyContract,
          OnSuccess: 'mypage'
      }

********************************************************************************
Navigation elements
********************************************************************************

Navigation( params, …) 
==============================
The function displays a panel with "bread crumbs" and the link to edit the current page **Edit**. Например, Navigation( LiTemplate(dashboard_default, citizen),government).

LinkPage(page,text,[params]) 
==============================
Функция создает ссылку  на страницу. Если *name* не указан, то текст ссылки будет такой же как *page*. C помощью этой же функции можно создавать ссылки на предопределенные страницы. В этом случае добавьте префикс **sys-** перед именем страницы. Например, *LinkPage(sys-interface, Interface)*. Кроме этого, эту функцию можно использовать для создания ссылок на приложения. Для этого перед именем страницы-приложения необходимо указывать **app-**.

* *page* - имя страницы;
* *text*  - текст ссылки;
* *params* - параметры передаваемые странице в формате *var:value* через запятую.

LiTemplate(page, [text], [params], [class]) 
==============================
The function creates the <li>*text*</li> container, containing a ling to a page

* *page* - page name;
* *text*  - link text;
* *params* - parameters transferred to the page in the *var:value* format separated by commas;
* *class*  - list of classes.

.. code:: js

      LiTemplate(mypage, Home page, global:1)
      

BtnPage(page, name,[params],[class], [anchor]) 
==============================
The function creates a button, that when clicking, allows shifting to the specified page. If the parameter with the class is not specified, then the buttons will have the *btn btn-primary classes*. The links to system pages may be created using the function. In this case, add sys- prefix before the page name. For instance, *BtnTemplate(sys-interface, Interface)*.

* *page* - shift page name; 
* *name* - column name;
* *params* - parameters transferred to the page;
* *class*  - list of button classes;
* *anchor* - an anchor (id of the page element) to scroll the page to the specified position.

BtnEdit( page, icon, [params] ) 
==============================
The function creates a button in the form of a gear with a link to the specified page and transfers the id as a parameter; it is used in the screen tables to refer to the elements editing. Для перехода на системные страницы или страницы приложений необходимо добавлять соответствено префиксы **sys-** и **app-**. Например,
*BtnEdit(sys-editPage, cog, "name: #name#, global: #global#")*.

Back(page, [params]) 
==============================
The function enters the display of the specified page in the display history.

* *page* - page name;
* *params* - parameters of the page display from history in the *var:value* format separated by commas.

********************************************************************************
Formatting the page template
********************************************************************************

PageTitle(header) … PageEnd() 
==============================
The function captures the page body and creates a panel with the header indicated in the *header* parameter 

Title(text) 
==============================
The function creates a header with the *content-heading* class..

FullScreen(state) 
==============================
The function converts the width of the page working area to the entire width of the window when the *state* parameter equals 1, and narrows the working area when the *state* is 0.

WhiteMobileBg(state) 
==============================
The function is an analogue of the **FullScreen** function for mobile devices; it converts the width of the page working area to the entire width of the window when the *state* parameter equals 1, and narrows the working area when the *state* is 0.

********************************************************************************
Organizing of a multi-level menu
********************************************************************************

MenuItem(title, page, [params], [icon]) 
==============================
The function creates a menu item. 

* *title* - the name of the menu item, If the language resource with the value indicated in *title* is included in the languages table, the text will be automatically translated;
* *page* - shift page name. To go to system pages, you need to specify prefixes **sys-** или **app-**;
* *params* - parameters transferred to the page in the *var:value* format separated by commas.
* *icon* - an icon.

MenuGroup(title,[idname],[icon]) … MenuEnd: 
==============================
The function forms a submenu in the menu. 

* *title* - the name of the menu item, If the language resource with the value indicated in *title* is included in the languages table, the text will be automatically translated;
* *idname* - the submenu identifier;
* *icon* - an icon.

.. code:: js

      MenuGroup(My Menu,mycitizen)
            MenuItem(Interface, sys-interface)
            MenuItem(Dahsboard, dashboard_default)
      MenuEnd:

MenuBack(title, [page]) 
==============================
The function replaces the link of shift to the parent menu (top menu item).

* *title* - the name of the menu item, If the language resource with the value indicated in text is included in the languages table, the text will be automatically translated;
* *page* - shift page name.


MenuPage(page) 
==============================
The function sets the page specified in the *page* parameter as the parent menu item

********************************************************************************
Data representation
********************************************************************************

Ring(count,[title],[size]) 
==============================
The function displays a circle with the *count* parameter value in the middle

* *title* - a title;
* *size* - the value size.

WiAccount(address) 
==============================
The function displays with the special design the account number (wallet address) transferred in the address parameter.

WiBalance(value, money) 
==============================
The function displays with the special design the *value* in the money format, and adds the currency designation specified in the *money* parameter.

WiCitizen(name, address, [image], [flag]) 
==============================
The function displays with the special design the information on the citizen.

* *name* - the name;
* *address*  - the wallet address, normalized to the 1234-...-5678 form;
* *name* - the image;
* *name* - country flag. 

Map(coords) 
==============================
The function displays on the page the google maps container with the coordinates indicated in the *coords* parameter. The container height is taken from the value of the predefined variable #hmap# (by default, 100 pixels), the width is stretched to the maximum possible value.

MapPoint(coords)
==============================
The function displays on the page the google maps container with a marker on the coordinates specified in the coords parameter. The container height is taken from the value of the predefined variable #hmap# (by default, 100 pixels), the width is stretched to the maximum possible value.

ChartPie 
==============================
The function displays a pie chart. The function has the named parameters, which are displayed in the shape buttons: 

* *Data* - data reflected by the diagram in the form of a list [[value,color,label],....]; each list item must contain three parameters: value, rrggbb color and signature; if this list is available, other parameters are ignored;
* *Columns* - the list of rrggbb colors separated by commas;
* *Table* - the name of the table from which the data will be taken;
* *FieldValue* - the name of the column with the values;
* *FieldLabel* - the name of the column with the signatures;
* *Order* - column name for sorting table rows, an optional parameter;
* *Where* - sampling condition, an optional parameter;
* *Limit* - offset and number of selected records, an optional parameter.

ChartBar 
==============================
The function displays the chart as columns. All parameters, except for *Data*, are identical to the **ChartPie** function.

********************************************************************************
Displaying language resources
********************************************************************************

LangJS(resname) 
==============================
The function creates the <span>*resname*</span> container with the language resource value. It is used to automatically substitute the language resources in the browser. (These are resources described in static/js/lang/*.js.)

LangRes(resname) 
==============================
The function returns the language resource with the specified name from the languages table. 

********************************************************************************
Service functions
********************************************************************************

BlockInfo(blockid) 
==============================
The function displays a link with a block number (blockid), that when clicked, will open a window with information about the block.

TxId(txname) 
==============================
The function returns the specified transaction identifier. For instance,

.. code:: js

      SetVar(
      type_new_page_id = TxId(NewPage),
      type_new_contract_id = TxId(NewContract)
      )

Json(data) 
==============================
The function creates a script tag with the jdata variable, and assigns of data specified in the data parameter in the json format. For instance,

.. code:: js

      Json(`param1: 1, param2: "string"`) 
      
we will obtain

.. code:: js

      var jdata = { param1: 1, param2: "string"}
