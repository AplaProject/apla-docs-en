Application Development Tutorial
================================

This tutorial explains how to write a simple application for |platform| blockchain platform.


The goal
--------

The goal of this tutorial is to create an application for |platform| blockchain platform that can:

.. todo::
    
    Populate this list

* TBD
* TBD


The application starts simple and grows in complexity as tutorial progresses.

.. todo::
    
    This tutorial must explain basic access permissions for apps and for tables (what is MainCondition and how to change it).



Part 1: The environment
-----------------------


quick-start
^^^^^^^^^^^

This tutorial uses |platform| `quick start`_. The quick start provides `docker`_ containers for five network nodes. Each node has its own backend, database, and client. The manage.sh script can be used to control the nodes.

The quick start will install docker as a part of the installation process.


.. _quick start: https://github.com/AplaProject/quick-start
.. _docker: https://docs.docker.com/engine/docker-overview



Molis
^^^^^

For this tutorial, you'll be writing contract code and performing all other actions in the :any:`Molis client`. Molis also provides a way to retrieve, save, and execute contract code on the blockchain, manage data structures (tables), assign access rights, and create applications.

The root account for |platform| ecosystems is called :any:`ecosystem founder` account. You must use this account to introduce major changes to the ecosystem, such as creating new apps and tables.

To login to Molis with founder account:

    #. Run ``$ sudo ./manage.sh start-clients``

        This command starts Molis clients for all nodes. 

    #. One of the started clients is for the ecosystem founder account. This account lets you select roles after logging in. Choose *Admin*.

        Password for all accounts is |quick_pass|.

.. |quick_pass| replace:: ``default``



Part 2: What is an app
----------------------


App components
^^^^^^^^^^^^^^

Genesis apps are a combination of contracts, tables, and user interfaces.

* Contracts

    Contracts are like functions. They take input parameters, they validate these parameters, and perform the defined actions.


* Tables

    Tables are like database tables. They hold information that is used by contracts and user interfaces. 

    .. todo::
        
        What about table types?

        Tables are divided into registry tables and document tables. 

    Just like with regular database tables, you can query, update, and insert. These actions are performed with three Simvolio language functions: :func:`DBFind`, :func:`DBInsert`, and :func:`DBUpdate`.


* User interfaces

    User interfaces are UI layouts that will be displayed to the user by Molis. 

    The interfaces are written in :doc:`Protypo language </topics/templates2>`. Molis provides a visual editor for constructing interfaces.


Contract code sections
^^^^^^^^^^^^^^^^^^^^^^

Contract code is written in :doc:`Simvolio language </topics/script>`.

Every contract consists of three sections:

.. describe:: data

    Declares the input data (names and types of variables).

.. describe:: conditions

    Validates the input data.

.. describe:: action

    Performs actions defined by contract logic.


Modularity
^^^^^^^^^^

The architecture of genesis apps is designed to be modular. Contracts, tables, and interfaces are stored in the blockchain and can be used by many different apps. 

For example, if you change the contract code, you must make a transaction that introduces the contract code change. After the transaction is validated and included in the blockchain, the new contract code becomes available to all nodes in the blockchain network. Same principle applies to tables and data stored in them, and to user interfaces.

.. todo::
    Illustration needed. Modules available to many apps and to each other.


Part 3: Hello, World
--------------------

You now have your blockchain network of five nodes and a basic understanding of what is an app. Your first application will start as a simple Hello, World application.


The spec
^^^^^^^^

The application stores a single string on the blockchain. It doesn't have any user interface but uses a table to store the string. 


New app
^^^^^^^

|platform| apps are created via transactions. You can create new apps from Molis.

To create a new app:

    #. Login as the founder.

    #. Go to the *Admin* tab.

    #. From the list on the left, select *Application*.

    #. In the *Applications* view, select *Create*.

    #. Specify the name of your app in the *Name* field.

    #. In the *Change conditions* specify ``true``.
        
        The ``true`` value will make possible for anyone to change the app. 

        Another option is to specify ``ContractConditions("MainCondition")``. This will forbid application changes to anyone except the  founder account.

        .. todo::
            Explain what these conditions are (link).

    #. Your app will appear in the list of apps. Click *select* to make it active.


New table
^^^^^^^^^

To store data, the application needs a table. 

Just like apps and almost all other entities, tables are created via transactions. You can create tables from Molis.


