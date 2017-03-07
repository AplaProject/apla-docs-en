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
*  *format - format template: YY short year, YYYY full year, MM - month, DD – day, for example, DD.MM.YY. If the format is not specified, then the *dateformat* parameter value, indicated in the languages table will be used, and if it is absent, then YYYY-MM-DD.

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
Функция создает контейнер <a href="*href*">*text*</a> с указанными классами (*class*).

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
Функция создает контейнер <span>*text*</span> с указанными классами  (*class*).

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
The function creates a <tagname>*text*</tagname> container with the specified classes; the h1-h6 tags are supported.

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
Условная конструкция 
********************************************************************************

If(condition) … Else … ElseIf … IfEnd 
==============================
Условная конструкция позволяет выводить разные фрагменты  шаблона страницы в зависимости от истинности или ложности условного выражения. Конструкции **If** могут быть вложенными, например,

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
Вывод элементов форм
********************************************************************************

Form(class) … FormEnd 
==============================
Функция обрамляет часть шаблона страницы контейнером <form>…</form>  с указанными классами (*class*).

Input(idname,[class],[placeholder],[type],[value]) 
==============================
Функция создает поле ввода формы;

* *idname* - имя идентификатора поля;
* *class*  - список классов;
* *placeholder* - текст подсказка;
* *type* - тип поля, по умолчанию *text*;
* *value* - значение по умолчанию.

Textarea(idname,[class],[value]) 
==============================
Функция выводит поле формы типа *textarea*.

* *idname* - имя идентификатора поля;
* *class*  - список классов;
* *value* - значение по умолчанию.


InputAddress(idname,[class],[value] ) 
==============================
Функция создает поле формы для ввода  адреса кошелька, при вводе адреса предполагаемые варианты показываются в выпадающем списке. 

* *idname* - имя идентификатора поля;
* *class*  - список классов;
* *value* - значение по умолчанию.

InputDate(idname,[class],[value] ) 
==============================
Функция создает поле формы для ввода даты и времени. 

* *idname* - имя идентификатора поля;
* *class*  - список классов;
* *value* - значение по умолчанию.

InputMoney(idname,[class],[value]) 
==============================
Функция создает поле формы для ввода денежных значений. 

* *idname* - имя идентификатора поля;
* *class*  - список классов;
* *value* - значение по умолчанию.

Select(idname, list, [class], [value]) 
==============================
Функция создает разворачивающийся список  <select>. 

* *idname* - идентификатор. 
* *list* - передает список значений; 
* *value* - значение списка, выбранное по умолчанию;
* *class*  - список классов.

Существует два варианта определения списка *list*: 

1. перечисление  имен списка через запятую, при этом значение value в <option … >  будет равно порядковому номеру имени начиная с 1;

2. получение значений из таблиц базы данных в формате **tablename.column.idname**, где tablename - имя таблицы, column - имя колонки, значения которой выводятся как имена списка, idname - имя колонки, значения которой используются в качестве value в <option … >. Если *idname* не указан, то по умолчанию используется колонка *id*; количество записей в таблице не может быть больше 50.
Если в таблице languages имеются языковые ресурсы со значением имен списка, то они будут автоматически переводиться.

TextHidden(idname,....) 
==============================
Функция создает множество скрытых полей textarea; в качестве  идентификаторов (id)  устанавливаются перечисленные через запятую имена; значения полей берутся из одноименных переменных. Например, если есть переменная #test# = "Строка", то TextHidden(test) создаст textarea с id="test" и значением "Строка".

Source(idname,[value]) 
==============================
Функция выводит поле ввода текста с подстветкой операторов, ключевых слов и т.д. Используется, например, для редактирования контрактов.

* *idname* - идентификатор; 
* *value* - значение по умолчанию.

********************************************************************************
Получение значений из базы данных
********************************************************************************

ValueById(table,idval,columns,[aliases]) 
==============================
Функция получает значения из записи таблицы базы данных по значению id строки.

* *table* - имя таблицы; 
* *idval* - значение id получаемой записи;
* *columns*  - имена колонок, перечисленные через запятую; по умолчанию будут созданы переменные с именами колонок, которым и будут переданы полученные значения; 
* *aliases*  - имена переменных, отличные от имен колонок, перечисленные через запятую в том же порядке, что и имена колонок.
Например, * ValueById(#state_id#_citizens, #citizen#, "name,avatar", "FirstName,Image") *

GetList(name, table, colnames, [where], [order], [limit]) 
==============================
Функция получает записи из таблицы table. 

*  *name* - имя, по которому извлекается конкретная запись из полученного списка с помощью функций **ListVal** или **ForList**;
*  *colnames* - список получаемых колонок, перечисленных через запятую; первым, необходимо указывать колонку с индексом и по этому значению будет реализован доступ к значениям в **ListVal** или **ForList**; 
*  *where*, *order *, *limit * - условие, сортировка и кол-во получаемых строк.

ListVal(name, index, column]) 
==============================
Функция возвращает значение из списка полученного функцией **GetList**; 

