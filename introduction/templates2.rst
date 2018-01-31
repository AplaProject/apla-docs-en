################################################################################
User Interfaces
################################################################################

.. contents::
  :local:
  :depth: 2

********************************************************************************
Interfaces building
********************************************************************************
Integrated Development Environment of the Molis software client, which is created using the *JavaScript React library*, includes an interface editor and a virtual interface designer. Interface pages are the essential part of applications that provides for retrieval and display of data from database tables, creation of forms for receipt of user input data, passing data to contracts, and navigation between application pages. Interface pages, just as contracts, are stored in the blockchain, which ensures their protection from falsification when loading them in the software client.  

Interface Template Engine
==============================
Interface elements (pages and menus) are formed on Validating Nodes in a so-called *template engine* from templates created by programmers in the interface editor of the Molis software client. All interface pages are built using the Protypo functional language developed by platform developers. Interfaces are requested from nodes on the network using the *content* API command. What the template engine sends as a reply to such request is not an HTML page, but a JSON code comprised of HTML tags that form a tree in accordance with the template structure. For testing purposes, a POST request can be sent to ``api/v2/content`` with the *template* parameter containing the name of a template to process.

Creating Interface Templates
==============================
Interfaces can be created and edited using a specialized editor, available in the **Interface** section of administrative tools in Molis. The editor provides for:

- Writing codes of interface pages with highlighting of keywords of the Protypo template language,
- Selecting a menu, which will be displayed on the page,
- Editing the page menu,
- Configuring permission to edit the page (typically, by way of specifying the name of the contract with permissions in the *ContractConditions* function, or by direct indication of access rights in the *Change conditions* field),
- Launching a visual interface designer,
- Page preview.

Visual Interface Designer
-----------------------------
Visual Interface Designer allows for creating page designs without resorting to the interface source code in Protypo language. The Designer allows for setting the positions of form elements and text on the page using drag-and-drop, as well as configuring sizes and design of page blocks. The Designer provides a set of ready-to-use blocks for displaying typical data models: panels with headers, forms, and information panels. The program logics (receipt of data and conditional constructs) can be added in the page editor after the page design is created. (In the future, we plan to create a full-scale visual interface editor.)

Use of Styles
-----------------------------
By default, interface pages are displayed using Angular Bootstrap Angle classes. If needed, users can create their own styles. Storage of styles is implemented using a special stylesheet parameter of the ecosystem configuration table. 

Page Blocks
-----------------------------
To use typical code fragments on multiple interface pages there is an option to create page blocks and embed them in the interface code using the Insert command. Such blocks can be created and edited on the Interface page of the administrative section in Molis. For blocks, just as for pages, permissions for editing can be defined.

Language Resources Editor
-----------------------------
The Molis software client includes a mechanism for interface localization using a special function of the Protypo template language – LangRes, which substitutes the language resource labels on the page with corresponding text lines in the language selected by the user in the software client (or browser for the web-version of the client). A shorter syntax $lable$ can be used instead of the LangRes function. Translation of messages in pop-up windows, initiated by contracts, is carried out by the LangRes function of the Simvolio language.

Language resources can be created and edited in the Language resources section of the administrative tools of the Molis software client. A language resource consists of a label (name) and the translations of this name into different languages with the indication of corresponding two-character language identifiers (EN, FR, JP, etc.).

Rights to add and change language resources can be configured using the same way as for any other table in the languages table (Tables section of the Molis administrative tools). 

********************************************************************************
Protypo Template Language
********************************************************************************

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

Overview of the Template Language Protypo
==============================
Page template language is a functional language that allows for calling functions using ``FuncName(parameters)``, and for nesting functions into each other. Parameters can be specified without quote marks. Unnecessary parameters can be dropped.

.. code:: js

      Text MyFunc(parameter number 1, parameter number 2) another text.
      MyFunc(parameter 1,,,parameter 4)
      
If a parameter contains a comma, it should be enclosed in quotes marks (back quotes or double quotes). If a function can have only one parameter, commas can be used in it without quotes.  Also, quotes should be used in case a parameter has an unpaired closing parenthesis.

