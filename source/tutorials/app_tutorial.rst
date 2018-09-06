.. _docker: https://docs.docker.com/engine/docker-overview


.. -- Conditionals Genesis / Apla -------------------------------------------------

.. quick-start project link
.. _quick-start: https://github.com/GenesisKernel/quick-start
.. .. _quick-start: https://github.com/AplaProject/quick-start

.. _quick-start-readme: https://github.com/GenesisKernel/quick-start/blob/master/README.md
.. .. _quick-start-readme: https://github.com/AplaProject/quick-start/blob/master/README.md

.. password for quick-start
.. |pass_quickstart| replace:: ``genesis``
.. .. |pass_quickstart| replace:: ``default``


Application development tutorial
================================

This tutorial explains how to write a simple application for |platform|.


The goal
--------

The goal of this tutorial is to create an application for |platform|. 

The application starts simple and grows in complexity as the tutorial progresses.

The final version of the app stores simple messages (strings) on the blockchain with a timestamp and a message sender's account identifier. A user can access the list of messages and add new messages from the app's page. The app's page can be accessed from the ecosystem menu.

.. todo::
    
    This tutorial must explain basic access permissions for apps and for tables (what is MainCondition and how to change it). Also, basic styles and layouts must be explained.


Part 1: The environment
-----------------------


quick-start
^^^^^^^^^^^

This tutorial uses |platform| `quick start <quick-start>`_. The quick start provides `docker`_ containers for five network nodes. Each node has its own backend, database, and client. The manage.sh script can be used to control the nodes.

For installation instructions, see the `quick-start readme <quick-start-readme>`_ on GitHub.


.. _founder-login:

Molis
^^^^^

For this tutorial, you'll be writing contract code, page template code, and performing all other actions in the Molis client. Molis also provides a way to retrieve, save, and execute contract code on the blockchain, manage data structures (tables), assign access rights, and create applications.

The "root" account for |platform| ecosystems is called *founder's account*. By default, this account has access to all operations. You must use this account to introduce major changes to the ecosystem, such as creating new apps and tables.

To login to Molis with founder's account: 

    #. Make sure that quick-start is running. See `quick-start documentation <quick-start-readme>`_ for more information.

    #. Run ``$ sudo ./manage.sh start-clients``

        This command starts Molis clients for all nodes. 

    #. One of the started clients is for the founder's account. This account lets you select roles after logging in. Choose *Admin*.

        Password for all accounts is |pass_quickstart|.



Part 2: What is an app
----------------------


App components
^^^^^^^^^^^^^^

|platform| apps are a combination of contracts, tables, and user interfaces.

* Contracts

    Contracts are like functions. They take input parameters, validate these parameters, and perform defined actions.

    Contracts are written in :doc:`Simvolio language</topics/script>` and are stored on the blockchain.


* Tables

    Tables hold information that is used by contracts and user interfaces.

    Just like with regular database tables, you can query, update, and insert. These actions are performed with three :doc:`Simvolio language</topics/script>` functions: :func:`DBFind`, :func:`DBInsert`, and :func:`DBUpdate`.


* User interfaces

    User interfaces are pages and menus that will be displayed to users by Molis. 

    The interfaces are written in :doc:`Protypo language </topics/templates2>`. Molis provides a visual editor for constructing interfaces.


Modularity
^^^^^^^^^^

The architecture of |platform| apps is designed to be modular. Contracts, tables, and interfaces are stored in the blockchain and can be used by many different apps. 

For example, if you change the contract code, you must make a transaction that introduces the contract code change. After the transaction is validated and included in the blockchain, the new contract code becomes available to all nodes in the network. Same principle applies to tables and data stored in them, and to user interfaces.

.. todo::

    Illustration needed.


Resource Access
^^^^^^^^^^^^^^^

An app is a collection of its resources: contracts, pages, and tables. All resources of all  apps within one ecosystem are available to each other. One resource can be used by many apps. Resources do not need to belong to a same app to be accessible.

For example, a dashboard page can use many tables that store information about ecosystem members and business processes; a high-level contract can update several tables that are used by many ecosystem apps.

.. todo::

    Fix this after access rights chapter is written.

Access to resources is managed with access rights, which are implemented with contracts.


Part 3: The contract
--------------------

You now have your blockchain network of five nodes and a basic understanding of what is an app and how apps work. Your first application will start as a simple "Hello, World!" application.