* *name* - в качестве значения параметра  следует использовать имя, которое было указано в функции *GetList*;
* *index* - значение идентификатора поиска по первой колонке, указанной в *GetList*; 
* *column* - имя колонки с возвращаемым значением.

ForList(name) … FormListEnd 
==============================
Функция  выводит полный список записей, полученных с помощью функции **GetList**; в качестве значения параметра *name* следует использовать имя, которое было указано в функции *GetList*. Конец шаблона вывода одной записи фиксируется закрывающей функции **FormListEnd**. Значения колонок записи содержат переменные вида #name_column#, в которых после знака подчеркивания указывается имя колонки таблицы; доступна переменная #index#, которая содержит порядковый номер записи, начиная с 1.

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
Функция возвращает  значение из таблицы базы данных по условию.

* *colname* - имя возвращаемой колонки;
* *table* полной имя таблицы (#state#_mytable); 
* *where*  условие,
* *value* - значение условия, если параметр *value* не указан, то тогда параметр *where* должен содержать полный запрос.

GetRow(prefix, table, colname, [value]) 
==============================
Функция формирует множество переменных со значениями из  записи таблицы базы данных, полученной поиском по указанному полю и значению или по запросу.

* *prefix* - префикс, используемый для образования имен переменных, в которые записываются значения полученной записи: переменные имеют вид *#prefix_id#, #prefix_name#*, где после знака подчеркивания указывается имя колонки таблицы.
* *table* - полной имя таблицы (#state#_mytable); 
* *colname* - имя колонки, по которой ищется запись;
* *value* - значение, по которому ищется запись, если параметр *value* не указан, тогда параметре *colname * должен содержать полный запрос where к таблице.

StateVal(name, [index]) 
==============================
Функция выводит значение параметра из таблицы state_parameters.

* *name* - имя значения;
* *index* - порядковый номер значения, если их список приведен через запятую, например, *gender | male,female*, тогда StateVal(gender, 2) возвратит *female*  
Если есть языковый ресурс полученным именем, то подставится его значение.

Table 
==============================
Функция создает таблицу со значениями из базы данных. Функция имеет именованные параметры, которые выводятся в фигурных кнопках: 

* *Table* - имя таблицы;
* *Order* - имя колонки для сортировки строк таблицы, необязательный параметр;
* *Where* - условие выборки, необязательный параметр;
* *Columns* - массив отображаемых колонок, состоящий из заголовка и значений [[ColumnTitle, value],...]; соответствующие строке значения колонки из таблицы базы возвращаются переменной с именем колонки (#column_name#).

.. code:: js
     Table{
         Table:  citizens
         Order: id
         Columns: [[Avatar, Image(#avatar#)],  [ID, Address(#id#)],  [Name, #name#]]
     }

********************************************************************************
Вызов контрактов
********************************************************************************

BtnContract(contract, name, message, params, [class], [onsuccess], [pageparams]) 
==============================
Функция создает кнопку, при клике на которой открывается модальное окно с предложением отказаться  или подтвердить вызов контракта. 

* *contract* - имя контракта;
* *name* - название кнопки;
* *message* - текст для модального окна;
* *params* - параметры, передаваемые контракту;
* *class*  - список классов кнопки;
* *onsuccess* - имя страницы, на которую следует сделать переход в случае успешного выполнения контракта;
* *pageparams* - параметры, передаваемые странице в формате *var:value* через запятую.

Например, *BtnContract(DelContract, Delete, Delete Item?, "IdItem:id_item",'btn btn-default')*

TxButton 
==============================
Функция создает кнопку при клике на которой запускается выполнение контракта. Функция имеет именованные параметры, которые выводятся в фигурных кнопках:

* *Contract* - имя контракта;
* *Name* - название кнопки, по умолчанию **Send**;
* *Class* - список классов для контейнера <div> с кнопкой;
* *ClassBtn* - список классов для кнопки;
* *Inputs* - список передаваемых в контракт значений. По умолчанию, значения параметров контракта (секция *data*) берутся их HTML элементов (скажем, полей формы) с одноименными идентификаторами (*id*). Если идентификаторами элементов отличаются от названий параметров контракта, то используется присваивание в формате *Inputs: "contractField1=idname1, contractField2=idname2" Присваивать можно и значения переменных в формате *Inputs: "contractField1#=var1, contractField2=var2" (будут переданы значения переменных #var1# и #var2#);
* *OnSuccess* - имя страницы, на которую будет осуществлен переход в случае успешного выполнения контракта, и через запятую передаваемые на страницу параметры в формате *var:value*, например,  * OnSuccess: "CompanyDetails, CompanyId:#CompanyId#" *;
* *Silent* - при значении 1 вывод сообщения  об успешной выполнении контракта;
* *AutoClose* - при значении 1 автоматическое закрытие сообщения об успешном выполнении контракта.

Например,

.. code:: js

      TxButton {
          Contract: MyContract,
          Inputs: 'Name=myname, Request #= myreq',
         OnSuccess: "MyPage, RequestId:# myreq#"
      }


TxForm 
==============================
Функция создает форму для вода данных контракта. Функция имеет именованные параметры, которые выводятся в фигурных кнопках:

* *Contract* - имя контракта;
* *OnSuccess* - имя страницы, на которую будет осуществлен переход в случае успешного выполнения контракта, и через запятую передаваемые на страницу параметры в формате *var:value*, например,  * OnSuccess: "CompanyDetails, CompanyId:#CompanyId#" *;
* *Silent* - при значении 1 происходит вывод сообщения  об успешной выполнении контракта;
* *AutoClose* - при значении 1 происходит автоматическое закрытие сообщения об успешном выполнении контракта.

.. code:: js

      TxForm {
          Contract: MyContract,
          OnSuccess: 'mypage'
      }

********************************************************************************
Элементы навигации
********************************************************************************

Navigation( params, …) 
==============================
Функция выводит панель с «хлебными крошками» и ссылкой на редактирование текущей страницы **Edit**. Например, Navigation( LiTemplate(dashboard_default, citizen),goverment).

LinkPage(page,text,[params]) 
==============================
Функция создает ссылку  на страницу. Если *name* не указан, то текст ссылки будет такой же как *page*. C помощью этой же функции можно создавать ссылки на предопределенные страницы. В этом случае добавьте префикс **sys-** перед именем страницы. Например, *LinkPage(sys-interface, Interface)*. Кроме этого, эту функцию можно использовать для создания ссылок на приложения. Для этого перед именем страницы-приложения необходимо указывать **app-**.

* *page* - имя страницы;
* *text*  - текст ссылки;
* *params* - параметры передаваемые странице в формате *var:value* через запятую.


LiTemplate(page, [text], [params], [class]) 
==============================
Функция создает контейнер  <li>*text*</li> содержащий ссылку  на страницу. 

* *page* - имя страницы;
* *text*  - текст ссылки;
* *params* - параметры передаваемые странице в формате *var:value* через запятую;
* *class*  - список классов.

.. code:: js

      LiTemplate(mypage, Home page, global:1)

BtnPage(page, name,[params],[class], [anchor]) 
==============================
Функция создает кнопку, при клике на которой происходит переход на указанную страницу. Если параметр с классом не указан, то у кнопки будут классы *btn btn-primary*. C помощью этой же функции можно создавать ссылки на системные страницы. В этом случае добавьте префикс sys- перед именем страницы. Например, *BtnPage(sys-interface, Interface)*.

* *page* - имя страницы перехода; 
* *name* - название  кнопки;
* *params* - параметры, передаваемые странице;
* *class*  - список классов кнопки;
* *anchor* - якорь (id элемента страницы) для скроллинга страницы в указанное положение.

BtnEdit( page, icon, [params] ) 
==============================
Функция создает кнопку с указанной иконкой. Может использоваться в таблицах для перехода на страницы редактирования. Для перехода на системные страницы или страницы приложений необходимо добавлять соответствено префиксы **sys-** и **app-**. Например,
*BtnEdit(sys-editPage, cog, "name: #name#, global: #global#")*.

Back(page, [params]) 
==============================
Функция вписывает вызов указанной страницы в историю вызовов. 

* *page* - имя страницы;
* *params* - параметры вызова страницы из истории в формате *var:value* через запятую.

********************************************************************************
Оформление шаблона страницы
********************************************************************************

PageTitle(header) … PageEnd() 
==============================
Функция фиксирует тело страницы и создает панель с заголовком, указанным в параметре *header*. 

Title(text) 
==============================
Функция создает заголовок с классом *content-heading*.

FullScreen(state) 
==============================
Функция переводит ширину рабочей области страницы на всю ширину окна когда параметр *state* равен 1, сужает рабочую область  при *state* равном  0.

WhiteMobileBg(state) 
==============================
Функция является аналогом функции **FullScreen** для мобильных устройств; переводит ширину рабочей области страницы на всю ширину окна когда параметр *state* равен 1, сужает рабочую область  при *state* равном  0.

********************************************************************************
Организация многоуровневого меню
********************************************************************************

MenuItem(title, page, [params], [icon]) 
==============================
Функция создает пункт меню. 

* *title* - имя пункта меню, если в таблице languages имеется языковой ресурс со значением, указанным в *title*, то текст будет автоматически переводиться;
* *page* - имя страницы перехода. Для перехода на системные страницы необходмо указывать префиксы **sys-** или **app-**;
* *params* - параметры, передаваемые странице в формате *var:value* через запятую.
* *icon* - иконка.

MenuGroup(title,[idname],[icon]) … MenuEnd: 
==============================
Функция формирует в меню вложенное подменю. 

* *title* - имя пункта меню, если в таблице languages имеется языковой ресурс со значением, указанным в *title*, то текст будет автоматически переводиться;
* *idname* - идентификатор подменю;
* *icon* - иконка.

.. code:: js

      MenuGroup(My Menu,mycitizen)
            MenuItem(Interface, sys-interface)
            MenuItem(Dahsboard, dashboard_default)
      MenuEnd:

MenuBack(title, [page]) 
==============================
Функция заменяет ссылку перехода к родительскому меню (верхний пункт меню).

* *title* - имя пункта меню, если в таблице languages имеется языковой ресурс со значением, указанным в *title*, то текст будет автоматически переводиться;
* *page* - имя страницы перехода.


MenuPage(page) 
==============================
Функция устанавливает в качестве родительского пункта меню указанную  в параметре *page* страницу.

********************************************************************************
Представление данных
********************************************************************************

Ring(count,[title],[size]) 
==============================
Функция выводит круг со значением параметра *count* посередине. 

* *title* - заголовок;
* *size* - размер значения.

WiAccount(address) 
==============================
Функция выводит в специальном оформлении номер аккаунта (адрес кошелька), переданном в параметре address.

WiBalance(value, money) 
==============================
Функция выводит в специальном оформлении значение *value* в денежном формате и добавляет обозначение валюты указанной в параметре *money*.

WiCitizen(name, address, [image], [flag]) 
==============================
Функция выводит в специальном оформлении информацию о гражданине. 

* *name* - имя;
* *address*  - адрес кошелька, приведенный к виду 1234-...-5678;
* *name* - изображение;
* *name* - флаг страны. 

Map(coords) 
==============================
Функция выводит на страницу контейнер google maps с координатами указанными в параметре *coords*. Высоты контейнера берется из значения предварительно определенной переменной #hmap# (по умолчанию 100 пикселей), ширина растягивается на максимально возможную величину.

MapPoint(coords) 
==============================
Функция выводит на страницу контейнер google maps с маркером по координатам указанным в параметре *coords*. Высоты контейнера берется из значения предварительно определенной переменной #hmap# (по умолчанию 100 пикселей), ширина растягивается на максимально возможную величину.

ChartPie 
==============================
Функция выводит круговую диаграмму. Функция имеет именованные параметрами,  которые выводятся в фигурных кнопках: 

* *Data* - данные отражаемые диаграммой в виде списка [[value,color,label],....]; каждый элемент списка должен содержать три параметра: значение, цвет rrggbb и подпись; при наличии этого списка другие параметры будут игнорироваться;
* *Columns* - список цветов rrggbb через запятую;
* *Table* - имя таблицы, откуда будут браться данные;
* *FieldValue* - имя столбца со значениями;
* *FieldLabel* - имя столбца с подписями;
* *Order* - имя колонки для сортировки строк таблицы, необязательный параметр;
* *Where* - условие выборки, необязательный параметр;
* *Limit* - смещение и количество выбираемых записей, необязательный параметр.

ChartBar 
==============================
Функция выводит диаграмму в виде столбцов. Все параметры, за исключением *Data*, идентичны функции **ChartPie**.

********************************************************************************
Вывод языковых ресурсов
********************************************************************************

LangJS(resname) 
==============================
Функция создает контейнер <span>*resname*</span>  со значением языкового ресурса. Используется для автоматической подстановки языковых ресурсов в браузере. (Речь идет о ресурсах, которые описаны в static/js/lang/*.js.)

LangRes(resname) 
==============================
Функция возвращает из таблицы languages языковой ресурс с указанным именем.

********************************************************************************
Служебные функции
********************************************************************************

BlockInfo(blockid) 
==============================
Функция выводит ссылку с номером блока (blockid), при клике по которой будет открываться окно с информацией о блоке.

TxId(txname) 
==============================
Функция возвращается идентификатор указанной транзакции. Например,

.. code:: js

      SetVar(
      type_new_page_id = TxId(NewPage),
      type_new_contract_id = TxId(NewContract)
      )

Json(data) 
==============================
Функция создает тэг script с переменной jdata с присвоением ей указанных в параметре data  данными в формате json.
Например,

.. code:: js

      Json(`param1: 1, param2: "string"`) 
      
получим 

.. code:: js

      var jdata = { param1: 1, param2: "string"}
