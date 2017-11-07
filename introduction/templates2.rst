################################################################################
Page Template Engine V2
################################################################################
********************************************************************************
Updates in the new version
********************************************************************************
Due to the transition of the interface communications to API-only and the transition of the interface itself to the new platform (**React**), we had to change the result of the template engine operation. Also, the new version had to reflect comments and suggestions sent to us by users of the first version of the template engine
Unlike the first version of the template engine, which produced ready-to-use HTML pages, the Page Template Engine V2 produces trees with HTML tags, positioned in accordance with their description in the template engine. To produce such a tree for pages and menus, please use the content command from API V2. Also, for testing purposes, you can send POST requests to **api/v2/content**, specifying the template to be processed in the *template* parameter.
Please note, that Page Template Engine V2 does not any more have two function types – () and {}. From now on, you can use only calls (), but for all calls there is an option to specify parameters by their names. It will be discussed in more detail further in this document. The list of functions is subject to be changed later.

********************************************************************************
Overview of the Template Engine
********************************************************************************
Function types
==============================
Templates of application pages can be created using a set of functions that can be regarded as a specialized language for creation of application interfaces on APLA. Functions can be divided into several groups based on their types:

* reading values from the database;
* working with formats and values of variables;
* data representation as tables and diagrams;
* building forms with any set of fields for input of contract data;
* display of navigation elements and calling contracts;
* creation of HTML layouts – various containers with CSS classes;
* implementation of conditional display of page template parts; 
* creation of multi-level menus.

Overview of the Template Language
==============================
Page template language is a functional language that allows for calling functions using FuncName(parameters), and for nesting functions into each other. Parameters can be specified without quote marks. Unnecessary parameters can be dropped.

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
      
In description of functions, every parameter has a specific name. You can call functions and specify parameters in the order they were declared, or specify any set of parameters in any order by their names: **Parameter_name: Parameter_value**. This approach allows to safely add new function parameters without breaking the compatibility with current templates. For example, all of these calls are correct in terms of language use for a function described as **MyFunc(Class,Value,Body)**:

.. code:: js

      MyFunc(myclass, This is value, Div(divclass, This is paragraph.))
      MyFunc(Body: Div(divclass, This is paragraph.))
      MyFunc(myclass, Body: Div(divclass, This is paragraph.))
      MyFunc(Value: This is value, Body: 
           Div(divclass, This is paragraph.)
      )
      MyFunc(myclass, Value without Body)
      
Functions can return text, generate HTML elements (for instance, *Input*), or create HTML elements with nested HTML elements (*Div, P, Span*). In the latter case a parameter with a pre-defined name **Body** should be used to define nested elements. For example, two *div*, nested in another *div*, can look like this:

.. code:: js

      Div(Body:
         Div(class1, This is the first div.)
         Div(class2, This is the second div.)
      )
      
To define nested elements, which are described in the *Body* parameter, the following representation can be used: **MyFunc(...){...}**. Nested elements should be specified in curly braces. 

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
     
The language allows for assigning variables using the **SetVar** function. To substitute values of variables use **#varname#**.

