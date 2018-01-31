################################################################################
Applications on Apla
################################################################################

.. contents::
  :local:
  :depth: 3
  
An application on Apla is a system of contracts, tables and interfaces that performs a certain function or provides a dedicated service. An application is not a autonomous program module - the only thing that unites the application elements is the performance of certain functions and exchange data. The boundaries of an application are not always strictly defined, since its elements can be simultaneously used in multiple applications.  

The following is a typical functional fragment of an application:

1. Redirect to a page that displays: 

* information, received from database tables (*DBFind* function), 
* form fields for user input of new data, 
* button (*Button* function), which calls a contract.

2. On the contract call:

* data from form fields is passed to the ``data`` section of the contract, 
* the ``conditions`` section checks user rights to launch this contract and the validity of data received from the page; if for some reason the contract cannot be executed, a message is displayed on the screen (``error``, ``warning`` or ``info``), leaving the user on the same interface page, 
* the ``action`` section receives additional data from tables (DBFind function) and records data into the database (using *DBUpdate* and *DBInsert* functions).

3. After the contract has been successfully executed, the user is redirected to the page, which name was specified in the *Page* parameter of the *Button* function that launched the contract, and the parameters listed in the *PageParams* are passed to the new page.

A page can have more than one button to call various contracts. Buttons that call contracts and redirect to pages can be built in rows of tables that display data about objects in the user interface. In this case, object identifiers are used in button parameters, which can be used to redirect to object-related pages (for example, object editing).
  
=========================
Tables and Data Storage
=========================

Applications use database tables that can be conditionally divided into two types: 

1. Tables that store big arrays of structured data about objects (persons, organizations, property, etc.), 
2. Document tables that store the current state of the processes implemented by a particular application (process type, stage, signatures, etc.) or data about its current operations (notifications,  messages, records about voting and transactions). 