The spec
^^^^^^^^

The application stores a single string on the blockchain. It doesn't have any user interface but uses a table to store the string.


New app
^^^^^^^

|platform| apps are created via transactions. By default, only ecosystem's founder can create an app. You can create new apps from Molis.


To create a new app:

    #. :ref:`Login as the founder <founder-login>`. 

    #. Go to the *Admin* tab.

    #. From the list on the left, select *Application*.

    #. In the *Applications* view, select *Create*.

    #. Specify the name of your app in the *Name* field.

    #. In the *Change conditions* specify ``true``.
        
        The ``true`` value will make it possible for anyone to change the app. 

        Another option is to specify ``ContractConditions("MainCondition")``. This will forbid application changes to anyone except the founder.

        .. todo::
            
            Explain morea bout access rights.

    #. Your app will appear in the list of apps. Click *select* to make it active.

        .. note::
        
            Selecting apps in the *Admin* tab makes it easier to navigate resources related to the selected app. It has no effect on the ecosystem. All ecosystem apps will still be available, no matter which one is selected.


New table
^^^^^^^^^

To store data, the application needs a table. Create this table from Molis.

To create a table: 

    #. On the *Admin tab*, select *Resources* > *Tables*.

        This will display all tables for the selected app. The list will be empty, because your app doesn't have any tables yet.

    #. Click *Create*.

        Molis will display the *Create table* view.
        
    #. Specify a name for your table in the *Name* field.

        This tutorial uses ``apptable`` name for the table.

    #. Add a column. Name it ``message`` and set its type to ``Text``.

        As a result, the table must have two columns: ``id`` (predefined), and ``message``. You'll add more columns later.

        .. image:: /_static/app-tut-table.png
            :scale: 60%

    #. For write permissions, specify ``true`` in every field.

        This will allow anyone to perform inserts and updates on the table, and to add columns.

        As an option, you can restrict writing permissions to the founder account. In this case, specify ``ContractConditions("MainCondition")`` in this parameter.



The contract
^^^^^^^^^^^^


Contract code sections
""""""""""""""""""""""

Every contract has three sections: 

* ``data``

    Declares the input data (names and types of variables).

*   ``conditions``

    Validates the input data.

*   ``action``

    Performs actions defined by the contract logic.


Creating a new contract
"""""""""""""""""""""""

#. On the *Admin tab*, select *Resources* > *Contracts*.

    This will display all contracts for the selected app. The list for your new app will be empty.

#. Click *Create*.
    
    A new contract template will open in the editor.


An empty contract template looks like this: 

.. code-block:: js

    contract ... {
        data {

        }
        conditions {

        }
        action {

        }
    }


Contract name
"""""""""""""

To start, give a name to your contract.