.. code:: js

      MyFunc("parameter number 1, the second part of first paremeter")
      MyFunc(`parameter number 1, the second part of first paremeter`)
      
If you put a parameter in quotes, but a parameter itself includes quotes, then you can use different type of quotes or double them in the text.
      
      .. code:: js

      MyFunc("parameter number 1, ""the second part of first"" paremeter")
      MyFunc(`parameter number 1, "the second part of first" paremeter`)
      
In description of functions, every parameter has a specific name. You can call functions and specify parameters in the order they were declared, or specify any set of parameters in any order by their names: ''Parameter_name: Parameter_value''. This approach allows to safely add new function parameters without breaking the compatibility with current templates. For example, all of these calls are correct in terms of language use for a function described as ''MyFunc(Class,Value,Body)'':

.. code:: js

      MyFunc(myclass, This is value, Div(divclass, This is paragraph.))
      MyFunc(Body: Div(divclass, This is paragraph.))
      MyFunc(myclass, Body: Div(divclass, This is paragraph.))
      MyFunc(Value: This is value, Body: 
           Div(divclass, This is paragraph.)
      )
      MyFunc(myclass, Value without Body)
      
Functions can return text, generate HTML elements (for instance, ''Input''), or create HTML elements with nested HTML elements (''Div, P, Span''). In the latter case a parameter with a pre-defined name **Body** should be used to define nested elements. For example, two *div*, nested in another *div*, can look like this:

.. code:: js

      Div(Body:
         Div(class1, This is the first div.)
         Div(class2, This is the second div.)
      )
      
To define nested elements, which are described in the *Body* parameter, the following representation can be used: ``MyFunc(...){...}``. Nested elements should be specified in curly braces. 

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
     
The following variables are predefined 

* ``#key_id#`` - current user account identifier,
* ``#ecosystem_id#`` - current ecosystem identifier.

Passing parameters to a page using PageParams
-----------------------------
There is a number of functions that support the **PageParams** parameter, which serves for passing parameters when redirecting to a new page. For example, ``PageParams: "param1=value1,param2=value2"``. Parameter values can be both simple strings or rows with value substitution. When parameters are passed to a page, variables with parameter names are created; for example, ``#param1#`` and ``#param2#``.  

* ``PageParams: "hello=world"`` - the page will receive the hello parameter with world as value,
* ``PageParams: "hello=#world#"`` - the page will receive the hello parameter with the value of the world variable.

Additionally, the **Val** function allows for obtaining data from forms, which were specified in redirect. In this case,

* ``PageParams: "hello=Val(world)"`` - the page will receive the hello parameter with the value of the world form element.


Calling Contracts
-----------------------------
Protypo implements contract calling by clicking on a button in a form (*Button* function). Once  this event is initiated, the data entered by the user in the fields of the interface forms is passed to the contract (if the names of form fields correspond to the names of variables in the data section of the called contract, data is transferred automatically). The Button function allows for opening a modal window for user verification of the contract execution (Alert), and initiation of redirect to a specified page after the successful execution of the contract, and passing certain parameters to this page.    

********************************************************************************
Functions of Protypo
********************************************************************************

Operations with variables
==============================
GetVar(Name)
------------------------------
This function returns the value of the current variable if it exists, or returns an empty string if a variable with this name is not defined. An element with **getvar** name is created only when a tree for editing is requested. The difference between ``GetVar(varname)`` and ``#varname#`` is that in case *varname* does not exist, *GetVar* will return an empty string, whereas *#varname#* will be interpreted as a string value.

* *Name* - variable name.