.. code:: js

     SetVar(name, My Name)
     Span(Your name: #name#)
     
To substitute the language resources of the ecosystem, you can use the **$langres$**, where *langres* is the name of the language source.
.. code:: js

     Span($yourname$: #name#)
     
********************************************************************************
Return Value
********************************************************************************

The resulting JSON tree consists of **Node** objects with the following parameters:

* *tag* String – a name of an HTML element or special object.
* *attr* Object – an object consisting of key-value pairs of transferred attributes. As a rule, these include all parameters with names in lower case. Example: **class, value, id**.
* *text* String – plain text. In this case, *tag* equals **text**. 
* *children* Array – an array of nested *Node* objects. This includes all elements described in the **Body** parameter.     

********************************************************************************
Functions
********************************************************************************

Address (Wallet)
==========================
This function returns the wallet address in the 1234-5678-...-7990 format given the numerical value of the address; if the address is not specified, the address of the current user will be taken as the argument. 

.. code:: js

      Span(Your wallet: Address(#wallet#))
      
And (Parameters)
==========================
This function returns the result of execution of the **and** logical operation with all parameters listed in parentheses and separated by commas. The parameter value will be **false** if it equals an empty string (""), zero or *false*. In all other cases the parameter value is **true**. The function returns 1 if true or 0 in all other cases. The element named **and** is created only when a tree for editing is requested. 

.. code:: js

      If(And(#myval1#,#myval2#), Span(OK))
      
      
Button(Body, Page, Class, Contract, Params, PageParams) [.Alert(Text,ConfirmButton,CancelButton,Icon)] [.Style(Style)]
==========================
Creates a **button** HTML element. This element creates a button, which sends a specified contract for execution.

* *Body* - child text or elements.
* *Page* - name of the page to redirect to.
* *Class* - classes for the button.
* *Contract* - name of the contract to execute.
* *Params* - list of values to pass to the contract. By default, values of contract parameters (data section) are obtained from HTML elements (for example, input fields) with similarly-named identifiers (id). If the element identifiers differ from the names of contract parameters, then the assignment in the *contractField1=idname1, contractField2=idname2* format should be used. This parameter is returned to *attr* as an object *{field1: idname1, field2: idname2}*.
**NOTE** In cases where Inputs are not specified, the implementation on the front end can take call controls from the form where the button is located, or independently request a list of parameters from API and take *input* values with same identifiers.
* *PageParams* - parameters for redirection to the page.

**Alert** - displays a message.

* *Text* - message text.
* *ConfirmButton* - confirm button caption.
* *CancelButton* - cancel button caption.
* *Icon* - icon.

**Style** - serves for specifying css styles.

* *Style* - css styles.

.. code:: js

      Button(Submit, default_page, mybtn_class).Alert(Alert message)
      Button(Contract: MyContract, Body:My Contract, Class: myclass, Params:"Name=myid,Id=i10,Value")

Data(Source,Columns,Data) [.Custom(Column,Body)]
==========================
Creates a **data** element and returns data from a database table. Returned are two arrays - *columns* with column names and *data* with entries.
 
* *Source* - data source name. You can specify any name, which will have to be included in other commands later on (ex. *Table*) as a data source.
* *Columns* - list of columns.
* *Data* - one data entry per line, divided into columns by commas. Data should be in the same order as set in *Columns*. Entry values can be embraced in double quotes. If you need to use quote marks in the text, use double quotes.
 
* **Custom** - allows for assigning calculated columns for data. For example, you can specify a template for buttons and additional page layout elements. Several calculated columns can be assigned. As a rule, these fields are assigned for output to *Table* and other commands that use received data.
 
  * *Column* - column name. A unique name should be assigned.
  * *Body* - a code fragment. You can obtain values from other columns in this entry using **#columnname#** and use them in this code fragment.

.. code:: js

    Data(mysrc,"id,name"){
	"1",John Silver
	2,"Mark, Smith"
	3,"Unknown ""Person"""
     }

DBFind(Name, Source) [.Columns(columns)] [.Where(conditions)] [.WhereId(id)] [.Order(name)] [.Limit(limit)] [.Offset(offset)] [.Ecosystem(id)] [.Custom(Column,Body)][.Vars(Prefix)]
==========================
+Создает элемент **dbfind** и возвращает данные из таблицы базы данных. В *attr* возвращаются три массива - *columns* c именами колонок, *types*, где для обычной колонки указан *text*, а для Custom колонок указан тип *tags* и массив *data* с записями. Последовательность в именах колонок соответствует последовательности значений в записях в *data*.
 
* *Name* - table name.
* *Source* - data source name. You can specify any name, which will have to be included in other commands later on (ex. *Table*) as a data source.
 
* **Columns** - list of columns to be returned. If not specified, all columns will be returned.
* **Where** - search condition. For example, *.Where(name = '#myval#')*
* **WhereId** - search by ID. For example, *.WhereId(1)*
* **Order** - sort by this field.
* **Limit** - number of returned rows. Default value = 25, maximum value = 250.
* **Offset** - offset of returned rows.
* **Ecosystem** - ecosystem ID. By default, data is taken from the specified table in the current ecosystem.
* **Custom** - allows for assigning calculated columns for data. For example, you can specify a template for buttons and additional page layout elements. You can assign any number of calculated columns. As a rule, these fields are assigned for output to *Table* and other commands that use received data.
 
  * *Column* - column name. A unique name should be assigned.
  * *Body* - a code fragment. You can obtain values from other columns in this entry using **#columnname#** and use them in this code fragment.
  
  * **Vars** - the function generates a set of variables with values from the database table, obtained from this query. When specifying this function, the *Limit* parameter automatically becomes equal to 1 and only one record is returned.

* *Prefix* - префикс, используемый для образования имен переменных, в которые записываются значения полученной записи: переменные имеют вид *#prefix_id#, #prefix_name#*, где после знака подчеркивания указывается имя колонки таблицы.

.. code:: js

    DBFind(parameters,myparam)
    DBFind(parameters,myparam).Columns(name,value).Where(name='money')
    DBFind(parameters,myparam).Custom(myid){Strong(#id#)}.Custom(myname){
       Strong(Em(#name#))Div(myclass, #company#)
    }
    
Div(Class, Body) [.Style(Style)]
==========================
Creates a **div** HTML element.

* *Class* - classes for this *div*.
* *Body* - child elements.

**Style** - serves for specifying css styles.

* *Style* - css styles.

.. code:: js

      Div(class1 class2, This is a paragraph.)
      
EcosysParam(Name, Index, Source) 
==============================
Функция выводит значение параметра из таблицы parameters текущей экосистемы. Если есть языковый ресурс c полученным именем, то подставится его значение.
 
* *Name* - имя значения;
* *Index* - вы можете указать порядковый номер значения c 1, если параметр является список с элементами раззделенными запятыми. например, *gender = male,female*, тогда EcosysParam(gender, 2) возвратит *female*.  
* *Source* - вы можете получить значения параметра разделенными запятыми в виде объекта *data*. В дальнейшем этот список можно указывать в качестве источника данных как для *Table*, так и для *Select*. Если вы указывайте этот параметр, то сама команда не будет возвращать значение, а возвратит список.

.. code:: js

     Address(EcosysParam(founder_account))
     EcosysParam(gender, Source: mygender)

Em(Body, Class)
==========================
Creates an **em** HTML element.

* *Body* - child text or elements.
* *Class* - classes for this *em*.

.. code:: js

      This is an Em(important news).

Form(Class, Body) [.Style(Style)]
==========================
Creates a **form** HTML element.

* *Class* - classes for this *form*.
* *Body* - child elements.

**Style** - specifies css styles.

* *Style* - css styles.

.. code:: js

      Form(class1 class2, Input(myid))
      
GetVar(Name)
==========================
This function returns the value of the current variable if it exists, or returns an empty string if a variable with this name is not defined. An element with **getvar** name is created only when a tree for editing is requested. The difference between *GetVar(varname)* and *#varname#* is that in case *varname* does not exist, *GetVar* will return an empty string, whereas *#varname#* will be interpreted as a string value.

* *Name* - variable name.

.. code:: js

     If(GetVar(name)){#name#}.Else{Name is unknown}
      
If(Condition){ Body } [.ElseIf(Condition){ Body }] [.Else{ Body }]
==========================
Conditional statement. Returned are child elements of the first *If* or *ElseIf* with fulfilled *Condition*. Otherwise, returned are child elements of *Else*, if it exists.

* *Condition* - a condition is considered non-fulfilled if it equals an *empty string*, *0* or *false*. In other cases the condition is considered true.
* *Body* - child elements.

.. code:: js

      If(#value#){
         Span(Value)
      }.ElseIf(#value2#){Span(Value 2)
      }.ElseIf(#value3#){Span(Value 3)}.Else{
         Span(Nothing)
      }

Image(Src,Alt,Class) [.Style(Style)]
==============================
Creates an **image** HTML element.
 
* *Src* - image source, file or *data:...*;
* *Alt* - alternative text for the image;
* *Сlass* - list of classes.

.. code:: js

    Image(\images\myphoto.jpg)

ImageInput(Name, Width, Ratio) 
==============================
Создает элемент **imageinput** для загрузки картинок. По желанию можно указать ширину вырезаемой картинки и высоту или отношение сторон в виде *1/2*, *2/1*, *3/4* и т.п. в третьем параметре. По умолчанию берется ширина в 100 пикселей и отношение сторон *1/1*.

* *Name* - имя элемента;
* *Width* - ширина вырезаемого изображения;
* *Ratio* - отношение сторон (ширины к высоте) или высота картинки. Если будет указана высота, то отношение сторон рассчитается автоматически.

.. code:: js

   ImageInput(avatar, 100, 2/1)
   
Include(Name)
==========================
This command inserts a template with name *Name* from table *blocks*. On insertion, the template is parsed and single blocks are inserted.

* *Name* - name of the inserted template from the *blocks* table.

.. code:: js

      Div(myclass, Include(mywidget))

Input(Name,Class,Placeholder,Type,Value) [.Validate(validation parameters)] [.Style(Style)]
==========================
Creates an **input** HTML element.

* *Name* - element name.
* *Class* - classes for the *input*.
* *Placeholder* - *placeholder* for the *input*.
* *Type* - *input* type.
* *Value* - element value.

**Validate** - validation parameters.

**Style** - serves for specifying css styles.

* *Style* - css styles.

.. code:: js

      Input(Name: name, Type: text, Placeholder: Enter your name)
      Input(Name: num, Type: text).Validate(minLength: 6, maxLength: 20)

InputErr(Name,validation errors)]
==========================
Creates an **inputerr** element with validation error texts.

* *Name* - name of the corresponding **Input** element.

.. code:: js

      InputErr(Name: name, 
          minLength: Value is too short, 
          maxLength: The length of the value must be less than 20 characters)
          
Label(Body, Class, For) [.Style(Style)]
==========================
Creates a **label** HTML element.

* *Body* - child text or elements.
* *Class* - classes for this *label*.
* *For* - this label's *for* value.

**Style** - serves for specifying css styles.

* *Style* - css styles.

.. code:: js

      Label(The first item).
      
LangRes(Name, Lang)
==========================
Returns a specified language resource. In case of request to a tree for editing it returns the **langres** element.

* *Name* - name of language resource.
* *Lang* - by default, returned is the language defined in request to *Accept-Language*. You can specify your own two-character language identifier.

.. code:: js

      LangRes(name)
      LangRes(myres, fr)

LinkPage(Body, Page, Class, PageParams) [.Style(Style)]
==========================
Creates a **linkpage** element – a link to a page.
 
* *Body* - child text or elements.
* *Page* - page to redirect to.
* *Class* - classes for this button.
* *PageParams* - redirection parameters.

**Style** - specifies css styles.

* *Style* - css styles.

.. code:: js

      LinkPage(My Page, default_page, mybtn_class)

MenuGroup(Title, Body, Icon) 
==============================
This function forms a nested submenu in a menu and returns the **menugroup** element. 

* *Title* - menu item name.
* *Body* - child elements in submenu;
* *Icon* - icon.

.. code:: js

      MenuGroup(My Menu){
          MenuItem(Interface, sys-interface)
          MenuItem(Dahsboard, dashboard_default)
      }

MenuItem(Title, Page, Params, Icon) 
==============================
Creates a menu item and returns the **menuitem** element. 

* *Title* - menu item name;
* *Page* - page to redirect to;
* *Params* - parameters, passed to the page in the *var:value* format, separated by commas.
* *Icon* - icon.

.. code:: js

       MenuItem(Interface, interface)

Now(Format, Interval) 
==============================
Функция возвращает текущее время в указанном формате, по умолчанию выводится  в UNIX-формате (число секунд с 1970 года). Если в качестве формата указано *datetime*, то дата и время выводится в виде YYYY-MM-DD HH:MI:SS. Во втором параметре можно указать интервал, например, *+5 days*.

* *Format* - формат вывода с комбинацией YYYY, MM, DD, HH, MI, SS или *datetime*;
* *Interval* - дополнтельный сдвиг времени назад или вперед;

.. code:: js

       Now()
       Now(DD.MM.YYYY HH:MM)
       Now(datetime,-3 hours)

Or(parameters)
==========================
This function returns a result of the **IF** logical operation with all parameters specified in parentheses and separated by commas. The parameter value is considered **false** if it equals an empty string (""), 0 or *false*. In all other cases the parameter value is considered **true**. The function returns 1 for true or 0 in all other cases. Element named **or** is created only when the tree for editing is requested. 

.. code:: js

      If(Or(#myval1#,#myval2#), Span(OK))

P(Body, Class) [.Style(Style)]
==========================
Creates a **p** HTML element.

* *Body* - child text or elements.
* *Class* - classes for this *p*.

**Style** - specifies css styles.

* *Style* - css styles.

.. code:: js

      P(This is the first line.
        This is the second line.)
        
Select(Name, Source, NameColumn, ValueColumn, Value, Class) [.Validate(validation parameters)] [.Style(Style)]
==========================
Creates a **select** HTML element.

* *Name* - имя элемента.
* *Source* - имя источника данных. Например, из команды *DBFind* или *Data*.
* *NameColumn* - Имя колонки, из которой будeт браться текст для элементов.
* *ValueColumn* - Имя колонки, из которой будут браться значения для элементов. В этом параметре нельзя указывать имена колонок созданных через Custom.
* *Value* - Значение по умолчанию.
* *Class* - Классы для элемента.

**Validate** - параметры валидации.

**Style** - служит для указания css стилей.

* *Style* - css стили.

.. code:: js

      DBFind(mytable, mysrc)
      Select(mysrc, name)

SetVar(Name, Value)
==========================
Assigns a *Value* to a *Name* variable. Element named **setvar** is created only when a tree for editing is requested.

* *Name* - name of the variable.
* *Value* - value of the variable, which can contain a reference to another variable.

.. code:: js

     SetVar(name, John Smith).(out, I am #name#)
     Span(#out#)
     
Span(Body, Class) [.Style(Style)]
==========================
Creates a **span** HTML element.

* *Body* - child class or elements.
* *Class* - classes for this *span*.

**Style** - specifies css styles.

* *Style* - css styles.

.. code:: js

      This is Span(the first item, myclass1).
      
Strong(Body, Class)
==========================
Creates a **strong** HTML element.

* *Body* - child text or elements.
* *Class* - classes for this *strong*.

.. code:: js

      This is Strong(the first item, myclass1).
      
Table(Source, Columns) [.Style(Style)]
==========================
Создает HTML элемент **table**.

* *Source* - data source name as specified, for example, in the *DBFind* command.
* *Columns* - Headers and corresponding column names, as follows: **Title1=column1,Title2=column2**.

**Style** - specifies css styles.

* *Style* - css styles.

.. code:: js

      DBFind(mytable, mysrc)
      Table(mysrc,"ID=id,Name=name")
      