Typically, register tables contain the most up-to-date information, which means that information about objects in the registers is updated as soon as a new piece of data is received. The work with documents is based on a completely different principle. Due to the fact that the *DBFind* functions  of the Simvolio and Protypo languages can request information only from a single table (which means that they don't support *JOIN*), it is necessary to record exhaustive information (IDs, names, pictures) in the tables that store documents. For example, when requesting information from a table that stores messages, we need to receive not just the ``id`` of the sender, but all information required to display the message on an interface page, including the user name and avatar (userpic). After having being saved, the data in the document tables should not be modified. It should be noted that non-normalization of data in tables is not related only to the technical limitations of *JOIN* use, but is prescribed by the very ideology of the blockchain as a temporal database that is designed to store the complete history of events. This means that a document (for example, a message), which was signed and saved in a table, should not under any circumstances be modified in the future, even, for example, in case its author changes their name in the register of users (whereas such modification is inevitable under the relational data scheme). 

=========================
Navigation
=========================
Users can switch between applications using sections, displayed in the software client as tabs, or using cascading menus inside these sections. After clicking on a section tab, users are redirected to the section's main page, which can be defined from the administrative tools. 
 
Navigation within an application is carried out for the most part using the menus (each page has one menu liked to it). Transition between pages can be implemented using links (*LinkPage*) or button clicks (*Button*). In both cases parameters *PageParams* can be passed to the *Page* target page. Calling a new page is also possible after successful execution of a contract. In this case, the parameters of the *Button* function, which calls the contract, should include the name of the page to redirect to (*Page*) and the passed parameters *PageParams*.

Navigation in multi-user applications can be organized using ready-to-use *Notification* and *Roles* applications, which are installed by default in every ecosystem. The *Notification* application allows for display of messages to specific users or user roles in the Molis software client. A notification message consists of a header and a link to a page. Additionally, parameters can be passed using *PageParams* to the target page in the format, supported by the *LinkPage* and *Button* functions. Sometimes, a target page where, for example, a user needs to make some decision, can be accessed only from a notification, because there are no links from menus or other pages to this target page. Notifications are formed with contracts *Notifications_Single_Send* or *Notifications_Roles_Send*, which should be called from a contract that ends some functional stage of an application. After the user receives a notification, opens the target page and performs the required action (thus, executing a contract), the notification should be deactivated by calling the *Notifications_Single_Close* or *Notifications_Roles_Finishing* contract. The list of notifications and their statuses is available in the *Notification* application.

=========================
Interface Pages
=========================
Page structure is formed using the ``If(Condition){Body } .ElseIf(Condition){Body} .Else{Body}`` conditional statement, which can use the PageParam parameters sent to the page when it was called by the LinkPage or Button functions. If some code fragments need to be used in many pages, they should be recorded into page blocks. Such blocks can be included in a page using the *Include* function.

=========================
Variables, Column Names and Language Resources
=========================
The unification of names of variables (on pages and in contracts), identifiers of interface page form fields, table column names and language resource labels can help significantly speed up the development of applications and make the program code easier to read. Let's suppose we want to pass parameters from an interface page to a contract. In this case, if a the name of the username variable in the data section of the contract corresponds to the name of the username field of an interface page, which was passed to this contract, then you don't need to specify this ``username=username`` pair in the *Params* parameters of the *Button* function. Using the same names for variables and column names makes it easier to use the DBInsert and DBUpdate functions; for example, ``DBUpdate("member", $id, "username",$username)``. Using the same names for variables and language resource labels makes it easier to display the columns names of interface tables ``Table(mysrc,"ID=id,$username$=username")``.

=========================
Access Rights
=========================
The most important element of an application is the system for management of access rights to its resources. These access rights can be established on a number of levels:

1. Permission to call a specific contract by the current user. This permission can be configured in the ``conditions`` section of the contract by using a logical expression in the ``If`` statement, or with nested contracts; for example, *MainConditions* or *RoleConditions*, where typical rights or user role rights are defined.
2. Current user's permission to change (using the contracts) values in table columns or to add rows to tables. Permissions can be set using the *ContractConditions* function in *Permissions* fields of table columns and in the *Permissions / Insert* field on the table editing page.
3. Permission only for specific contracts to change values in table columns or to add rows to tables. Contract names should be specified in the parameters of the *ContractAccess* function, which should be written in *Permissions* fields of table columns, and in the *Permissions / Insert* field on the table editing page.
4. Permission to edit application elements (contracts, pages, menu, and page blocks). Permissions can be set in the *Change conditions* fields in element editors. This is done using the *ContractConditions* function, to which the name of the contract that checks the permissions of the current user should be passed as a parameter.

=========================
Application Example:  SendTokens
=========================
The application sends tokens from one user account to another. Information about the amounts of tokens on accounts is stored in the *keys* tables (*amount* column), which are installed in ecosystems by default. This example implies that the tokens have already been distributed to user accounts. 

System Contract
-----------------
The main contract for this application is the *TokenTransfer* contract, which has the exclusive permission to change values in the *amount* column of the *keys* table. In order to activate this permission, we should write the ``ContractAccess("TokenTransfer")`` function in the *Permissions* field of the *amount* column. From this moment, any operations with tokens can only be carried out by calling the TokenTransfer contract.

In order to prevent the execution of the TokenTransfer contract from within another contract without the knowledge of the account holder, TokenTransfer should be a contract with confirmation. This means that its ``data`` section should contain the ``Signature string "optional hidden"`` string, and the confirmation parameters should be set on the *Contracts with Confirmation* page in the Molis administrative tools, which includes: text and parameters that should be displayed to the user in a pop-up information window (for details, see the *Contracts with Confirmation* section).

.. code:: js

    contract TokenTransfer {
    data {
        Amount money
        Sender_AccountId int
        Recipient_AccountId int
        Signature string "optional hidden"
    }
    conditions {
        //check the sender
        $sender = DBFind("keys").Where("id=$", $Sender_AccountId)
        if(Len($sender) == 0){
            error Sprintf("Sender %s is invalid", $Sender_AccountId)
        }
        $vals_sender = $sender[0]
    
        //check the recipient
        $recipient = DBFind("keys").Where("id=$", $Recipient_AccountId)
        if(Len($recipient) == 0){
            error Sprintf("Recipient %s is invalid", $Recipient_AccountId)
        }
        $vals_recipient = $recipient[0]
    
        //check amount
        if $Amount == 0 {
            error "Amount is zero"
        }
    
        //check balance
        var sender_balance money
        sender_balance = Money($vals_sender["amount"])
        if $Amount > sender_balance {
            error Sprintf("Money is not enough %v < %v", sender_balance, $Amount)
        }
    }
    action {
        DBUpdate("keys", $Sender_AccountId, "-amount", $Amount)
        DBUpdate("keys", $Recipient_AccountId, "+amount", $Amount)
    }
    }
    
The following checks are carried out in the conditions section of the TokenTransfer contract: the accounts involved in the transaction should exist, the amount of tokens to be transferred should be non-zero, the amount of transaction should be smaller or equal to the balance of the sender's account. The action section carries out the modification of values in the amount column of the sender's and receiver's accounts.

Token Sending Form
-----------------
The token sending form contains fields to input the transaction amount and the recipient address.  

.. code:: js

    Div(Class: panel panel-default){
      Form(){ 
        Div(Class: list-group-item text-center){
          Span(Class: h3, Body: LangRes(SendTokens))  
        }
        Div(Class: list-group-item){
          Div(Class: row df f-valign){
            Div(Class: col-md-3 mt-sm text-right){
              Label(For: Recipient_Account){
                Span(Body: LangRes(Recipient_Account))
              }
            }
            Div(Class: col-md-9 mb-sm text-left){
              Input(Name: Recipient_Account, Type: text, Placeholder: "xxxx-xxxx-xxxx-xxxx") 
            } 
          }
          Div(Class: row df f-valign){
            Div(Class: col-md-3 mt-sm text-right){
              Label(For: Amount){
                Span(Body: LangRes(Amount))
              }
            }
            Div(Class: col-md-9 mc-sm text-left){
              Input(Name: Amount, Type: text, Placeholder: "0", Value: "5000000")
            } 
          }
        }
        Div(Class: panel-footer clearfix){
          Div(Class: pull-right){
            Button(Body: LangRes(send), Contract: SendTokens, Class: btn btn-default)
          }
        }
      }
    }      
    
We could use the Button function to directly call the TokenTransfer transfer contract and pass the current user's (sender) account address to it, but for the purpose of demonstration of the work of contracts with confirmation we'll create an intermediary user contract SendTokens. It is important to note, that since the names of data in the data section of the contract and the names of interface form fields are the same, we don't need to specify the Params parameters in the Button function.

The form can be placed on any page in the software client. After the contract execution has ended, the user will stay on the current page (because we didn't specify a target page Page in the Button function).

Custom Contracts
-----------------
The TokenTransfer contract is defined as a contract with confirmation, and that is why in order to call it from another contract we need to place the Signature string "signature:TokenTransfer" in the data section of our custom contract. 
The conditions section of the SendTokens contract checks the availability of the account; the action section calls the TokenTransfer contract and passes parameters to it.

.. code:: js

    contract SendTokens {
        data {
            Amount money
            Recipient_Account string
            Signature string "signature:TokenTransfer"
        }
    
        conditions {
            $recipient = AddressToId($Recipient_Account)
            if $recipient == 0 {
                error Sprintf("Recipient %s is invalid", $Recipient_Account)
            }
        }
    
        action {
            TokenTransfer("Amount,Sender_AccountId,Recipient_AccountId,Signature", $Amount, $key_id, $recipient, $Signature)
        }
    }