To create a table:

    #. On the *Admin tab*, select *Resources* > *Tables*.

        This will display all tables for the selected app. The list will be empty, because your app doesn't have any tables yet.

    #. Click *Create*.

        Molis will display the *Create table* view.
        
    #. Specify a name for your table in the *Name* field.

        This tutorial uses ``testtable`` name for the table.

    #. Add a column. Name it ``message`` and set its type to ``Text``.

        As a result, the table must have two columns: ``id`` (predefined), and ``message``. You'll add more columns later.

    #. For write permissions, specify ``true`` in every field.

        This will allow anyone to perform inserts and updates on the table, and to add columns.

        As an option, you can restrict writing permissions to the founder account. In this case, specify ``ContractConditions("MainCondition")`` in this parameter.


The contract
^^^^^^^^^^^^

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
            error "Text is empty"
        }
    }


Action section
""""""""""""""

Fill the ``action`` section. The single action is writing the message to the table

.. code-block:: js

    action {
        DBInsert("testtable", "message", $Message)
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
                error "Text is empty"
            }
        }
        action {
            DBInsert("testtable", "message", $Message)
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

        The contract has one parameter, ``Message``, so:

            * Specify ``Message`` as *Key*.
            * Specify ``Hello, World`` as *Value*.

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

After a simple Hello, World app is working, it's time to expand it into something more useful. In this part, you'll be implementing the UI and extra functionality.


The spec
^^^^^^^^

The app stores strings on the blockchain, like entries in a log. Every string has an author and a timestamp. 

A user can view the stored list of strings from the application page, which is a simple table at this point. 

The app does not provide a way to add new strings from the UI yet.


New columns
^^^^^^^^^^^

Just like before, edit the table from the *Admin* > *Resources* > *Tables* view.

Add the following columns to the ``tesstable`` table:

* ``author`` of type ``Number`` with Update set to ``true``.

    This field will store the identifier of the author's account.

* ``timestamp`` of type ``Date/Time`` with Update set to ``true``.
    
    .. todo::

        How Update field works in this case? This is triggered on column update?


Updated contract
^^^^^^^^^^^^^^^^

Update the existing contract code to handle author IDs and timestamps. 

Author IDs are identifers of the ecosystem accounts. Timestamps are the date and time of the transaction in the Unix time format. 

Both of these values are provided by the :ref:`predefined values <simvolio-predefined-values>`. Since there is no need to input or validate the predefined values, changes are needed only in the action section.


Action section
""""""""""""""

Change the contract so that the author's ID and the timestamp are written to the table when a message is added. The author's ID is defined by ``$key_id``, the timestamp is defined by ``$time``.

.. code-block:: js

    action {
        DBInsert("testtable", "message, author, timestamp", $Message, $key_id, $time)
    }


The page
^^^^^^^^

For this part, UI is a simple page that displays information stored in the table.

Just like all other resources, UI pages can be created in Molis:

#. Navigate to *Admin* > *Resources* > *Pages*.

#. Click *Create*.

    A visual editor will open in the new tab.


Designer's view
"""""""""""""""

The default page is empty. Fortunately, you can use predefined structures to fill the page quickly.

Create a basic table with header: 

#. In the view selector on the right, click "Designer".

    The view will switch to the visual editor.

#. From the menu on the left, select *Table With Header* and drag it to the page.

    A table with several elements will appear.


Developer's view
""""""""""""""""

User interfaces for |platform| are written in :doc:`Protypo</topics/templates2>`. You'll need to write code for the page, so switch to the developer's view:

* In the view selector on the right, click "Developer".

    The view will switch to the code editor with the page code:

    .. code-block:: js

        Div(Class: panel panel-primary) {
            Div(Class: panel-heading, Body: Table block)
            Table(Source: test_key)
            Div(Class: panel-footer text-right) {
                Button(Class: btn btn-primary, Contract: ContractName, Body: More)
         }
        }


Get data from the table
"""""""""""""""""""""""

At the moment, the page template does nothing. Change the code, so that the page displays data from the ``testtable`` table.

#. To request data from a table, use the :ref:`DBFind function<DBFind_templates>`. 

    The function call in the following exaple gets data from the ``testtable`` table, puts it into the ``src_table`` source, and orders it by the timestamp field. The ``src_table`` source is later used as a source of data for the displayed table.

    .. code-block:: js

        DBFind(Name: testtable, Source: src_table).Columns(Columns: "author,timestamp,message").Order(timestamp)


#. To display data from the ``src_table`` source, specify it as a source along with a list of headers in the ``Table`` function.

    .. code-block:: js

        Table(Columns: "AUTHOR=author,TIME=timestamp,MESSAGE=message", Source: src_table)

#. Click *Preview* on the right to see if data is displayed correctly.


Full page code
""""""""""""""

Below is the full page code for this part. This basic page will be expanded later.

.. code-block:: js

    DBFind(Name: testtable, Source: src_table).Columns(Columns: "author,timestamp,message").Order(timestamp)

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

In the previous parts you've created a contract, a table to store data, and a basic UI to display this data.

In this part, you'll be finalizing the app, so it looks and behaves like an actual application.


The spec
^^^^^^^^

The app stores messages on the blockchain, like entries in a log. Every message has an author and a timestamp. 

A user can view the stored messages by opening the application UI from the ecosystem menu. The default view holds 25 mesages and provides a way to browse more.

The app provides a way to add new messages from the UI, one message at the time.


The menu
^^^^^^^^

Now that the app has a page, it's time to add a simple menu item that will open this page. 

Because each ecosystem can have many apps, this menu item must be added to the ecosystem's menu. This way other members of the ecosystem can access the page and view the messages.

.. note::

    In |platform|, an app is the collection of its resources: contracts, pages, and tables. All the resources of all the apps are available inside the ecosystem. One resource can be used by many apps and resources. Resources do not need to belong to the same app to be accessible.

    For example, a dashboard page can use many tables that store information about ecosystem members and business processes; a high-level contract can update several tables that are used by ecosystem apps.


Add a menu item
"""""""""""""""

Just like all other resources, menus can be created and edited in Molis:

#. Navigate to *Admin* > *Menu*.

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

#. Click *Messages*.

    The app's page will open.


Table navigation
^^^^^^^^^^^^^^^^

The default table can list only 25 first entries. Add a simple navigation that will allow users to navigate all table entries.

.. note::

    You can add extra table entries by executing the contract several times with different values.


The table navigation will use two buttons. Each button will  reload the app's page and pass parameters to it.


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

To calculate ``#record_count#``, modify the existing :ref:`DBFind <DBFind_templates>` function call. The variable specified in the ``.Count()`` call will store the record count.

    .. code-block:: js
        
        DBFind(Name: testtable, Source: src_table).Columns(Columns: "author,timestamp,message").Order(timestamp).Count(record_count)


Table offset
""""""""""""

The table offset must be passed to the page when it is opened. If ``#table_offset#`` is not passed, it is assumed to be ``0``.

Add the following code to the top of the page template. This code uses conditionals. :ref:`GetVar` function checks if the variable is set. :ref:`SetVar` function sets the variable.

    .. code-block:: js

        If(GetVar(table_offset)){
        }.Else{
            SetVar(table_offset, 0)
        }

Modify the :ref:`DBFind <DBFind_templates>` function call again. This time it must use the new table offset. 

    .. code-block:: js

        DBFind(Name: testtable, Source: src_table).Columns(Columns: "author,timestamp,message").Order(timestamp).Count(record_count).Offset(#table_offset#)


Buttons
"""""""

:ref:`Buttons <Button>` in Protypo can invoke contracts and open pages, depending on the arguments.

If you haven't already done so, open the page in the editor, and delete the existing *More* button.

Afterwards, locate the :ref:`Div` function call that defines the footer, ``Div(Class: panel-footer text-right)``. Add the button code to it. 

The *Previous* button will be displayed only if there is at least one step to go back to. The new offset for the page, ``offset_previous`` is calculated when the button is added. Parameters are passed to the reopened page in the ``PageParams`` parameter.

    .. code-block:: js

    If(#table_offset# >= 25) {
        SetVar(offset_previous, Calculate(#table_offset# - 25))
        Button(Class: btn btn-primary, Body: Previous, Page: AppPage, PageParams:"table_offset=#offset_previous#")
    }


The *Next* button will be displayed only if the total record count is more than what is displayed on the page. The new offset for the page, ``offset_next`` is calculated when the button is added. Parameters are passed to the reopened page in the ``PageParams`` parameter.

    .. code-block:: js

  If(#record_count# >= Calculate(#table_offset# + 25)) {
    SetVar(offset_next, Calculate(#table_offset# + 25))
    Button(Class: btn btn-primary, Body: Next, Page: AppPage, PageParams:"table_offset=#offset_next#")
  }