.. code:: js

     If(GetVar(name)){#name#}.Else{Name is unknown}
      
SetVar(Name, Value)
------------------------------
Assigns a *Value* to a *Name* variable. 

* *Name* - name of the variable,
* *Value* - value of the variable, which can contain a reference to another variable.

.. code:: js

     SetVar(name, John Smith).(out, I am #name#)
     Span(#out#)      

Navigation
==============================     
AddToolButton(Title, Icon, Page, PageParams)
------------------------------
Adds a button to the buttons panel. Creates **addtoolbutton** element. 

* *Title* - button title,
* *Icon* - icon for the icon,
* *Page* - page name for the jump,
* *PageParams* - parmeters for the page.

.. code:: js

      AddToolButton(Help, help, help_page) 
      
Button(Body, Page, Class, Contract, Params, PageParams) [.Alert(Text,ConfirmButton,CancelButton,Icon)] [.Style(Style)]
------------------------------
Creates a **button** HTML element. This element creates a button, which sends a specified contract for execution.

* *Body* - child text or elements,
* *Page* - name of the page to redirect to,
* *Class* - classes for the button,
* *Contract* - name of the contract to execute,
* *Params* - list of values to pass to the contract. By default, values of contract parameters (data ``section``) are obtained from HTML elements (for example, input fields) with similarly-named identifiers (``id``). If the element identifiers differ from the names of contract parameters, then the assignment in the ``contractField1=idname1, contractField2=idname2`` format should be used. This parameter is returned to *attr* as an object ``{field1: idname1, field2: idname2}``,
* *PageParams* - parameters for redirection to a page in the following format: ``contractField1=idname1, contractField2=idname2``. In this case, variables with parameter names ``#contractField1#`` and ``#contractField2`` are created on the target page, and are assigned the specified values (see the parameter passing specifications in the "*Passing Parameters to a Page Using PageParams*" section above).

**Alert** - displays a message.

* *Text* - message text,
* *ConfirmButton* - confirm button caption,
* *CancelButton* - cancel button caption,
* *Icon* - icon.

**Style** - serves for specifying css styles.

* *Style* - css styles.

.. code:: js

      Button(Submit, default_page, mybtn_class).Alert(Alert message)
      Button(Contract: MyContract, Body:My Contract, Class: myclass, Params:"Name=myid,Id=i10,Value")
      
LinkPage(Body, Page, Class, PageParams) [.Style(Style)]
------------------------------
Creates a **linkpage** element – a link to a page.
 
* *Body* - child text or elements,
* *Page* - page to redirect to,
* *Class* - classes for this button,
* *PageParams* - redirection parameters,

**Style** - specifies css styles,

* *Style* - css styles.

.. code:: js

      LinkPage(My Page, default_page, mybtn_class)

Data Operations
==============================
And (Parameters)
------------------------------
This function returns the result of execution of the **and** logical operation with all parameters listed in parentheses and separated by commas. The parameter value will be ``false`` if it equals an empty string (``""``), zero or *false*. In all other cases the parameter value is ``true``. The function returns 1 if true or 0 in all other cases. The element named ``and`` is created only when a tree for editing is requested. 

.. code:: js

      If(And(#myval1#,#myval2#), Span(OK))
      
Calculate(Exp, Type, Prec)
------------------------------
This function returns the result of an arithmetic expression passed in the **Exp** parameter. The following operations can be used: +, -, *, /, and parenthesis (). 

* **Exp** - arithmetic expression. Can contain numbers and *#name#* variables.
* **Type** - result data type: **int, float, money**. If not specified, then the result type will be *float* in case there are numbers with a decimal point, or *int* in all other cases.
* **Prec** - the number of significant digits after the point can be specified for *float* and *money* types.

Calculate( Exp: (342278783438+5000)*(#val#-932780000), Type: money, Prec:18 )
Calculate(10000-(34+5)*#val#)
Calculate("((10+#val#-45)*3.0-10)/4.5 + #val#", Prec: 4)      

CmpTime(Time1, Time2)
------------------------------
This function compares two time values in the same format (preferably, standard format - YYYY-MM-DD HH:MM:SS, but any format can be used provided that the sequence is followed from years to seconds). Returns:

* **-1** - Time1 < Time2, 
* **0** - Time1 = Time2, 
* **1** - Time1 > Time2.

.. code:: js

     If(CmpTime(#time1#, #time2#)<0){...}
     
DateTime(DateTime, Format)
------------------------------
This function displays time and date in the specified format. 

 *  *DateTime* - time and date in standard format ``2006-01-02T15:04:05``.
 *  *Format* -  format template: ``YY`` 2-digit year format, ``YYYY`` 4-digit year format, ``MM`` - month, ``DD`` - day, ``HH`` - hours, ``MM`` - minutes, ``SS`` – seconds. Example: ``YY/MM/DD HH:MM``. If the format is not specified, the *timeformat* parameter value set in the *languages* table will be used. If this parameter is absent, the ``YYYY-MM-DD HH:MI:SS`` format will be used instead.
 
 .. code:: js

    DateTime(2017-11-07T17:51:08)
    DateTime(#mytime#,HH:MI DD.MM.YYYY)

Now(Format, Interval) 
------------------------------
This function returns the current time in the specified format, which by default is the UNIX format (number of seconds elapsed since January 1, 1970). If the requested time format is *datetime*, then date and time are shown as ``YYYY-MM-DD HH:MI:SS``. An interval can be specified in the second parameter (for instance, *+5 days*).

* *Format* - output format with a desired combination of ``YYYY, MM, DD, HH, MI, SS`` or *datetime*,
* *Interval* - backward or forward time offset.

.. code:: js

       Now()
       Now(DD.MM.YYYY HH:MM)
       Now(datetime,-3 hours)

Or(parameters)
------------------------------
This function returns a result of the **IF** logical operation with all parameters specified in parentheses and separated by commas. The parameter value is considered ``false`` if it equals an empty string (``""``), 0 or ``false``. In all other cases the parameter value is considered ``true``. The function returns 1 for true or 0 in all other cases. Element named **or** is created only when the tree for editing is requested. 

.. code:: js

      If(Or(#myval1#,#myval2#), Span(OK))

Displaying data
==============================
Code(Text)
------------------------------
Creates a **code** element for displaying the specified code.
	
* *Text* - source code, which will be displayed.

.. code:: js

      Code( P(This is the first line.
          Span(This is the second line.))
      )  
      
Chart(Type, Source, FieldLabel, FieldValue, Colors)
------------------------------
Creates an HTML diagram.

* *Type* - diagram type,
* *Source* - name of the data source, for example, from the *DBFind* command,
* *FieldLabel* - name of the field used for headers,
* *FieldValue* - name of the field used for values,
* *Colors* - list of used colors.

.. code:: js

      Data(mysrc,"name,count"){
          John Silver,10
          "Mark, Smith",20
          "Unknown ""Person""",30
      }
      Chart(Type: "bar", Source: mysrc, FieldLabel: "name", FieldValue: "count", Colors: "red, green")
	  
ForList(Source, Body)
------------------------------
Displays a list of elements from the *Source* data source in the template format set out in *Body*, and creates the **forlist** element.

* *Source* - data source from *DBFind* or *Data* functions,
* *Body* - a template to insert the elements in.

.. code:: js

      ForList(mysrc){Span(#name#)}
      
Image(Src,Alt,Class) [.Style(Style)]
------------------------------
Creates an **image** HTML element.
 
* *Src* - image source, file or ``data:...``,
* *Alt* - alternative text for the image,
* *Сlass* - list of classes.

.. code:: js

    Image(\images\myphoto.jpg)    
    
MenuGroup(Title, Body, Icon) 
------------------------------
Forms a nested submenu in the menu and returns the **menugroup** element. The *name* parameter will also return the value of *Title* before replacement with language resources.

* *Title* - menu item name,
* *Body* - child elements in submenu,
* *Icon* - icon.

.. code:: js

      MenuGroup(My Menu){
          MenuItem(Interface, sys-interface)
          MenuItem(Dahsboard, dashboard_default)
      }
      
MenuItem(Title, Page, Params, Icon, Vde) 
------------------------------
Creates a menu item and returns the **menuitem** element. 

* *Title* - menu item name,
* *Page* - page to redirect to,
* *Params* - parameters, passed to the page in the *var:value* format, separated by commas,
* *Icon* - icon,
* *Vde* -  is a parameter that defines the transition to a virtual ecosystem. If ``Vde: true``, then the link redirects to VDE; if ``Vde: false``, then the link redirects to the blockchain; if the parameter was not specified, then it is defined based on where the menu was loaded.

.. code:: js

       MenuItem(Interface, interface)
       
Table(Source, Columns) [.Style(Style)]
------------------------------
Создает HTML элемент **table**.

* *Source* - data source name as specified, for example, in the *DBFind* command,
* *Columns* - Headers and corresponding column names, as follows: ``Title1=column1,Title2=column2``.

**Style** - specifies css styles,

* *Style* - css styles.

.. code:: js

      DBFind(mytable, mysrc)
      Table(mysrc,"ID=id,Name=name")
      
Receiving data
==============================
Address (account)
------------------------------
This function returns the account address in the ``1234-5678-...-7990`` format given the numerical value of the address; if the address is not specified, the address of the current user will be taken as the argument. 

.. code:: js

      Span(Your wallet: Address(#account#))

Data(Source,Columns,Data) [.Custom(Column,Body)]
------------------------------
Creates element **data** and fills it with specified data and put into the *Source*, that then should be specified in *Table* and other commands resivieng *Source* as the input data. The sequence of column names corresponds to that of *data* entry values.
 
* *Source* - data source name. You can specify any name, which will have to be included in other commands later on (ex. *Table*) as a data source,
* *Columns* - list of columns,
* *Data* - one data entry per line, divided into columns by commas. Data should be in the same order as set in *Columns*, Entry values can be embraced in double quotes. If you need to use quote marks in the text, use double quotes.
* **Custom** - allows for assigning calculated columns for data. For example, you can specify a template for buttons and additional page layout elements. Several calculated columns can be assigned. As a rule, these fields are assigned for output to *Table* and other commands that use received data,
 
  * *Column* - column name. A unique name should be assigned,
  * *Body* - a code fragment. You can obtain values from other columns in this entry using ``#columnname#`` and use them in this code fragment.

.. code:: js

    Data(mysrc,"id,name"){
	"1",John Silver
	2,"Mark, Smith"
	3,"Unknown ""Person"""
     }.Custom(link){Button(Body: View, Class: btn btn-link, Page: user, PageParams: "id=#id#"}    


DBFind(table, Source) [.Columns(columns)] [.Where(conditions)] [.WhereId(id)] [.Order(name)] [.Limit(limit)] [.Offset(offset)] [.Ecosystem(id)] [.Custom(Column,Body)][.Vars(Prefix)]
------------------------------
Creates the **dbfind** element, fills it with data from the *table* table, and puts it to the *Source* structure. The *Source* structure can be then used in *Table* and other commands that receive *Source* as input data. The sequence of records in *data* should correspond to the sequence of column names.

* *Name* - table name,
* *Source* - arbitrary data source name,
 
* **Columns** - list of columns to be returned. If not specified, all columns will be returned,
* **Where** - search condition. For example, ``.Where(name = '#myval#')``,
* **WhereId** - search by ID. For example, ``.WhereId(1)``,
* **Order** - sort by this field,
* **Limit** - number of returned rows. Default value = 25, maximum value = 250,
* **Offset** - offset of returned rows,
* **Ecosystem** - ecosystem ID. By default, data is taken from the specified table in the current ecosystem,
* **Custom** - allows for assigning calculated columns for data. For example, you can specify a template for buttons and additional page layout elements. You can assign any number of calculated columns. As a rule, these fields are assigned for output to *Table* and other commands that use received data,
 
  * *Column* - column name. A unique name should be assigned,
  * *Body* - a code fragment. You can obtain values from other columns in this entry using **#columnname#** and use them in this code fragment.
  
  * **Vars** - the function generates a set of variables with values from the database table, obtained from this query. When specifying this function, the *Limit* parameter automatically becomes equal to 1 and only one record is returned,

* *Prefix* - * *Prefix* - prefix function is used to generate names for variables, to which the values of the resulting row are saved: variables are of format *#prefix_id#, #prefix_name#*, where the column name follows the underscore sign.

.. code:: js

    DBFind(parameters,myparam)
    DBFind(parameters,myparam).Columns(name,value).Where(name='money')
    DBFind(parameters,myparam).Custom(myid){Strong(#id#)}.Custom(myname){
       Strong(Em(#name#))Div(myclass, #company#)
    }
    
EcosysParam(Name, Index, Source) 
------------------------------
This function gets a parameter value from the parameters table of the current ecosystem. If there is a language resource for the resulting name, it will be translated accordingly.
 
* *Name* - value name,
* *Index* - in cases where the requested parameter is a list of elements separated by commas, you can specify an index starting from 1. For example, if ``gender = male,female``, then ``EcosysParam(gender, 2)`` will return *female*,  
* *Source* - you can receive the parameter values separated by commas as a *data* object. After that you will be able to specify this list as a data source for both *Table* and *Select*. If you specify this parameter, then the function will return a list as a *Data* object, not a separate value.

.. code:: js

     Address(EcosysParam(founder_account))
     EcosysParam(gender, Source: mygender)
 
     EcosysParam(Name: gender_list, Source: src_gender)
     Select(Name: gender, Source: src_gender, NameColumn: name, ValueColumn: id)
     
LangRes(Name, Lang)
------------------------------
Returns a specified language resource. In case of request to a tree for editing it returns the ``$langres$`` element.

* *Name* - name of language resource,
* *Lang* - by default, returned is the language defined in request to *Accept-Language*. You can specify your own two-character language identifier.

.. code:: js

      LangRes(name)
      LangRes(myres, fr)     

SysParam(Name) 
------------------------------
Displays the value of a system parameter from the system_parameters table. 

* *Name* - parameter name.

.. code:: js

     Address(SysParam(founder_account))

Elements of data formatting
============================== 
Div(Class, Body) [.Style(Style)]
------------------------------
Creates a **div** HTML element.

* *Class* - classes for this *div*,
* *Body* - child elements.

**Style** - serves for specifying css styles,

* *Style* - css styles.

.. code:: js

      Div(class1 class2, This is a paragraph.)
      
Em(Body, Class)
------------------------------
Creates an **em** HTML element.

* *Body* - child text or elements,
* *Class* - classes for this *em*.

.. code:: js

      This is an Em(important news).
      
P(Body, Class)
------------------------------
Creates a **p** HTML element.

* *Body* - child text or elements,
* *Class* - classes for this *p*,

**Style** - specifies css styles,

* *Style* - css styles.

.. code:: js

      P(This is the first line.
        This is the second line.)
	
SetTitle(Title)
------------------------------
Sets the page title. The element **settitle** will be created.

* *Title* - page title.

.. code:: js

     SetTitle(My page)	
	
Label(Body, Class, For) [.Style(Style)]
------------------------------
Creates a **label** HTML element.

* *Body* - child text or elements,
* *Class* - classes for this *label*,
* *For* - this label's *for* value,

**Style** - serves for specifying css styles,

* *Style* - css styles.

.. code:: js

      Label(The first item).	
	
Span(Body, Class) [.Style(Style)]
------------------------------
Creates a **span** HTML element.

* *Body* - child class or elements,
* *Class* - classes for this *span*,

**Style** - specifies css styles,

* *Style* - css styles.

.. code:: js

      This is Span(the first item, myclass1).
      
Strong(Body, Class)
------------------------------
Creates a **strong** HTML element.

* *Body* - child text or elements,
* *Class* - classes for this *strong*.

.. code:: js

      This is Strong(the first item, myclass1).
      
Elements of forms
==============================      
Form(Class, Body) [.Style(Style)]
------------------------------
Creates a **form** HTML element.

* *Class* - classes for this *form*,
* *Body* - child elements.

**Style** - specifies css styles.

* *Style* - css styles.

.. code:: js

      Form(class1 class2, Input(myid))
      
ImageInput(Name, Width, Ratio, Format) 
------------------------------
This function creates an **imageinput** element for image upload. In the third parameter you can specify either image height or aspect ratio to apply: *1/2*, *2/1*, *3/4*, etc. The default width is 100 pixels with *1/1* aspect ratio.

* *Name* - element name,
* *Width* - width of cropped image,
* *Ratio* - aspect ratio (width to height) or height of the image,
* *Format* - format of the uploaded image,

.. code:: js

   ImageInput(avatar, 100, 2/1)    
   
Input(Name,Class,Placeholder,Type,Value) [.Validate(validation parameters)] [.Style(Style)]
------------------------------
Creates an **input** HTML element.

* *Name* - element name,
* *Class* - classes for the *input*,
* *Placeholder* - *placeholder* for the *input*,
* *Type* - *input* type,
* *Value* - element value.

**Validate** - validation parameters.

**Style** - serves for specifying css styles.

* *Style* - css styles.

.. code:: js

      Input(Name: name, Type: text, Placeholder: Enter your name)
      Input(Name: num, Type: text).Validate(minLength: 6, maxLength: 20)

InputErr(Name,validation errors)]
------------------------------
Creates an **inputerr** element with validation error texts.

* *Name* - name of the corresponding **Input** element.

.. code:: js

      InputErr(Name: name, 
          minLength: Value is too short, 
          maxLength: The length of the value must be less than 20 characters)
	  

RadioGroup(Name, Source, NameColumn, ValueColumn, Value, Class) [.Validate(validation parameters)] [.Style(Style)]
------------------------------
Creates a **radiogroup** element.

* *Name* - element name,
* *Source* - data source name from *DBFind* or *Data* functions,
* *NameColumn* - column name to use a source of element names,
* *ValueColumn* - column name to use a source of element values. Columns created using Custom should not be used in this parameter,
* *Value* - default value,
* *Class* - classes for the element,

**Validate** - validation parameters,

**Style** - specification of css styles,
 
* *Style* - css styles.

.. code:: js

      DBFind(mytable, mysrc)
      RadioGroup(mysrc, name)	  
      
Select(Name, Source, NameColumn, ValueColumn, Value, Class) [.Validate(validation parameters)] [.Style(Style)]
------------------------------
Creates a **select** HTML element.

* *Name* - element name,
* *Source* - data source name. For example, *DBFind* or *Data*,
* *NameColumn* - column from which the element names will be taken,
* *ValueColumn* - column from which the element values will be taken. Columns created using Custom should not be specified in this parameter,
* *Value* - default value,
* *Class* - element classes,

**Validate** - validation parameters,

**Style** - specification of css styles,

* *Style* - css styles.

.. code:: js

      DBFind(mytable, mysrc)
      Select(mysrc, name) 
      
Operations with code
=========================
If(Condition){ Body } [.ElseIf(Condition){ Body }] [.Else{ Body }]
------------------------------
Conditional statement. Returned are child elements of the first ``If`` or ``ElseIf`` with fulfilled ``Condition``. Otherwise, returned are child elements of ``Else``, if it exists.

* *Condition* - a condition is considered non-fulfilled if it equals an *empty string*, *0* or *false*. In other cases the condition is considered true.
* *Body* - child elements.

.. code:: js

      If(#value#){
         Span(Value)
      }.ElseIf(#value2#){Span(Value 2)
      }.ElseIf(#value3#){Span(Value 3)}.Else{
         Span(Nothing)
      }
   
Include(Name)
------------------------------
This command inserts a template with name *Name* in the code of a page. 

* *Name* - name of the block.

.. code:: js

      Div(myclass, Include(mywidget))
      
************************************************
Styles for mobile app
************************************************

Typography
==============================

Headings
------------------------------

* ``h1`` ... ``h6``

Emphasis Classes
------------------------------

* ``.text-muted``
* ``.text-primary``
* ``.text-success``
* ``.text-info``
* ``.text-warning``
* ``.text-danger``

Colors
------------------------------

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
==============================
* ``.row``
* ``.row.row-table``
* ``.col-xs-1`` ... ``.col-xs-12`` works only when the parent has ``.row.row-table`` class

Panel
==============================

* ``.panel``
* ``.panel.panel-heading``
* ``.panel.panel-body``
* ``.panel.panel-footer``

Form
==============================

* ``.form-control``

Button
==============================

* ``.btn.btn-default``
* ``.btn.btn-link``
* ``.btn.btn-primary``
* ``.btn.btn-success``
* ``.btn.btn-info``
* ``.btn.btn-warning``
* ``.btn.btn-danger``

Icons
==============================

All icons from FontAwesome: ``fa fa-<icon-name></icon-name>``

All icons from SimpleLineIcons: ``icon-<icon-name>``
   
      