.. code-block:: js

    contract AppContract {


Data section
""""""""""""

Fill the ``data`` section. The app must write strings to the blockchain, so a ``string`` type variable is needed.

In the example below, ``Message`` is the name of the variable, ``string`` is its type.

.. code-block:: js

    data {
        Message string
    }


Condition section
"""""""""""""""""

Fill the ``conditions`` section. The single validation condition is that the specified string must not be empty. 

.. code-block:: js

    conditions {
        // avoid writing empty strings
        if Size($Message) == 0 {
            error "Message is empty"
        }
    }


Action section
""""""""""""""

Fill the ``action`` section. The single action is writing the message to the table.

.. code-block:: js

    action {
        DBInsert("apptable", "message", $Message)
    }


Full contract code
""""""""""""""""""

Below is the full contract code for this part. 

All |platform| contracts are constructed like this and always contain ``data``, ``conditions``, and ``action`` sections.

.. code-block:: js

    contract AppContract {
        data {
            Message string
        }
        conditions {
            // avoid writing empty strings
            if Size($Message) == 0 {
                error "Message is empty"
            }
        }
        action {
            DBInsert("apptable", "message", $Message)
        }
    }


Save & execute
""""""""""""""

The contract is ready for testing: 

    #. In the Editor menu, click *Save*.

        This updates the contract code on the blockchain.

    #. In the Editor menu, click *Execute*.

        This displays the *Execute contract* view.

    #. In the *Execute contract* view, enter the input parameters for the contract.

        The contract has one parameter, ``Message``, so specify ``Message`` in *Key* and ``Hello, World!`` in *Value*.

        .. image:: /_static/app-tut-execute.png
            :scale: 60%            

    #. Click *Exec*.

        The results will be displayed on the right.

If the string was added successfully, the results will contain the block number of the transaction that introduced the change, and the error code.

.. code-block:: js

    {
       "block": "31",
       "error": null
    }


Part 4: The interface
---------------------

After the contract is working, it's time to expand it into something more useful. In this part, you'll be implementing the UI and extra functionality.


The spec
^^^^^^^^

The app stores strings on the blockchain, like entries in a log. Every string has an author and a timestamp. 

A user can view the stored list of strings from the application page, which is a simple table at this point.

The app does not provide a way to add new strings from the UI yet.


New columns
^^^^^^^^^^^

Just like before, edit the table from the *Admin* > *Resources* > *Tables* view.

Add the following columns to the ``apptable`` table: 

* ``author`` of type ``Number`` with *Update* set to ``true``.

    This field will store the identifier of the author's account.

* ``timestamp`` of type ``Date/Time`` with *Update* set to ``true``.
    
    .. todo::

        Explain how Update condition works in this case.


Updated contract
^^^^^^^^^^^^^^^^

Update the contract code to handle author IDs and timestamps. 

Author IDs are identifers of the ecosystem accounts. Timestamps are the date and time of the transaction in the Unix time format.

Both of these values are provided by the :ref:`predefined values <simvolio-predefined-values>`. Since there is no need to input or validate the predefined values, changes are needed only in the action section.


Action section
""""""""""""""

Change the contract so that the author's ID and the timestamp are written to the table when a message is added. The author's ID is defined by ``$key_id``, the timestamp is defined by ``$time``.

.. code-block:: js

    action {
        DBInsert("apptable", "message, author, timestamp", $Message, $key_id, $time)
    }


The page
^^^^^^^^

For this part, the application's interface is a simple page that displays information stored in the table.

Just like all other resources, UI pages can be created in Molis:

#. Navigate to *Admin* > *Resources* > *Pages*.

#. Click *Create*.

    A visual editor will open in the new tab.


Designer's view
"""""""""""""""

The default page is empty. Fortunately, you can use predefined structures to fill the page quickly.

    .. image:: /_static/app-tut-designer.png
        :scale: 60%


Create a basic table with header: 

#. In the view selector on the right, click *Designer*.

    The view will switch to the visual editor.

#. From the menu on the left, select *Table With Header* and drag it to the page.

    A table with several elements will appear.


Developer's view
""""""""""""""""

User interfaces for |platform| are written in :doc:`Protypo</topics/templates2>`. You'll need to write code for the page, so switch to the developer's view.

    .. image:: /_static/app-tut-developer.png
        :scale: 60%

To switch to the developer's view: 

#. In the view selector on the right, click *Developer*.

    The view will switch to the code editor with the page code.


Get data from the table
"""""""""""""""""""""""

At the moment, the page template does nothing. Change the code, so that the page displays data from the ``apptable`` table.

#. To request data from a table, use the :func:`DBFind` function. 

    The function call in the following exaple gets data from the ``apptable`` table, puts it into the ``src_table`` source, and orders it by the timestamp field. The ``src_table`` source is later used as a source of data for the displayed table.

    .. code-block:: js

        DBFind(Name: apptable, Source: src_table).Columns(Columns: "author,timestamp,message").Order(timestamp)


#. To display data from the ``src_table`` source, specify it as a source along with a list of headers in the ``Table`` function.

    .. code-block:: js

        Table(Columns: "AUTHOR=author,TIME=timestamp,MESSAGE=message", Source: src_table)


#. In the view selector on the right, Click *Preview* to check that the data is displayed correctly.


Full page code
""""""""""""""

Below is the full page code for this part. This basic page will be expanded later.

.. code-block:: js

    DBFind(Name: apptable, Source: src_table).Columns(Columns: "author,timestamp,message").Order(timestamp)

    Div(Class: panel panel-primary) {
        Div(Class: panel-heading, Body: Table block)
        Table(Columns: "AUTHOR=author,TIME=timestamp,MESSAGE=message", Source: src_table)
        Div(Class: panel-footer text-right) {
            Button(Class: btn btn-primary, Contract: ContractName, Body: More)
        }
    }


Save the page
"""""""""""""

Click *Save* to save the page: 

#. Specify ``AppPage`` or any other name for a page in the *Name* field.

#. Leave the *Menu* option at ``default_menu``.

#. In *Change Conditions* specify ``true``.

#. Click *Confirm*.


Part 5: The app
---------------

In the previous parts you've created a contract, a table to store data, and a basic UI page to display this data.

In this part, you'll be finalizing the app, so it looks and behaves like an actual application.


The spec
^^^^^^^^

The app stores messages on the blockchain, like entries in a log. Every message has an author and a timestamp. 

A user can view the stored messages by opening the application page from the ecosystem menu. The default view holds 25 mesages and provides a way to browse more.

A user can add new messages from the UI page, one message at a time.


The menu
^^^^^^^^

The menus are always linked to pages. For example, the ecosystem menu that is displayed on the *Home* tab is linked to the ``default_page`` page.

Because the tutorial app is small (just one page), there is no need to create an individual menu for it. A new menu item in the default menu will be enough.

.. note::
    
    You can define what menu is displayed for the page by editing page properties in *Admin* > *Resources* > *Pages*. For example, if your app has several pages, you may want to create a menu to navigate between these pages and assign it to all pages of your app.


Add a menu item
"""""""""""""""

Just like all other resources, menus can be created and edited in Molis:

#. Navigate to *Admin* > *Menu*.

    .. image:: /_static/app-tut-menu-list.png
        :scale: 60%


#. Click the edit button next to the ``default_menu`` entry.

    A visual editor will open in the new tab displaying Protypo template for the default ecosystem menu.

#. Add a new menu item to the end of the template. This menu item will open the app's page. The icon is from the `FontAwesome`_ icon set.

    .. code-block:: js

        MenuItem(Title:Messages, Page:AppPage, Icon:"fa fa-envelope")

#. Click *Save*.


.. _FontAwesome: https://fontawesome.com/icons


Test the new menu item
""""""""""""""""""""""

Check that the new menu item works: 

#. Open the *Home* tab.


#. Click *Refresh* in the menu.

    A new item titled *Messages* will appear.

    .. image:: /_static/app-tut-menu-messages.png
        :scale: 100%


#. Click *Messages*.

    The app's page will open.


Table navigation
^^^^^^^^^^^^^^^^

The default table can show only 25 first entries. Add a simple navigation that will allow users to navigate all table entries.

.. note::

    To have more than 25 messages, you can add extra table entries by executing the contract several times with different values.


Navigation buttons
""""""""""""""""""

The table navigation will use two buttons. Each button will reload the app's page and pass parameters to it.


    * *Previous* button will show previous 25 entries. If there are no additional entries, the button will not be displayed.

    * *Next* button will show next 25 entries. If there are no additional entries, the button will not be displayed.


Variables
"""""""""

This navigation requires two variables to store the table state: 

    * ``#table_offset#``

        This variable stores the current table offset.

        Navigation buttons will pass this as a parameter when reloading the page.

    * ``#record_count#``

        This variable stores the total number of entries in the table.

        This value will be calculated.


Record count
""""""""""""

To calculate ``#record_count#``, modify the existing :func:`DBFind` function call. The variable specified in the ``.Count()`` call will store the record count.

    .. code-block:: default
        
        DBFind(Name: apptable, Source: src_table).Columns(Columns: "author,timestamp,message").Order(timestamp).Count(record_count)


Table offset
""""""""""""

The table offset must be passed to the page when it is opened. If ``#table_offset#`` is not passed, it is assumed to be ``0``.

Add the following code to the top of the page template. This code uses conditionals. :func:`GetVar` function checks if the variable is set. :func:`SetVar` function sets the variable.

    .. code-block:: default

        If(GetVar(table_offset)){
        }.Else{
            SetVar(table_offset, 0)
        }

Modify the :func:`DBFind` function call again. This time it must use the new table offset. 

    .. code-block:: default

        DBFind(Name: apptable, Source: src_table).Columns(Columns: "author,timestamp,message").Order(timestamp).Count(record_count).Offset(#table_offset#)


Button code
"""""""""""

Buttons in Protypo can invoke contracts and open pages, depending on the arguments.

If you haven't already done so, open the page in the editor, and delete the existing *More* button.

Afterwards, locate the :func:`Div` function call that defines the footer, ``Div(Class: panel-footer text-right)``. Add the button code to it.

    .. code-block:: default

        Div(Class: panel-footer text-right) {

        }

The *Previous* button will be displayed only if there is at least one step to go back to. The new offset for the page, ``offset_previous`` is calculated when the button is added. Parameters are passed to the reopened page in the ``PageParams`` parameter.

    .. code-block:: default

        If(#table_offset# >= 25) {
            SetVar(offset_previous, Calculate(#table_offset# - 25))
            Button(Class: btn btn-primary, Body: Previous, Page: AppPage, PageParams:"table_offset=#offset_previous#")
        }


The *Next* button will be displayed only if the total record count is more than what is displayed on the page. The new offset for the page, ``offset_next`` is calculated when the button is added. Parameters are passed to the reopened page in the ``PageParams`` parameter.

    .. code-block:: default

        If(#record_count# >= Calculate(#table_offset# + 25)) {
            SetVar(offset_next, Calculate(#table_offset# + 25))
            Button(Class: btn btn-primary, Body: Next, Page: AppPage, PageParams:"table_offset=#offset_next#")
        }


.. image:: /_static/app-tut-navigation.png
    :scale: 60%

After the buttons are added, save the page and test it from the *Home* > *Messages* menu item.


Sending messages
^^^^^^^^^^^^^^^^

In Protypo, contracts are activated with buttons. 

The :func:`Button` function has two arguments for contracts:

* ``Contract``

    Name of the contract that must be activated.

* ``Params``

    Input parameters for the contract.


Form
""""

To send data to contracts, add a form to the app's page. This form must have an input field for the message, and a button that will activate the AppContract contract.

Below is an example of such form. It is enclosed in its own :func:`Div`. Place it after the Div element that holds the table. The :func:`Input` field of this form has a defined name, ``message_input``. This name is used by the button to send ``Message`` parameter value to the contract. Finally, :func:`Val` function is used to obtain the value of the input field.

.. code-block:: default

    Div(Class: panel panel-primary) {
      Form() {
            Input(Name: message_input, Class: form-control, Type: text, Placeholder: "Write a message...", )
            Button(Class: btn btn-primary, Body: Send, Contract: AppContract, Params: "Message=Val(message_input)")
      }
    }


Page refresh
""""""""""""

One final functionality that must be implemented is the automatic update of the table located on the page. When a user sends a new message, it must be displayed in the table.

You can implement this by making the *Send* button re-open the current page in addition to executing the contract. The ``#table_offset#`` parameter must be passed to the page without changes.

Add ``Page`` and ``PageParams`` arguments to *Send* button code like demonstrated below.

.. code-block:: default

    Button(Class: btn btn-primary, Body: Send, Contract: AppContract, Params: "Message=Val(message_input)", Page:AppPage, PageParams:"table_offset=#table_offset#")


Full page code
^^^^^^^^^^^^^^

This part introduced many changes to the application page template. Below is the full code for the app page.

.. code-block:: default

    If(GetVar(table_offset)){
    }.Else{
        SetVar(table_offset, 0)
    }

    DBFind(Name: apptable, Source: src_table).Columns(Columns: "author,timestamp,message").Order(timestamp).Count(record_count).Offset(#table_offset#)

    Div(Class: panel panel-primary) {
     Div(Class: panel-heading, Body: Table block)
     Table(Columns: "AUTHOR=author,TIME=timestamp,MESSAGE=message", Source: src_table)
     Div(Class: panel-footer text-right) {

      If(#table_offset# >= 25) {
        SetVar(offset_previous, Calculate(#table_offset# - 25))
        Button(Class: btn btn-primary, Body: Previous, Page: AppPage, PageParams:"table_offset=#offset_previous#")
      }
      
      If(#record_count# >= Calculate(#table_offset# + 25)) {
        SetVar(offset_next, Calculate(#table_offset# + 25))
        Button(Class: btn btn-primary, Body: Next, Page: AppPage, PageParams:"table_offset=#offset_next#")
      }

     }
    }

    Div(Class: panel panel-primary) {
      Form() {
            Input(Name: message_input, Class: form-control, Type: text, Placeholder: "Write a message...", )
            Button(Class: btn btn-primary, Body: Send, Contract: AppContract, Params: "Message=Val(message_input)", Page:AppPage, PageParams:"table_offset=#table_offset#")
      }
    } 


Conclusion
----------

This tutorial stops at the point where you have the basic application for your ecosystem. It doesn't explain other important topics for application developers like layout styles, access rights management and interaction between apps and resources. Please consult the rest of the documentation for more information about these advanced topics. 

.. todo::

    Redirect to content focus for app developers (/topics).
